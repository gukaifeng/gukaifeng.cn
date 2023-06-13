




## 1. 遇到的错误报告

在安装/运行 mysql 时可能会遇到类似如下错误（节选）：

```
...
mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by xxx)
...
mysqld: /usr/lib64/libstdc++.so.6: version `CXXABI_1.3.13' not found (required by xxx)
...
```

> 其他场景也可能遇到类似错误，这其实很高频，解决方法都是一样的。



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
2. 只更新 `libstdc++.so.6` 库，然后修改 `/usr/lib64/libstdc++.so.6` 的链接目标。

下面两个方法二选一即可。



### 3.1. 更新 GCC 到更高版本



下载并解压新版 GCC：

```shell
wget https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz
tar -zxvf gcc-12.2.0.tar.gz
cd gcc-12.2.0/
```

我这里下载的版本是 12.2.0（撰写此篇文章时的最新版本），你也可以[下载其他版本](https://ftp.gnu.org/gnu/gcc/)。

执行 `download_prerequisites` 脚本，下载 `gcc` 依赖文件和库：

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

开始编译，这里可能耗时较长（**非常长！！！**）：

```shell
make -j`nproc`
```

安装（若提示无权限则加上 `sudo`）：

```shell
make install
```

到这里，GCC 的安装就结束了，我们可以通过 `gcc -v` 检查版本：

```
[gukaifeng@3afe42f77751 ~]$ gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-pc-linux-gnu/12.2.0/lto-wrapper
目标：x86_64-pc-linux-gnu
配置为：../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
线程模型：posix
Supported LTO compression algorithms: zlib
gcc 版本 12.2.0 (GCC) 
```

可以看到，GCC 的版本已经是我们刚刚安装的新版本了。

\-

**但是，现在如果我们再次执行之前安装 mysql 出错的语句，依然会有同样的错误。**

```
...
mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by xxx)
...
mysqld: /usr/lib64/libstdc++.so.6: version `CXXABI_1.3.13' not found (required by xxx)
...
```



原因是 `/usr/lib64/libstdc++.so.6` 并没有链接我们的新版本库。

**注意，`/usr/lib64/libstdc++.so.6` 是个软链接！**

```shell
[gukaifeng@3afe42f77751 ~]$ ll /usr/lib64/libstdc++.so.6
lrwxrwxrwx 1 root root 19 11月 13  2021 /usr/lib64/libstdc++.so.6 -> libstdc++.so.6.0.25
```

可以看到，目前其连接到 `libstdc++.so.6.0.25`。

我们查看下我们最新安装的库的位置：

```shell
[gukaifeng@3afe42f77751 ~]$ find /usr -name libstdc++.so*
/usr/lib/gcc/x86_64-redhat-linux/8/32/libstdc++.so
/usr/lib/gcc/x86_64-redhat-linux/8/libstdc++.so
/usr/share/gdb/auto-load/usr/lib64/__pycache__/libstdc++.so.6.0.25-gdb.cpython-36.opt-1.pyc
/usr/share/gdb/auto-load/usr/lib64/__pycache__/libstdc++.so.6.0.25-gdb.cpython-36.pyc
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.25-gdb.py
/usr/lib64/libstdc++.so.6
/usr/lib64/libstdc++.so.6.0.25
/usr/local/lib64/libstdc++.so
/usr/local/lib64/libstdc++.so.6
/usr/local/lib64/libstdc++.so.6.0.30
/usr/local/lib64/libstdc++.so.6.0.30-gdb.py
```

最后面的 `/usr/local/lib64/libstdc++.so.6.0.30`（找最新版） 就是我们最终需要的（其他几个都是软链接，链接到这个）。

我们将 `/usr/lib64/libstdc++.so.6` 重新链接到 `/usr/local/lib64/libstdc++.so.6.0.30` 即可。

```shell
unlink /usr/lib64/libstdc++.so.6
ln -s /usr/local/lib64/libstdc++.so.6.0.30 /usr/lib64/libstdc++.so.6
```



**现在，我们再重试一开始的操作，就会发现正常运行了，OVER！**

 

### 3.2. 仅下载并链接缺少的库



（未完成，因为我遇到此问题的时候，上面的方法已经解决了，等我下次遇到这个问题的时候，来补充此方法。）
