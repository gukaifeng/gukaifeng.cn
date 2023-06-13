


本文不介绍 JuiceFS ，不了解的朋友可以看一下[官网](https://juicefs.com/)，或我写过的 [JuiceFS 单机模式初体验](https://gukaifeng.cn/posts/juicefs-dan-ji-mo-shi-chu-ti-yan/)。



## 1. 编译目标



如果你有使用过 JuiceFS 或看过相关文档，就会知道 JuiceFS 实际上只有一个二进制文件，用起来相当简单。

所以我们的编译目标也很简单了，就是**编译得到一个二进制可执行文件。**



## 2. 下载 JuiceFS 源码



我这里还是先创建一个空文件夹，并在这里面进行后续操作（个人习惯）：

```shell
mkdir juicefs-compile && cd juicefs-compile
```



然后克隆 JuiceFS 源码：

```shell
git clone https://github.com/juicedata/juicefs.git
```



## 3. 安装依赖



JuiceFS 的依赖主要有两个：Golang 1.18+ 和 GCC 5.4+，如果你的环境里已经有了这两个依赖，则可以跳过这小节。

可以使用 `go version` 和 `gcc -v` 两个命令检查以下。



### 3.1. 简单安装的方式

最简单的安装方式就是 yum 直接安装，我这里 yum 内提供的版本是 Golang  1.18.10 和 GCC 8.5.0，均符合要求。

```shell
sudo yum install golang gcc
```



### 3.2. 安装最新稳定版本（可选）

因为我这个是刚配好的开发机，很空，而且我们知道 Golang 也好，GCC 也好，其版本在未来的开发中是蛮重要的。

所以我打算在这个开发机上，手动安装当前最新的 Golang 和 GCC 版本，而不是使用 yum 快速安装（虽然 yum 中提供的版本是符合要求的）。

安装最新版的 Golang 和 GCC 是比较独立的两件事，所以我写在了两个独立的文章内：



* [（CentOS/RedHat 系）Linux 安装最新版 Golang](https://gukaifeng.cn/posts/centos-redhat-xi-linux-an-zhuang-zui-xin-ban-golang/)
* [（CentOS/RedHat 系）Linux 安装最新版 GCC](https://gukaifeng.cn/posts/centos-redhat-xi-linux-an-zhuang-zui-xin-ban-gcc/)



本文最终安装的分别是 Golang 和 GCC 12.2.0：

```shell
$ go version
go version go1.20.3 linux/amd64
```

```shell
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



>因为 Go 会在编译时从网络获取 Go 模块，所以对于中国地区用户，为了加快获取 Go 模块的速度，可以通过执行：
>
>```shell
>go env -w GOPROXY=https://goproxy.cn,direct
>```
>
>来将 `GOPROXY` 环境变量设置国内的镜像服务器。详情请参考：[Goproxy China](https://github.com/goproxy/goproxy.cn)。



## 4. 编译



进入源码目录：

```shell
cd juicefs
```

`git tag` 查看目前都有哪些 tag：

```shell
$ git tag | tail -1
v1.0.4
```

可以看到目前最新的 tag 是 `v1.0.4`，这也就是我们要编译的版本了（你也可以选择其他版本，`v1.0.4` 是截止此文章编写时的最新 tag）。

切换到目标 tag：

```shell
git checkout v1.0.4
```

编译：

```shell
make -j`nproc`
```



## 5. 安装

上面的编译完成后，会在当前目录下输出一个名为 `juicefs` 的二进制文件，也就是我们最终的编译目标：

```shell
$ ll | grep juicefs
-rwxrwxr-x.  1 gukaifeng gukaifeng 76337680 Apr 10 14:55 juicefs
```



将其移动到 `/usr/local/bin/` 目录下，就算安装完成了：



```shell
sudo mv juicefs /usr/local/bin/
```



## 6. 验证



简单验证以下，命令行直接输入 `juicefs`，应当输出其帮助信息：



```shell
$ juicefs
NAME:
   juicefs - A POSIX file system built on Redis and object storage.

USAGE:
   juicefs [global options] command [command options] [arguments...]

VERSION:
   1.0.4+2023-04-06.f1c475d

COMMANDS:
   ADMIN:
     format   Format a volume
     config   Change configuration of a volume
     destroy  Destroy an existing volume
     gc       Garbage collector of objects in data storage
     fsck     Check consistency of a volume
     dump     Dump metadata into a JSON file
     load     Load metadata from a previously dumped JSON file
     version  Show version
   INSPECTOR:
     status   Show status of a volume
     stats    Show real time performance statistics of JuiceFS
     profile  Show profiling of operations completed in JuiceFS
     info     Show internal information of a path or inode
   SERVICE:
     mount    Mount a volume
     umount   Unmount a volume
     gateway  Start an S3-compatible gateway
     webdav   Start a WebDAV server
   TOOL:
     bench     Run benchmarks on a path
     objbench  Run benchmarks on an object storage
     warmup    Build cache for target directories/files
     rmr       Remove directories recursively
     sync      Sync between two storages

GLOBAL OPTIONS:
   --verbose, --debug, -v  enable debug log (default: false)
   --quiet, -q             show warning and errors only (default: false)
   --trace                 enable trace log (default: false)
   --no-agent              disable pprof (:6060) and gops (:6070) agent (default: false)
   --pyroscope value       pyroscope address
   --no-color              disable colors (default: false)
   --help, -h              show help (default: false)
   --version, -V           print version only (default: false)

COPYRIGHT:
   Apache License 2.0
```



Over ！





