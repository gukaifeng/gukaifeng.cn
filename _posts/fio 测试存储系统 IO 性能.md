---
title: "fio 测试存储系统 I/O 性能"
date: 2023-05-26 00:25:00
updated: 2023-05-27 16:13:00
categories: [技术杂谈]
tags: [fio]
---





## 1. 什么是 fio ？

fio（Flexible I/O Tester）是一个用于测试和评估 I/O 负载和性能的工具，具有高度的可配置性和灵活性。它可以模拟不同类型的负载，包括随机和顺序读写，随机和顺序混合读写，随机和顺序的深度嵌套目录结构等，同时可以产生各种不同的 I/O 负载模式，例如随机性、对齐和分配大小、顺序性等特征，以便更好地模拟实际应用程序中的 I/O 负载。fio 在评估和比较存储系统方面非常有用，例如硬盘驱动器、固态硬盘、闪存存储器、网络存储器等。 \- ChatGPT





本文主要介绍在 Linux 上使用 fio 测试**磁盘/文件系统**的性能，fio 的其他功能本文不关注。  
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



所以这小节几种 fio 安装方法都介绍，读者根据自己的使用场景选择就好，即简单使用可以直接包管理器安装，追求新版本特性则可以手动编译安装或者下载官方提供的 fio 二进制文件。





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
* `jobfile`：作业文件，可以有多个。其值可以为 `-`，则表示从标准输入读取。



作业文件 `jobfile` 中描述了 fio 要做什么。所以我们使用 fio 进行 I/O 测试前，必须先写至少一个作业文件。



下面分别来说，fio 进行 I/O 测试的方法，以及如何编写作业文件。



注意本文介只介绍比较常用的部分，更多内容可以看 [ fio 官方文档](https://fio.readthedocs.io/en/latest/fio_doc.html#platforms)。



### 3.1. 几种 fio 的使用方法



fio 有三种常用的使用方法。



1. 一种是仅使用命令行设定参数和选项，即：

   ```shell
   $ fio [options]
   ```

2. 一种是仅使用作业文件：

   ```shell
   $ fio [jobfile]
   ```

3. 也可以混合使用，就如一开始说的那样：

   ```shell
   $ fio [options] [jobfile] ...
   ```

   注意混合使用的时候，命令行的参数优先级大于作业文件中设定的，即如果命令行和作业文件中设定了相同的参数，则会使用命令行中指定的值。





### 3.2. 从一个例子开始



下面两个例子，使用 fio 测试文件系统顺序写大文件性能，两个例子完全等价。







#### 3.2.1. 仅使用命令行



```shell
fio --name=seqwrite --ioengine=sync --rw=write --bs=1M --size=10G --numjobs=1 --time_based --runtime=120s --eta=never --filename=/data/testfile.dat
```





#### 3.2.2. 仅使用作业文件



先编写一个 fio 作业文件，我这里名为 `seqwrite.fio`：

```ini
[global]
ioengine=sync
direct=1
time_based
eta=never

[seqwrite]
filename=/data/testfile.dat
rw=write
bs=1M
size=10G
numjobs=1
runtime=120s
```



然后在命令行使用作业文件进行测试：

```shell
fio seqwrite.fio
```

#### 3.2.3. 上述例子的理解





上面 3.2.1 和 3.2.2 的代码配置了一个顺序写的测试负载，写入一个名为 `testfile.dat` 的文件中，使用 1MB 的块大小，写入 10GB 的数据，持续运行 120 秒。



这里不解释这些参数的含义，不过我们可以知道几件事：

1. 命令行的参数与作业文件中的属性可以一一对应。例如命令行参数项 `--ioengine` 对应作业文件中的 `ifengine`；命令行中的测试名字 `--name` 对应作业文件中的中括号 `[]` 里的部分，如这里命令行参数 `--name=seqwrite` 对应作业文件中的 `[seqwrite]`（注意 `[global]` 是特殊的，指定的作业文件内所有测试的通用参数，如果下面某个作业的参数与 `[global]` 中指定的冲突，则以作业内配置的为准）。

2. 作业文件内可以有多组测试，例如我们可以把 `seqwrite.fio` 修改为：

   ```ini
   [global]
   ioengine=sync
   direct=1
   time_based
   eta=never
   
   [seqwrite]
   filename=/data/testfile.dat
   rw=write
   bs=1M
   size=10G
   numjobs=1
   runtime=120s
   
   [randwrite]
   filename=/data/randtestfile.dat
   rw=randwrite
   bs=1M
   size=10G
   numjobs=1
   runtime=120s
   ```

   这样就在作业文件内增加了一组随机写测试。

3. `filename` 属性指定的文件（目录），应当在我们要测试的磁盘/文件系统内，测试的才是指定存储系统的性能。



还有一些用法这里没有介绍，因为我认为不具备什么价值（很少使用），简单列出一下：



1. 仅命令行也可以进行多组测试，但不建议使用。因为这样使用会导致命令语句非常长，参数非常多，而且很容易写错，缺点很多。
2. fio 同时指定多个作业文件，每个作业文件用 `-f` 参数指定，例如 `fio -f jobfile1.fio -f jobfile2.fio`，但也不常使用。主要原因一方面是复杂度增加，一方面是多个作业文件是并发执行的，可能会出现资源竞争、测试数据互相干扰等问题。
3. 这里没有给出混合使用命令行参数和作业文件的例子，其使用场景在下面的建议中说明。







### 3.3. 几种 fio 使用方法的选取建议



通过前面的介绍，我这里总结了几点 fio 使用方法的选取建议：



1. 简单的、临时的测试，可以采取仅命令行的操作，方便快捷。
2. 当需要进行复杂的测试或者需要定期运行一组测试时，使用作业文件更加适合。
3. 当我们使用作业文件时，临时想修改某个属性（参数）又不想修改作业文件时，可以临时在命令行上添加此参数以覆盖作业文件内的。





## 4. fio 的测试存储系统的相关属性（参数）

fio 的可选参数相当多，这里仅列出测试存储系统性能的相关属性，更多选项与用法见 [fio 官方文档](https://fio.readthedocs.io/en/latest/fio_doc.html)。









## 5. fio 的测试存储系统的结果分析





