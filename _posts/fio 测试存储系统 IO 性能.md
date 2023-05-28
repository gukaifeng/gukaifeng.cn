---
title: "fio 测试存储系统 I/O 性能"
date: 2023-05-26 00:25:00
updated: 2023-05-28 18:21:00
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

#### 2.2.2. 配置 fio



```shell
./configure
```



\-



这里其实是有一些选项可以配置的，不过一般默认情况已经能够满足绝大部分场景，这里就不解释了，就列个清单看看：

```
--prefix=               Use this directory as installation prefix
--cpu=                  Specify target CPU if auto-detect fails
--cc=                   Specify compiler to use
--extra-cflags=         Specify extra CFLAGS to pass to compiler
--build-32bit-win       Enable 32-bit build on Windows
--target-win-ver=       Minimum version of Windows to target (only accepts 7)
--enable-pdb            Enable Windows PDB symbols generation (needs clang/lld)
--build-static          Build a static fio
--esx                   Configure build options for esx
--enable-gfio           Enable building of gtk gfio
--disable-numa          Disable libnuma even if found
--disable-rdma          Disable RDMA support even if found
--disable-rados         Disable Rados support even if found
--disable-rbd           Disable Rados Block Device even if found
--disable-http          Disable HTTP support even if found
--disable-gfapi         Disable gfapi
--enable-libhdfs        Enable hdfs support
--enable-libnfs         Enable nfs support
--disable-libnfs        Disable nfs support
--disable-lex           Disable use of lex/yacc for math
--disable-pmem          Disable pmem based engines even if found
--enable-lex            Enable use of lex/yacc for math
--disable-shm           Disable SHM support
--disable-optimizations Don't enable compiler optimizations
--enable-cuda           Enable GPUDirect RDMA support
--enable-libcufile      Enable GPUDirect Storage cuFile support
--disable-native        Don't build for native host
--with-ime=             Install path for DDN's Infinite Memory Engine
--enable-libiscsi       Enable iscsi support
--enable-libnbd         Enable libnbd (NBD engine) support
--disable-xnvme         Disable xnvme support even if found
--disable-libblkio      Disable libblkio support even if found
--disable-libzbc        Disable libzbc even if found
--disable-tcmalloc      Disable tcmalloc support
--dynamic-libengines    Lib-based ioengines as dynamic libraries
--disable-dfs           Disable DAOS File System support even if found
--enable-asan           Enable address sanitizer
--seed-buckets=         Number of seed buckets for the refill-buffer
--disable-tls           Disable __thread local storage
```



#### 2.2.3. 编译 fio

依次执行：

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





不过要注意，**有一些参数只能在命令行中使用**，例如 `--minimal` 参数会最简化 fio 的输出，即便我们使用作业文件，此参数也只能体现到命令行中，例如：

```shell
$ fio --minimal myjobfile.job
```





## 4. fio 测试存储系统的相关属性（参数）

