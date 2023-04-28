---
title: "在 CentOS 上编译 Ceph"
date: 2023-04-17 14:49:00
updated: 2023-04-17 14:49:00
categories: [技术杂谈]
tags: [Linux,CentOS,Ceph]
---



本文编译环境为：

* 硬件架构：x86_64
* Linux 版本：CentOS 7.3.1611
* Ceph 版本： v14.2.21

\-

由于：  

1. Ceph 的依赖过于复杂（且似乎包含很多无效依赖）；
2. 官方提供的编译过程可能会遭遇大量问题；
3. 在不同的硬件架构和系统版本上编译不同版本的 Ceph，编译过程中遭遇的错误可能会有很大不同。

所以 **此文章仅对  x86_64 上的 CentOS 7.3.1611 与 Ceph 14.2.21 负责！**

\-

另外有几点需要注意：

1. Ceph 的编译全过程耗时可能很长，具体取决你的机器配置。
2. Ceph 的依赖安装和编译等操作需要大量的磁盘空间，建议给 `/` 目录预留 32G 以上空间。
3. Ceph 的编译过程会涉及到大量的网络下载，请自行解决相关的网络问题（例如 使用境外服务器编译、配置代理 等）。
4. Ceph 的整个编译过程中有多处需要 root 权限，但**请使用 root 用户直接操作！请使用 root 用户直接操作！请使用 root 用户直接操作！**不要在普通用户上通过 `sudo` 或 `sudo su` 来操作，否则可能会有各种奇奇怪怪的问题而且很难解决（Ceph 安装依赖、编译的过程里，很多很多操作都是在其内部的环境进行的。比如不管你系统里装了哪些版本的 Python，Ceph 都只会用其自己目录下的那一个，而且是临时下载的，不管依赖安装是否成功，脚本一结束就把这些都删掉了，我还排查不了。想要解决就得深度修改编译脚本、Makefile、CMakeLists.txt 等的。我真的已经麻了！！！）。





## 1. 配置外部环境

安装 yum 包：

```shell
yum install -y git \
               centos-release-scl
```

```shell
yum install -y devtoolset-11
```

启动 `devtoolset-11`：

```shell
scl enable devtoolset-11 bash
```

`devtoolset-11` 是个开发工具集，比如这里启动后 GCC 版本将变为 11.x（你也可以装其他版本的，这里只要 GCC 7+ 就可以）。此命令仅对当前终端生效。



## 2. 准备 Ceph 源码



我这里直接克隆 v14.2.21 版本的 Ceph 源码仓库：

```shell
git clone --branch v14.2.21 https://github.com/ceph/ceph
```

进入 Ceph 目录：

```shell
cd ceph
```

检查/更新子模块：

```shell
git submodule update --init --recursive
```





## 3. 安装依赖



执行官方提供的依赖安装脚本：

```shell
./install-deps.sh
```



\-



**可能遇到的问题 1：**

```
Error: No Package found for python-scipy
```

解决方法：

```shell
sed -i -e '/BuildRequires:  python%{_python_buildid}-scipy/d' ceph.spec.in
```

然后重新执行刚刚的脚本。



## 4. 编译 Ceph

先执行 `./do_cmake.sh` 脚本：

```shell
./do_cmake.sh
```

执行成功会有类似如下的输出：

```
-- Build files have been written to: /root/ceph/build
+ cat
+ echo done.
done.
+ cat

****
WARNING: do_cmake.sh now creates debug builds by default. Performance
may be severely affected. Please use -DCMAKE_BUILD_TYPE=RelWithDebInfo
if a performance sensitive build is required.
****
```

然后会在当前目录生成一个 `build/` 目录，我们进入：

```shell
cd build/
```

执行编译（如果你机器配置不高的话，可能需要很长很长很长时间）：

```
make -j`nproc`
```



\-

**可能遇到的问题 1：**

执行 `make` 的时候，报错如下：

