---
title: "解决 mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.30' not found"
date: 2022-10-27 17:07:45
updated: 2022-10-27 17:07:45
categories: [技术杂谈]
tags: [Linux,CentOS,MySQL,数据库]
---





## 1. 遇到的错误报告

在安装 mysql 时可能会遇到类似如下错误（节选）：

```
...
mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by xxx)
...
mysqld: /usr/lib64/libstdc++.so.6: version `CXXABI_1.3.13' not found (required by xxx)
...
```



## 2. 错误分析



猜测主要原因是 GCC 的版本过低，而编译过程依赖更新版本 GCC 中的库。

以 `GLIBCXX` 举例，我们按如下命令我们当前 `/usr/lib64/libstdc++.so.6` 关联的 `GLIBCXX` ：

```shell
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX*
```

```shell
[gukaifeng@3afe42f77751 ~]$ strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX*
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBCXX_3.4.25
GLIBCXX_DEBUG_MESSAGE_LENGTH
```

可以看到这里面确实没有我们刚刚需求的更高版本的 `GLIBCXX`，我上面节选的错误显示缺少的版本是 `GLIBCXX_3.4.30`，而 `/usr/lib64/libstdc++.so.6` 关联的 `GLIBCXX`  最高版本是 `GLIBCXX_3.4.25`，确实没有 `GLIBCXX_3.4.30`。

这里也就验证了我们的猜想。





## 3. 具体的解决方案

解决方法有两个：

1. 更新 GCC 到更高版本;
2. 只下载我们缺少的库，然后将其链接到 `libstdc++.so.6`。

下面两个方法二选一即可。



### 3.1. 更新 GCC 到更高版本



下载并解压新版 GCC：

```shell
wget https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz
tar -zxvf gcc-12.2.0.tar.gz
cd gcc-12.2.0/
```

我这里下载的版本是 12.2.0（撰写此篇文章时的最新版本），你也可以[下载其他版本](https://ftp.gnu.org/gnu/gcc/)。

执行 `download_prerequisites` 脚本，下载 `gcc` 依赖文件和库：

```shell
./contrib/download_prerequisites
```

输出类似如下信息即为成功：

```
[terark@3afe42f77751 gcc-12.2.0]$ ./contrib/download_prerequisites
gmp-6.2.1.tar.bz2: OK
mpfr-4.1.0.tar.bz2: OK
mpc-1.2.1.tar.gz: OK
isl-0.24.tar.bz2: OK
All prerequisites downloaded successfully.
```

>这里可能会遇到的问题：
>
>`./contrib/download_prerequisites: line 261: bzip2: command not found`
>
>解决方法：
>
>`sudo yum install -y bzip2`



然后配置编译：

```shell
mkdir build && cd build
../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
```

开始编译，这里可能耗时较长：

```shell
make -j`nproc`
```

安装：

```shell
make install
```