fio 的可选参数相当多，这里仅列出与测试存储系统性能相关的，更多选项与用法见 [fio 官方文档](https://fio.readthedocs.io/en/latest/fio_doc.html)。



下面给出 fio 测试存储系统性能的必要参数与可选值（注意这里给出的都是作业文件中的名称，在命令行中通常加前缀 `--`）：





1. `ioengine`=str：定义作业如何向文件发出 I/O。测试存储系统有以下几个常用取值：

   * `sync`：同步 I/O，每次操作都会等待 IO 完成后再进行下一次操作。适用于简单的 IO 测试，但是性能较差。
   * `psync`：同步 I/O，与 sync 类似，但是预提交缓存模式。该模式在 IO 操作时会先将数据存放在内存中的缓存区中，然后定期（或者每次 I/O 操作后）将缓存区的内容提交到磁盘中，以此提高磁盘 I/O 的性能。
   * `pvsync`：同步 I/O，类似 psync，但能够更好地利用预提交缓存机制，通常会有更好的性能表现。适用于需要测试高并发 I/O 性能的场景，如云计算、虚拟化等环境。
   * `io_uring`：使用 io_uring 实现异步 I/O，相比 libaio 和 posixaio（这里均未列出），具有更高的性能和更少的上下文切换。
   * `mmap`：同步 I/O，内存映射模式，将数据文件映射到内存中，以便快速访问文件内容。适用于对顺序读写的测试，可以实现很高的 IOPS。

2. `direct`=bool：布尔值（默认为 false）。如果为 true，则使用直接 I/O，反之使用缓冲 I/O。

3. `readwrite`=str, `rw`=str：指定 I/O 模式。常用取值有以下几个：

   * `read`：顺序读。
   * `write`：顺序写。
   * `randread`：随机读。
   * `randwrite`：随机写。
   * `rw,readwrite`：混合顺序读写，读写占比默认各 50%。
   * `randrw`：混合随机读写，读写占比默认各 50%。

4. `blocksize`=int\[,int][,int], `bs`=int\[,int][,int]：I/O 单元的块大小，以字节为单位（默认 4096）。

   以下是几个示例（注意可以用 k、m、g 后缀，分别表示 1024、1024\*1024、1024\*1024\*1024，不区分大小写）：

   * `bs=256k`：读、写和 trim 的块大小都是 256k 字节（注意 trim 概念这里并不涉及，后面将不再提及）。
   * `bs=8k,32k`：读的块大小为 8k 字节，写的块大小是 32k 字节。
   * `bs=,8k`：读的块大小为默认值（即 4096，4k 字节），写的块大小为 8k 字节。
   * `bs=8k,`：读的块大小为 8k 字节，写的块大小为默认值（即 4096，4k 字节）。

5. `size`=int：指定了测试文件的大小（字节），可以使用具体的大小（比如 8G）以及区间范围（比如 10G-20G）。在使用具体大小的情况下，文件将被创建为指定大小。在使用区间范围的情况下，文件将被随机创建为分布在该范围内的不同大小。

6. `iodepth`=int：I/O 深度，用于异步 I/O（默认值为 1，同步 I/O 即便设置也无效），指每个线程并发 I/O 请求的数量，也可以理解为发出多少个 I/O 请求后才等待响应。例如，如果使用 `--iodepth=32`，每个线程就同时发送 32 个 I/O 请求，然后等待 I/O 响应，再发送另外 32 个 I/O 请求。具体来说，这意味着 fio 将在测试期间创建多个线程，每个线程负责模拟多个客户端并发地读写文件。

7. `fliename`=str：指定测试文件的文件名（含路径），与 `filename_format` 二选一。参数 `size` 指定了这个文件要读写的大小。

8. `directory`=str：指定测试文件的目录。fio 会把 `directory` 和 `filename` 进行拼接，得出最终的测试文件路径。

9. `filename_format`=format：与 `filename` 二选一，这个支持在文件名中包含变量，变量名以 `$` 开头。如果 `filename` 和 `filename_format` 都未设定，就会以 `$jobname.$jobnum.$filenum` 为默认的文件名。

   支持的变量名有以下几个：

   1. `$jobname`：作业名。
   2. `$clientuid`：IP of the fio process when using client/server mode.
   3. `$jobnum`：The incremental number of the worker thread or process.
   4. `$filenum`：The incremental number of the file for that worker thread or process.

10. `thread`=int：fio 的并发线程总数，默认值为 1。仅用于异步 I/O，同步 I/O 配置了也无效，因为同步 I/O 只能顺序执行。

11. `numjobs`=int：fio 并发执行的 job 数量，默认值为 1。

   这里解释下参数 thread 和 numjobs 的区别。thread 指的是 fio 启动的线程**总数**，numjobs 指的是并发的 job 数。我们可以把 numjobs 理解为应用程序，有多少个应用程序在同时访问文件系统。例如 --thread=8，--numjobs=4, 可以理解为有 4 个应用程序在同时访问文件系统，每个应用程序里有 2 个线程。



\-



上面这些参数，已经基本可以满足大部分 fio 测试场景了。不过这里还有一些我认为比较有用的，可以用于调试等的参数（仅可以用在命令行的参数有标注，都命令行与作业文件都可以用的，命令行里的属性名字要添加前缀 `--`）：



1. `group_reporting`：控制 fio 的报告方式。默认情况下，每个作业的结果单独输出，你可以使用此选项将所有作业的结果合并输出（包含多个 job，以及 numjobs > 1 的情况）。此参数写在作业文件中的话，放在 `[global]` 字段下对于所有 job 有效，否则只将设定了此属性的 job 有效。

2. `runtime`：最大的运行时间。作业将在执行完成，或者到达此参数设定的最大时间时停止。有些时候我们无法预估一个作业的执行时间，使用此参数可以让我们轻松限定作业执行的最大时间。 

3. `time_based`：一般与 `--runtime` 搭配使用。如果设置，即使文件被完全读取或写入，fio 也会在指定的 `--runtime` 期间继续运行，它只会在运行时允许的情况下多次循环相同的工作负载。

4. `stonewall`：适用于一个作业文件内有多个测试 job 的场景，表示等待作业文件中的先前作业退出，然后再开始此作业，可用于在作业文件中插入序列化点。`stonewall` 石墙还意味着开始一个新的报告组 group_reporting。

5. `--output`：后接文件名，将 fio 的输出写入到指定文件中。**此参数只能在命令行使用。**

6. `--output-format`=format：指定输出信息的格式。**此参数只能在命令行使用。**有以下几种选择：

   * `normal`：默认值，就是默认的输出格式。
   * `minimal`：最小化输出，就是把所有测试数据挤在一坨，由分号 `;` 隔开，可以看做是表格（CSV 那种）。`--output-format=minimal` 与直接在命令行使用 `--minmal` 等价。
   * `terse`：与 minimal 基本相同，基于 CSV 格式，输出更详细一丁点，列出了更多指标（例如百分比延迟）。
   * `json`：以 json 的形式输出结果。
   * `json+`：与 `json` 基本一样，只是添加了完整了延迟 buckets。

7. `--eta`：作业执行时，在旁边显示预估的剩余时间。**此参数只能在命令行使用。**有三个值：

   1. `auto`：自动选择，是默认值。
   2. `always`：总是显示。
   3. `never`：从不显示。

8. `--debug`=type：显示更多的调试信息，可以设置多个项，每个项之间用逗号 `,` 隔开（如 `--debug=file,mem`）。**此参数只能在命令行使用。**有以下取值（这里都比较简单，而且也不是特别常用，就不翻译了=。=）：

   1. `process`：Dump info related to processes.
   2. `file`：Dump info related to file actions.
   3. `io`：Dump info related to I/O queuing.
   4. `mem`：Dump info related to memory allocations.
   5. `blktrace`：Dump info related to blktrace setup.
   6. `verify`：Dump info related to I/O verification.
   7. `random`：Dump info related to random offset generation.
   8. `parse`：Dump info related to option matching and parsing.
   9. `diskutil`：Dump info related to disk utilization updates.
   10. `job:x`：Dump info only related to job number x.
   11. `mutex`：Dump info only related to mutex up/down ops.
   12. `profile`：Dump info related to profile extensions.
   13. `time`：Dump info related to internal time keeping.
   14. `net`：Dump info related to networking connections.
   15. `rate`：Dump info related to I/O rate switching.
   16. `compress`：Dump info related to log compress/decompress.
   17. `steadystate`：Dump info related to steadystate detection.
   18. `helperthread`：Dump info related to the helper thread.
   19. `zbd`：Dump info related to support for zoned block devices.
   20. `all`：Enable all debug options.
   21. `?` or `help`：Show available debug options.

   







## 5. fio 测试存储系统的结果分析



我这里使用一个简单的、但包含了上面提到的所有常用属性的作业文件 `jobfile.fio`（虽然包含了上面提到的全部参数，但是有些参数之间是有冲突的，下面分析输出的时候会说道），其内容如下：

```ini
[global]
ioengine=pvsync
direct=1
iodepth=16
thread=4
numjobs=2
group_reporting

[my-example-job]
rw=readwrite
bs=8k
size=100m
filename_format=./mytestfile-$jobname-$jobnum-$filenum
```



我这里选择的是读写模式，这样可以尽可能的在输出中看到更多的内容，方便解释各个部分值的含义。



这些参数的含义上面都说过了，这里就不解释了。然后执行这个作业：

```shell
$ fio jobfile.fio
```

输出如下：

```shell
my-example-job: (g=0): rw=rw, bs=(R) 8192B-8192B, (W) 8192B-8192B, (T) 8192B-8192B, ioengine=pvsync, iodepth=16
...
fio-3.35-2-g954b8
Starting 2 threads
my-example-job: Laying out IO file (1 file / 100MiB)
my-example-job: Laying out IO file (1 file / 100MiB)
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
Jobs: 2 (f=2): [M(2)][85.7%][r=14.4MiB/s,w=14.0MiB/s][r=1840,w=1796 IOPS][eta 00m:01s]
my-example-job: (groupid=0, jobs=2): err= 0: pid=504766: Sun May 28 14:46:42 2023
  read: IOPS=2120, BW=16.6MiB/s (17.4MB/s)(99.2MiB/5987msec)
    clat (usec): min=85, max=8313, avg=438.34, stdev=1502.25
     lat (usec): min=85, max=8313, avg=438.40, stdev=1502.25
    clat percentiles (usec):
     |  1.00th=[   90],  5.00th=[   93], 10.00th=[   94], 20.00th=[   97],
     | 30.00th=[   99], 40.00th=[  101], 50.00th=[  103], 60.00th=[  108],
     | 70.00th=[  112], 80.00th=[  124], 90.00th=[  161], 95.00th=[  474],
     | 99.00th=[ 7767], 99.50th=[ 7832], 99.90th=[ 7898], 99.95th=[ 7963],
     | 99.99th=[ 8160]
   bw (  KiB/s): min=13280, max=43911, per=100.00%, avg=17155.55, stdev=4351.00, samples=22
   iops        : min= 1660, max= 5488, avg=2144.36, stdev=543.75, samples=22
  write: IOPS=2155, BW=16.8MiB/s (17.7MB/s)(101MiB/5987msec); 0 zone resets
    clat (usec): min=128, max=8837, avg=486.08, stdev=1503.94
     lat (usec): min=128, max=8837, avg=486.22, stdev=1503.94
    clat percentiles (usec):
     |  1.00th=[  135],  5.00th=[  139], 10.00th=[  139], 20.00th=[  143],
     | 30.00th=[  145], 40.00th=[  149], 50.00th=[  153], 60.00th=[  157],
     | 70.00th=[  163], 80.00th=[  176], 90.00th=[  223], 95.00th=[  635],
     | 99.00th=[ 7832], 99.50th=[ 7898], 99.90th=[ 8225], 99.95th=[ 8455],
     | 99.99th=[ 8717]
   bw (  KiB/s): min=12816, max=46450, per=100.00%, avg=17501.27, stdev=4709.65, samples=22
   iops        : min= 1602, max= 5806, avg=2187.64, stdev=588.67, samples=22
  lat (usec)   : 100=17.83%, 250=74.61%, 500=2.43%, 750=0.44%, 1000=0.20%
  lat (msec)   : 2=0.12%, 4=0.04%, 10=4.34%
  cpu          : usr=0.49%, sys=1.18%, ctx=25603, majf=0, minf=0
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=12697,12903,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
   READ: bw=16.6MiB/s (17.4MB/s), 16.6MiB/s-16.6MiB/s (17.4MB/s-17.4MB/s), io=99.2MiB (104MB), run=5987-5987msec
  WRITE: bw=16.8MiB/s (17.7MB/s), 16.8MiB/s-16.8MiB/s (17.7MB/s-17.7MB/s), io=101MiB (106MB), run=5987-5987msec

Disk stats (read/write):
  vda: ios=12320/12573, merge=0/0, ticks=5326/6099, in_queue=11425, util=98.35%
```

并且当前目录下会有两个 100MB 的文件（这是我们作业文件里配置的）。

```shell
$ du -sh *
4.0K    jobfile.fio
100M    mytestfile-my-example-job-0-0
100M    mytestfile-my-example-job-1-0
```

有两个文件是因为我在参数 `filename_format` 中使用了 `$jobnum` 变量，并且参数 `numjobs` 是 2。`$jobnum` 变量将这两个 job 的输出文件名区分开了，所以会有两个。如果不使用 `$jobnum`，那么应当就只有一个，因为两个 job 输出的文件名相同。



---





下面我们开始解释上面那一大坨输出！我们一行一行看 ~



\-



第 1 行：

```shell
my-example-job: (g=0): rw=rw, bs=(R) 8192B-8192B, (W) 8192B-8192B, (T) 8192B-8192B, ioengine=pvsync, iodepth=16
```

这里主要是执行作业的部分参数，分别有作业名字 job1，作业组 `g`，`bs` 的值（R 为读、W 为写、T 为 Trim 但我们这里不涉及，就不解释了），还有 `ioengine` 和 `iodepth`。这些参数的含义我们之前都解释过，这里就直接过了，很简单。



\-



第 2 行：

```shell
...
```

这个就是个分隔行，第 1 行是打印了这个作业的一些参数，从第 3 行开始就是真的作业测试输出了。



\-



第 3 行：

```shell
fio-3.35-2-g954b8
```

当前 fio 的版本号。



\-



第 4 行：

```shell
Starting 2 threads
```

表示正在启动 2 个线程。



这里是由我们配置的 `ioengine`、`thread` 参数和 `numjobs` 参数共同决定的。  
当 `ioengine` 配置的是同步 I/O，`thread` 参数无效，不会被读取，实际启动的线程数等于 `numjobs`，即个 job 有 1 个线程。  
当 `ioengine` 配置的是异步 I/O，实际启动的线程数等于 `thread`，`numjobs` 个 job 一共使用 `thread` 个线程。



在我们这个场景中，我们配置了同步 I/O `ioengine=pvsync`，`thread=5` 和 `numjobs=2`，所以这里 `thread` 参数是无效的，实际执行的线程数等于 `numjobs`，即为 2。





\-



第 5 - 6 行：

```shell
my-example-job: Laying out IO file (1 file / 100MiB)
my-example-job: Laying out IO file (1 file / 100MiB)
```

每行指的是 job 1 正在生成一个 IO 文件，大小为 100MiB。这里有两行是因为我们配置了 `numjobs=2`。



\-



第 7 - 8 行：

```shell
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
```

这是一个错误信息，因为我上面配置了使用同步 I/O（pvsync）又配置了 `iodepth` ，而 `iodepth` 参数仅对异步 I/O 有效，这里是告诉我们 `iodepth` 会被设置为最大为 1。



这里有两行是因为我们配置了 `numjobs=2`，每个 job 都输出了一次。



\-



第 9 行：

```shell
Jobs: 2 (f=2): [M(2)][85.7%][r=14.4MiB/s,w=14.0MiB/s][r=1840,w=1796 IOPS][eta 00m:01s]
```

这行信息是进度条，是实时变化的，其中含义如下：

* `Jobs: 2 (f=2)`：对测试的工作线程数量进行的汇报。在这个例子中，fio 启动了 2 个工作线程（job）。`f=2` 表示 `numjobs` 的值是 2，`Jobs` 的值和 `f` 的值相同代表着线程数与设定的相同。有这项输出是因为 fio 并不一定会按照设定参数启动线程，fio 可以自动调整工作线程的数量以适应测试负载和 I/O 模式。
* `[M(2)][85.7%]`：表示当前测试的状态和进度百分比。`[M(2)]` 表示当前测试正在进行中，总共有 2 个工作线程（job）。`[85.7%]`表示测试已经完成的百分比，这个值最终应当为 100%，但是由于我这里写入的文件很小，还没来得及更新这个值呢程序就结束了，输出了下面那些行的信息。
* `[r=14.4MiB/s,w=14.0MiB/s]`：表示读取(r)和写入(w)速度的统计信息。在这个例子中，读取速度为 14.4 MiB/s，写入速度为 14.0 MiB/s。
* `[r=1840,w=1796 IOPS]`：表示每秒读取(r)和写入(w)的 I/O 操作数量统计信息。在这个例子中，读取速度为 1840 IOPS，写入速度为 1796 IOPS。
* `[eta 00m:01s]`：表示预估的剩余测试时间(estimated time of arrival)。在这个例子中，测试的剩余时间预计为 00 分钟 01 秒，即测试应该在 1 秒内完成。



**注意上述信息是在 fio 测试进行时实时变化的，并不是最终的测试结果信息。**





\-



第 10 行：

```shell
my-example-job: (groupid=0, jobs=2): err= 0: pid=504766: Sun May 28 14:46:42 2023
```

这行很简单，分别是作业名 `my-example-job`，组 id `groupid=0`，作业数 `jobs=2`，错误数 `err= 0`，进程 `pid=504766`，和执行此测试的时间 `Sun May 28 14:46:42 2023`。



\-



第 11 - 21 行：

```shell
  read: IOPS=2120, BW=16.6MiB/s (17.4MB/s)(99.2MiB/5987msec)
    clat (usec): min=85, max=8313, avg=438.34, stdev=1502.25
     lat (usec): min=85, max=8313, avg=438.40, stdev=1502.25
    clat percentiles (usec):
     |  1.00th=[   90],  5.00th=[   93], 10.00th=[   94], 20.00th=[   97],
     | 30.00th=[   99], 40.00th=[  101], 50.00th=[  103], 60.00th=[  108],
     | 70.00th=[  112], 80.00th=[  124], 90.00th=[  161], 95.00th=[  474],
     | 99.00th=[ 7767], 99.50th=[ 7832], 99.90th=[ 7898], 99.95th=[ 7963],
     | 99.99th=[ 8160]
   bw (  KiB/s): min=13280, max=43911, per=100.00%, avg=17155.55, stdev=4351.00, samples=22
   iops        : min= 1660, max= 5488, avg=2144.36, stdev=543.75, samples=22
```

这些都是读操作的测试结果。

这里的第一行是总体概括的信息，有以下内容：

- `IOPS=2120`：表示 IO 操作每秒的数量，即 I/O Operations Per Second，单位为 次/秒。
- `BW=16.6MiB/s（17.4MB/s）`：表示 IO 操作的带宽，即 Bandwidth，也就是数据传输速度，以 MiB/s 或 MB/s 为单位。
- `(99.2MiB/5987msec)`：表示本次读取操作传输的数据量和所花费的时间，其中数据量为 99.2MiB，时间为 5987 毫秒。

然后下面各项分别是（单位都是 **usec**，即微秒）：

* `clat` 指标是“完成时间”（Completion Latency）的缩写，代表的是 IO 操作完成所花费的时间，不包括 IO 队列等待时间。

* `lat` 指标是“延迟”（Latency）的缩写，代表的是 IO 操作完成所花费的总时间，包括 IO 队列等待时间和完成时间。

  `clat`通常用于衡量存储设备的性能，因为设备的响应速度是关键因素。而 `lat` 则更加适用于衡量整个系统的性能，因为它考虑了所有相关因素。

  `clat` 指标所显示的数值比 `lat` 要小，因为它不包括 IO 队列等待时间。但在由于本例中使用的是同步 IO，不需要等待 I/O 队列，所以 `clat` 和 `lat` 的值基本一样。

* `clat percentiles`：一组 `clat` 的百分位数的统计指标，用于显示I/O操作完成时间延迟（Completion Latency）的不同分布情况。

* `bw`： 带宽。

* `iops`：这个没啥好说的，就是每秒的 IO 次数。



然后具体到每个项，还有一些细分的值：

* `min`：最小值。
* `max`：最大值。
* `avg`：平均值。
* `stdev`：标准差。
* `per`：使用的百分数。`bw `中为 100.00% 表示 100% 的带宽分布在结果中。
* `samples`：表示测试期间完成 I/O 操作的次数，即测试的样本数量。这里的 22 表示整个测试过程完成的完整 I/O 操作次数为 22 次。





\-



第 22 - 32 行：

```shell
  write: IOPS=2155, BW=16.8MiB/s (17.7MB/s)(101MiB/5987msec); 0 zone resets
    clat (usec): min=128, max=8837, avg=486.08, stdev=1503.94
     lat (usec): min=128, max=8837, avg=486.22, stdev=1503.94
    clat percentiles (usec):
     |  1.00th=[  135],  5.00th=[  139], 10.00th=[  139], 20.00th=[  143],
     | 30.00th=[  145], 40.00th=[  149], 50.00th=[  153], 60.00th=[  157],
     | 70.00th=[  163], 80.00th=[  176], 90.00th=[  223], 95.00th=[  635],
     | 99.00th=[ 7832], 99.50th=[ 7898], 99.90th=[ 8225], 99.95th=[ 8455],
     | 99.99th=[ 8717]
   bw (  KiB/s): min=12816, max=46450, per=100.00%, avg=17501.27, stdev=4709.65, samples=22
   iops        : min= 1602, max= 5806, avg=2187.64, stdev=588.67, samples=22
```

这些都是写操作的测试结果，各个项的含义与上面说过的读操作的一一对应，就不再重复解释了。



\-



第 33 - 34 行：

```shell
  lat (usec)   : 100=17.83%, 250=74.61%, 500=2.43%, 750=0.44%, 1000=0.20%
  lat (msec)   : 2=0.12%, 4=0.04%, 10=4.34%
```

第一行中的 `lat (usec)` 表示延迟时间的单位为微秒（usec），后面的数字表示各个延迟时间区间所占的比例。例如，上述输出结果中的 `100=17.83%, 250=74.61%, 500=2.43%` 表示完成时间延迟在 100 微秒以下的 I/O 操作有 17.83%；延迟在 100～250 微秒之间的操作有 74.61%，延迟在 250～500 微秒之间的操作有 2.43%。



第二行中 `lat (msec)` 表示延迟时间的单位为毫秒（msec），后面的数字表示不同的延迟时间区间所占的比例。例如，上述输出结果中的`2=0.12%, 4=0.04%, 10=4.34%”` 表示完成时间延迟在 2 毫秒以下的 I/O 操作有 0.12%；延迟在 2 ~ 4 毫秒之间的操作有 0.04%；延迟在 4 毫秒以上但 10 毫秒以下的操作有 4.34%。



这些数据可以帮助我们了解 I/O 操作的完成时间延迟的分布情况，确定延迟时间的一些关键点，如 99th 百分位数或平均延迟时间等，以评估存储设备或系统的性能是否符合预期，并进行性能优化。



\-



第 35 行：

```shell
  cpu          : usr=0.49%, sys=1.18%, ctx=25603, majf=0, minf=0
```

在fio测试输出结果中，这一行是用来描述测试过程中 CPU 使用情况的，其中：

- `usr` 表示用户 CPU 时间的占比。在这里，"usr=0.49%" 表示完成 I/O 操作的过程中，用户态 CPU 总共使用了 0.49% 的时间。

- `sys` 表示内核 CPU 时间的占比。在这里，"sys=1.18%" 表示完成 I/O 操作的过程中，内核态 CPU 总共使用了 1.18% 的时间。

- `ctx` 表示上下文切换的次数。在这里，"ctx=25603" 表示完成 I/O 操作的过程中，操作系统的上下文切换次数共计 25603 次。

- `majf` 表示发生的主要页故障的数量，当进程访问不在主存中的页时需要主要页故障进行处理。在这里，"majf=0" 表示完成 I/O 操作的过程中，没有发生主要页故障。

- `minf` 表示发生的次要页故障的数量，它是当操作系统访问到磁盘上的虚拟内存页时发生的。在这里，"minf=0" 表示完成 I/O 操作的过程中，没有发生次要页故障。 

  

这些数据可以帮助我们评估测试环境中 CPU 的使用情况，以及系统的稳定性。在进行性能测试时，需要考虑测试过程中的各种因素，例如 CPU、磁盘、内存等的负载情况，并根据需要进行相应的调整，以获得更加准确的测试结果。





\-



第 36 - 40 行：

```shell
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=12697,12903,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=16
```

这段输出描述了 I/O 操作深度和 I/O 请求的分布情况。



* `depths` 列出了请求队列中处理 I/O 请求时所使用的深度。例如，`depths` 中的 “1=100.0%”，表示所有的 I/O 请求都是深度为 1 的。

* `submit` 表示针对不同深度的 I/O 请求的数量。例如，`submit` 中的 "4=100.0%" 表示所有的 I/O 请求的深度都是 4。

  当 fio 运行时，会创建一个 I/O 请求队列，其中包含了待处理的 I/O 请求。队列中的 I/O 请求按照深度级别（iodepth level）分组，通常每个深度级别都有一定数量的 I/O 请求。`submit` 则表示在 I/O 请求队列中最大的同时启动的请求数量，也就是说，每次最多启动多少个 I/O 请求同时加入后端处理队列。其值由 fio 的内部算法决定，具体取决于 I/O 操作的类型、深度、数据文件大小等因素。

* `complete` 表示已完成处理并通过返回数据来完成读操作或写操作的请求数。在这个示例中，`complete` 中的 “4=100.0%” 表示所有的请求都已经被完成处理。

* `issued rwts` 里 "total" 的值表示总共发出了 12697 个读、12903 个写请求。"short" 表示长度非常短的请求，可能是被操作系统合并或去重后的结果。"dropped" 表示在处理的时候，不需要实际发出的I/O 请求数量，这里的数量为 0。

* `latency` 列出了测试的延迟，包括目标、窗口和百分位数、深度信息。

  * `target` 表示测试设置的延迟目标。在这里，目标被设置为 0，即最小化延迟。
  * `window` 表示在运行测试期间，记录延迟的时间窗口大小。在这里，窗口大小被设置为 0，即记录所有延迟时间。
  * `percentile` 是指多少百分比的 I/O 操作完成时间在某个特定时间内。在这里，percentile 被设置为 100%，即记录所有 I/O 操作的完成时间。
  * `depth` 列出了要记录的最大深度，即要记录最大深度为 16 的 I/O 操作延迟时间。





\-



第 42 - 44 行：

```shell
Run status group 0 (all jobs):
   READ: bw=16.6MiB/s (17.4MB/s), 16.6MiB/s-16.6MiB/s (17.4MB/s-17.4MB/s), io=99.2MiB (104MB), run=5987-5987msec
  WRITE: bw=16.8MiB/s (17.7MB/s), 16.8MiB/s-16.8MiB/s (17.7MB/s-17.7MB/s), io=101MiB (106MB), run=5987-5987msec
```

这段输出描述了 fio 测试的运行状态和结果：

* "Run status group 0 (all jobs)" 表示所有测试任务组的状态和结果，因为只有一个组 0，所有任务都在这个组里。
* `READ` 和 `WRITE` 分别表示读和写的测试任务组。
* `bw` 表示每秒的传输速率（带宽），单位是MiB/s（MB/s）。
* `io` 表示总的数据传输量，单位是 MiB（MB）。
* `run` 表示测试的运行时间，单位是 msec（毫秒）。



在括号中的数字表示速率或持续时间的范围，例如 "16.6MiB/s-16.6MiB/s" 表示测试期间传输速率保持稳定。



\-



第 46 - 47 行：

```shell
Disk stats (read/write):
  vda: ios=12320/12573, merge=0/0, ticks=5326/6099, in_queue=11425, util=98.35%
```

这段输出描述了磁盘的统计信息，包括读取和写入操作的 I/O 个数、合并操作的次数、处理 I/O 的时间（以 ticks 表示）、等待处理的 I/O 个数以及磁盘的利用率。

* "vda" 表示磁盘的名称。可以用 `fdisk` 命令查看我们的磁盘名称，例如我这里磁盘名称就是 "vda"：

  ```shell
  $ sudo fdisk -l
  Disk /dev/vda: 60 GiB, 64424509440 bytes, 125829120 sectors
  ...
  ```

  

* `ios` 表示读取和写入操作的 I/O 个数。对于此示例，读取操作是 12320，写入操作是 12573。

* `merge` 表示合并操作的次数，在此示例中为 0。

* `ticks` 表示处理每个 I/O 请求所需的时间，以毫秒为单位。在此示例中，读操作处理时间为 5326 毫秒，写操作为 6099 毫秒。

* `in_queue` 表示等待处理的 I/O 数量。在此示例中，等待处理的 I/O 数量为 11425。

* `util` 表示磁盘的利用率。在此示例中，磁盘利用率为 98.35%，表明磁盘的读取和写入吞吐量很高，几乎处于满负荷状态。





## 6. 附录：常用的作业文件模板



这里给出一些常用的，fio 测试**文件系统**性能的作业文件。



我这里同步 I/O 使用 **pvsync**，异步 I/O 使用 **io_uring**。

### 6.1. 同步 I/O

```ini
[global]
ioengine=pvsync
direct=1
iodepth=32
bs=4k
rw=randwrite
runtime=60
time_based
size=1G
numjobs=1
group_reporting

[write-test]
directory=/path/to/test_directory
filename=testfile
stonewall
```





### 6.2. 异步 I/O

```ini
[global]
ioengine=io_uring
iodepth=32
runtime=30s
rw=randwrite
bs=4k
size=1G
numjobs=4

[asyncio_threads]
name=asyncio-threads
ioengine=io_uring
iodepth=32
bs=4k
size=1G
numjobs=4
thread
```



### 6.3. 扩展：使用 fio 创建指定数量的文件



这个场景其实和本文主题关系不大，但是是我在实际工作中会用到的，就是直接往文件系统里写数据。



作业文件如下：



```ini
[create-files]
directory=/mnt/data/
numjobs=2
nrfiles=100
filesize=1M
rw=write
bs=4k
direct=1
```

主要的意思就是启动 `numjobs` 个任务，每个任务在目录 `directory` 下写入 `nrfiles` 个文件，每个文件大小为 `filesize`，总共写入 `numjobs` * `nrfiles` 个文件。



不过这个就是偶尔用用，而且 fio 不太好删这些创建出来的文件，我一般都手动删除。



如果常用关于大量文件的创建、删除、修改等元数据操作的，可能其他工具更合适，例如 mdtest。