```
make[2]: *** [src/test/CMakeFiles/ceph_test_librgw_file_marker.dir/build.make:83: src/test/CMakeFiles/ceph_test_librgw_file_marker.dir/librgw_file_marker.cc.o] Error 1
make[1]: *** [CMakeFiles/Makefile2:10947: src/test/CMakeFiles/ceph_test_librgw_file_marker.dir/all] Error 2
make[1]: *** Waiting for unfinished jobs....
make[2]: *** [src/test/CMakeFiles/ceph_test_librgw_file_nfsns.dir/build.make:83: src/test/CMakeFiles/ceph_test_librgw_file_nfsns.dir/librgw_file_nfsns.cc.o] Error 1
make[1]: *** [CMakeFiles/Makefile2:10794: src/test/CMakeFiles/ceph_test_librgw_file_nfsns.dir/all] Error 2
make: *** [Makefile:161: all] Error 2
```

解决方法：先回到上一级 Ceph 根目录，执行：

```
sed -i src/test/librgw_file.cc -e "s/CLEANUP/CLEANUP1/"
sed -i src/test/librgw_file_aw.cc -e "s/CLEANUP/CLEANUP4/"
sed -i src/test/librgw_file_cd.cc -e "s/CLEANUP/CLEANUP3/"
sed -i src/test/librgw_file_gp.cc -e "s/CLEANUP/CLEANUP2/"
sed -i src/test/librgw_file_marker.cc -e "s/CLEANUP/CLEANUP6/"
sed -i src/test/librgw_file_nfsns.cc -e "s/CLEANUP/CLEANUP5/"
```

然后再回到 `build/` 目录重新 `make`。



## 5. 验证 Ceph



继续在 `./build/` 目录下，执行：

```shell
RGW=1 ../src/vstart.sh -n -d
```

这个命令会启动 RGW 测试环境。

如果有类似下面的输出（节选），就是成功了：

```shell
# RGW=1 ../src/vstart.sh -n -d
** going verbose **
rm -f core* 
hostname iZt4nhk1n6uvz2shbiu9mlZ
ip 172.31.24.59
port 40086
/root/ceph/build/bin/ceph-authtool --create-keyring --gen-key --name=mon. /root/ceph/build/keyring --cap mon 'allow *' 
creating /root/ceph/build/keyring
/root/ceph/build/bin/ceph-authtool --gen-key --name=client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *' /root/ceph/build/keyring 
...
...
...
2023-04-17 11:48:41.363 7fce2a17fa80 -1 AuthRegistry(0x7ffdfce18b48) no keyring found at /root/ceph/build/dev/osd0/keyring, disabling cephx
failed to fetch mon config (--no-mon-config to skip)
```

这里面一些类似报错的信息主要是我们还没有配置过 Ceph，这里就是看看编译成功没有，先不关注这个，跑起来就 OK。

然后我们可以用 `ps` 查看以下 Ceph 的运行情况：

```shell
# ps -ef | grep ceph
root     107783      1  0 11:49 ?        00:00:01 /root/ceph/build/bin/ceph-mon -i a -c /root/ceph/build/ceph.conf
root     107826      1  0 11:49 ?        00:00:00 /root/ceph/build/bin/ceph-mon -i b -c /root/ceph/build/ceph.conf
root     107869      1  0 11:49 ?        00:00:00 /root/ceph/build/bin/ceph-mon -i c -c /root/ceph/build/ceph.conf
root     108112      1  1 11:49 ?        00:00:02 /root/ceph/build/bin/ceph-mgr -i x -c /root/ceph/build/ceph.conf
root     108246  23481  0 11:51 pts/0    00:00:00 grep --color=auto ceph
```

\-

停止的话，输入：

```shell
../src/stop.sh
```

即可停止，再次使用 `ps` 查看，已经没有相关 Ceph 进程了：

```shell
# ps -ef | grep ceph
root     108290  23481  0 11:53 pts/0    00:00:00 grep --color=auto ceph
```



