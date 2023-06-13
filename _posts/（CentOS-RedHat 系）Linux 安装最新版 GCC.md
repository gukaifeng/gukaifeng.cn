




## 1. 准备

因为编译 GCC 源码的时候是需要 GCC 的，并且支持 C++ 11，所以我们得确保系统里当前是存在 GCC 的，如果没有的话，先安装一个：

```shell
sudo yum install gcc-c++
```

`yum` 中提供的 GCC 版本是 8.5.0，是满足我们编译最新版 GCC 源码的要求的。

也就是说，我们所谓的安装最新版 GCC，其实是升级现有的 GCC 到最新版。

另外 GCC 的编译耗时可能非常长（也取决你的配置），如果你是着急用的话，可能不太行。



## 2. 下载最新版 GCC

https://ftp.gnu.org/gnu/gcc/  中可以看到全部的 GCC 版本，找到你需要的版本的文件夹，进入，里面名字形如 `gcc-12.2.0.tar.gz` 就是我们需要的（一般也是 Size 最大的那个）。

我这里是找到最新版 GCC，即 12.2.0。



## 3. 编译安装



我一般喜欢在一个干净的文件夹内做一件事情（个人习惯）：

```shell
mkdir ~/gcc-install && cd ~/gcc-install
```

下面进入正题。

下载 tar 包并解压：

```shell
wget https://ftp.gnu.org/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz
tar -zxvf gcc-12.2.0.tar.gz
cd gcc-12.2.0/
```



执行 `download_prerequisites` 脚本，下载 `gcc` 依赖文件和库：

```shell
./contrib/download_prerequisites
```

输出类似如下信息即为成功：

```shell
$ ./contrib/download_prerequisites
gmp-6.2.1.tar.bz2: OK
mpfr-4.1.0.tar.bz2: OK
mpc-1.2.1.tar.gz: OK
isl-0.24.tar.bz2: OK
All prerequisites downloaded successfully.
```

> 这里可能会遇到的问题：
>
> ```
> ./contrib/download_prerequisites: line 261: bzip2: command not found
> ```
>
> 解决方法：
>
> ```shell
> sudo yum install -y bzip2
> ```

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

到这里，GCC 的安装就结束了，我们可以通过 `gcc -v` 检查版本（需要重登 SSH 终端或重启系统）：

```
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-pc-linux-gnu/12.2.0/lto-wrapper
Target: x86_64-pc-linux-gnu
Configured with: ../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 12.2.0 (GCC) 
```

可以看到，GCC 的版本已经是我们刚刚安装的新版本了。



