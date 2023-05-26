---
title: "I/O 测试工具 fio 详解"
date: 2023-05-26 00:25:00
updated: 2023-05-26 00:25:00
categories: [技术杂谈]
tags: [fio]
---





## 1. 什么是 fio ？

fio（Flexible I/O Tester）是一个用于测试和评估 I/O 负载和性能的工具，具有高度的可配置性和灵活性。它可以模拟不同类型的负载，包括随机和顺序读写，随机和顺序混合读写，随机和顺序的深度嵌套目录结构等，同时可以产生各种不同的 I/O 负载模式，例如随机性、对齐和分配大小、顺序性等特征，以便更好地模拟实际应用程序中的 I/O 负载。fio 在评估和比较存储系统方面非常有用，例如硬盘驱动器、固态硬盘、闪存存储器、网络存储器等。 \- ChatGPT





本文主要介绍在 Linux 上使用 fio 测试磁盘/文件系统的性能，fio 的其他功能本文不关注。  
对磁盘和文件系统的测试可以认为是一回事，因为文件系统挂载后在使用方面与本地磁盘没有什么区别，所以本文不会仔细区分这两个场景。



本文所使用的是系统为 CentOS 8.4.2105，其他发行版与本文可能会有出入，本文不做解释，可自行查看文档  [fio - Flexible I/O tester](https://fio.readthedocs.io/en/latest/fio_doc.html)。



## 2. 安装 fio



安装 fio 有几种常用方法，可以直接通过包管理工具（如 yum、dnf 等）安装，或者是手动编译安装，还有就是官方有提供编译好的 fio 二进制文件下载。



大部分场景中，我们使用包管理器安装就可以了。不过要知道，通常包管理器中的版本是要落后于官方最新版本的，例如我这里：

```shell
$ cat /etc/redhat-release 
CentOS Linux release 8.4.2105

$ yum info fio
Last metadata expiration check: 0:02:18 ago on Fri 26 May 2023 12:42:20 AM CST.
Available Packages
Name         : fio
Version      : 3.19
Release      : 3.el8
Architecture : x86_64
Size         : 538 k
Source       : fio-3.19-3.el8.src.rpm
Repository   : appstream
Summary      : Multithreaded IO generation tool
URL          : http://git.kernel.dk/?p=fio.git;a=summary
License      : GPLv2
Description  : fio is an I/O tool that will spawn a number of threads or processes doing
             : a particular type of io action as specified by the user.  fio takes a
             : number of global parameters, each inherited by the thread unless
             : otherwise parameters given to them overriding that setting is given.
             : The typical use of fio is to write a job file matching the io load
             : one wants to simulate.
```

可以看到我这里的系统为 CentOS 8.4.2105，yum 中提供的 fio 版本为  3.19。而截止本文撰写时间，官方最新版是 3.35。



可以通过 [fio 的开源仓库](https://github.com/axboe/fio/) 查到，fio 3.19 的发布时间是 2020/05/13，fio 3.35 的发布时间是 2023/05/24，yum 中的版本足足落后了 3 年多。



所以这小节两种 fio 安装方法都介绍，读者根据自己的使用场景选择就好，即简单使用可以直接包管理器安装，追求新版本特性则可以手动编译安装或者下载官方提供的 fio 二进制文件。





### 2.1. 包管理器安装



包管理器安装 fio 非常简单，以 CentOS 中的 yum 为例：



```shell
yum install fio
```







### 2.2. 手动编译安装



#### 2.2.1. 下载 fio 源码



fio 的源码仓库有好几个：

1. https://git.kernel.dk/cgit/fio/
2. https://git.kernel.org/pub/scm/linux/kernel/git/axboe/fio.git
3. https://github.com/axboe/fio.git

也可以从 https://brick.kernel.dk/snaps/ 中下载指定版本的 fio 的 tar 包，解压出源码。



这里以从 GitHub 仓库获取为例：



```shell
git clone https://github.com/axboe/fio.git
```



然后我们进入源码目录：

```shell
cd fio/
```

#### 2.2.2. 编译 fio



以此执行以下操作 ~



```shell
./configure
```

```shell
make
```

```shell
make install
```

注意 `make install` 可能需要 root 权限。



然后就编译安装完成了。





### 2.3. 下载二进制文件



官方提供了各种系统的 fio 二进制文件下载，这里也是有最新版的。我这里直接把官方链接贴过来了，自行下载就好。

唯一要注意的是记得把下载好的 fio 二进制文件移动到 `$PATH` 里，不然用起来太麻烦。



**Debian:**



Starting with Debian “Squeeze”, fio packages are part of the official Debian repository. https://packages.debian.org/search?keywords=fio .



**Ubuntu:**



Starting with Ubuntu 10.04 LTS (aka “Lucid Lynx”), fio packages are part of the Ubuntu “universe” repository. https://packages.ubuntu.com/search?keywords=fio .



**Red Hat, Fedora, CentOS & Co:**



Starting with Fedora 9/Extra Packages for Enterprise Linux 4, fio packages are part of the Fedora/EPEL repositories. https://packages.fedoraproject.org/pkgs/fio/ .



**Mandriva:**



Mandriva has integrated fio into their package repository, so installing on that distro should be as easy as typing `urpmi fio`.



**Arch Linux:**



An Arch Linux package is provided under the Community sub-repository: https://www.archlinux.org/packages/?sort=&q=fio



**Solaris:**



Packages for Solaris are available from OpenCSW. Install their pkgutil tool (http://www.opencsw.org/get-it/pkgutil/) and then install fio via `pkgutil -i fio`.



**Windows:**



Beginning with fio 3.31 Windows installers are available on GitHub at https://github.com/axboe/fio/releases. Rebecca Cran <[rebecca@bsdio.com](mailto:rebecca@bsdio.com)> has fio packages for Windows at https://bsdio.com/fio/ . The latest builds for Windows can also be grabbed from https://ci.appveyor.com/project/axboe/fio by clicking the latest x86 or x64 build and then selecting the Artifacts tab.



**BSDs:**



Packages for BSDs may be available from their binary package repositories. Look for a package “fio” using their binary package managers.



### 2.4. 验证



不管你使用的是哪种安装方法，安装之后的效果应该是一样的。



我们可以使用 `fio -v` 查看已安装 fio 的版本，正确输出版本信息就说明安装成功了。

```shell
$ fio -v
fio-3.35-2-g954b8
```





## 3. fio 的使用



fio 的使用非常简单，命令行用法如下：

```shell
$ fio [options] [jobfile] ...
```

这里面主要有两个需要关注的点：

* `options`：命令行选项，配置 fio 运行时的各种参数。
* `jobfile`：工作文件，可以有多个。其值可以为 `-`，则表示从标准输入读取。



工作文件 `jobfile` 中描述了 fio 要做什么。所以我们使用 fio 进行 I/O 测试前，必须先写至少一个工作文件。



下面分别来说，fio 进行 I/O 测试的常用选项，以及如何编写工作文件。



注意本文介绍的是仅仅是比较常用的部分，更多内容可以看 [ fio 官方文档](https://fio.readthedocs.io/en/latest/fio_doc.html#platforms)。



### 3.1. 常用选项





### 3.2. 如何编写工作文件





