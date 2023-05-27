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

6. `iodepth`=int：I/O 深度，用于异步 I/O（默认值为 1，同步 I/O 不需要设置），指每个线程并发 I/O 请求的数量，也可以理解为发出多少个 I/O 请求后才等待响应。例如，如果使用 `--iodepth=32`，每个线程就同时发送 32 个 I/O 请求，然后等待 I/O 响应，再发送另外 32 个 I/O 请求。具体来说，这意味着 fio 将在测试期间创建多个线程，每个线程负责模拟多个客户端并发地读写文件。

7. `fliename`=str：指定测试文件的文件名（含路径）。如果未设定，则 fio 会在命令执行的目录生成读写文件（命名格式为 `jobname.jobnumber.filenumber`）。参数 size 指定了这个文件要读写的大小。

8. `thread`=int：fio 的并发线程总数，默认值为 1。

9. `numjobs`=int：fio 并发执行的 job 数量，默认值为 1。

   这里解释下参数 thread 和 numjobs 的区别。thread 指的是 fio 启动的线程**总数**，numjobs 指的是并发的 job 数。我们可以把 numjobs 理解为应用程序，有多少个应用程序在同时访问文件系统。例如 --thread=8，--numjobs=4, 可以理解为有 4 个应用程序在同时访问文件系统，每个应用程序里有 2 个线程。



\-



上面这些参数，已经基本可以满足大部分 fio 测试场景了。不过这里还有一些我认为比较有用的，可以用于调试等的参数（仅可以用在命令行的参数有标注，都命令行与作业文件都可以用的，作业文件里的属性名字要去掉前缀 `--`）：



1. `--group_reporting`：控制 fio 的报告方式。默认情况下，每个作业的结果单独输出，你可以使用此选项将所有作业的结果合并输出（包含多个 job，以及 numjobs > 1 的情况）。此参数写在作业文件中的话，放在 `[global]` 字段下对于所有 job 有效，否则只将设定了此属性的 job 有效。

2. `--output`：后接文件名，将 fio 的输出写入到指定文件中。**此参数只能在命令行使用。**

3. `--output-format`=format：指定输出信息的格式。**此参数只能在命令行使用。**有以下几种选择：

   * `normal`：默认值，就是默认的输出格式。
   * `minimal`：最小化输出，就是把所有测试数据挤在一坨，由分号 `;` 隔开，可以看做是表格（CSV 那种）。`--output-format=minimal` 与直接在命令行使用 `--minmal` 等价。
   * `terse`：与 minimal 基本相同，基于 CSV 格式，输出更详细一丁点，列出了更多指标（例如百分比延迟）。
   * `json`：以 json 的形式输出结果。
   * `json+`：与 `json` 基本一样，只是添加了完整了延迟 buckets。

4. `--eta`：作业执行时，在旁边显示预估的剩余时间。**此参数只能在命令行使用。**有三个值：

   1. `auto`：自动选择，是默认值。
   2. `always`：总是显示。
   3. `never`：从不显示。

5. `--debug`=type：显示更多的调试信息，可以设置多个项，每个项之间用逗号 `,` 隔开（如 `--debug=file,mem`）。**此参数只能在命令行使用。**有以下取值（这里都比较简单，而且也不是特别常用，就不翻译了=。=）：

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



我这里使用一个简单的、但包含了上面提到的所有常用属性的作业文件 `jobfile.fio`，其内容如下：

```ini
[global]
ioengine=pvsync
direct=1
iodepth=16
thread=4
numjobs=2
group_reporting

[job1]
rw=readwrite
bs=8k
size=100m
filename=./testfile
```

这些参数的含义上面都说过了，这里就不解释了。然后执行这个作业：



```shell
$ fio jobfile.fio
```

输出如下：

```
job1: (g=0): rw=rw, bs=(R) 8192B-8192B, (W) 8192B-8192B, (T) 8192B-8192B, ioengine=pvsync, iodepth=16
...
fio-3.35-2-g954b8
Starting 2 threads
job1: Laying out IO file (1 file / 100MiB)
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
note: both iodepth >= 1 and synchronous I/O engine are selected, queue depth will be capped at 1
Jobs: 2 (f=2): [M(2)][85.7%][r=14.4MiB/s,w=14.0MiB/s][r=1838,w=1798 IOPS][eta 00m:01s]
job1: (groupid=0, jobs=2): err= 0: pid=503658: Sun May 28 02:00:53 2023
  read: IOPS=2127, BW=16.6MiB/s (17.4MB/s)(99.2MiB/5967msec)
    clat (usec): min=85, max=13235, avg=452.48, stdev=1454.81
     lat (usec): min=85, max=13235, avg=452.54, stdev=1454.80
    clat percentiles (usec):
     |  1.00th=[   91],  5.00th=[   94], 10.00th=[   96], 20.00th=[   99],
     | 30.00th=[  101], 40.00th=[  105], 50.00th=[  110], 60.00th=[  116],
     | 70.00th=[  130], 80.00th=[  159], 90.00th=[  281], 95.00th=[  848],
     | 99.00th=[ 7701], 99.50th=[ 7767], 99.90th=[ 7898], 99.95th=[ 7963],
     | 99.99th=[ 9634]
   bw (  KiB/s): min=13600, max=45056, per=100.00%, avg=17213.09, stdev=4516.09, samples=22
   iops        : min= 1700, max= 5632, avg=2151.64, stdev=564.51, samples=22
  write: IOPS=2162, BW=16.9MiB/s (17.7MB/s)(101MiB/5967msec); 0 zone resets
    clat (usec): min=130, max=10501, avg=468.26, stdev=1420.98
     lat (usec): min=130, max=10501, avg=468.38, stdev=1420.98
    clat percentiles (usec):
     |  1.00th=[  135],  5.00th=[  139], 10.00th=[  141], 20.00th=[  145],
     | 30.00th=[  147], 40.00th=[  151], 50.00th=[  157], 60.00th=[  163],
     | 70.00th=[  174], 80.00th=[  190], 90.00th=[  241], 95.00th=[  545],
     | 99.00th=[ 7701], 99.50th=[ 7832], 99.90th=[ 7963], 99.95th=[ 8094],
     | 99.99th=[10552]
   bw (  KiB/s): min=14064, max=47440, per=100.00%, avg=17569.45, stdev=4837.88, samples=22
   iops        : min= 1758, max= 5930, avg=2196.18, stdev=604.73, samples=22
  lat (usec)   : 100=12.29%, 250=77.64%, 500=4.44%, 750=0.71%, 1000=0.39%
  lat (msec)   : 2=0.12%, 4=0.20%, 10=4.20%, 20=0.01%
  cpu          : usr=0.32%, sys=1.42%, ctx=25671, majf=0, minf=0
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=12697,12903,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
   READ: bw=16.6MiB/s (17.4MB/s), 16.6MiB/s-16.6MiB/s (17.4MB/s-17.4MB/s), io=99.2MiB (104MB), run=5967-5967msec
  WRITE: bw=16.9MiB/s (17.7MB/s), 16.9MiB/s-16.9MiB/s (17.7MB/s-17.7MB/s), io=101MiB (106MB), run=5967-5967msec

Disk stats (read/write):
  vda: ios=12413/12674, merge=0/0, ticks=5565/6095, in_queue=11659, util=98.24%
```

并且当前目录下会有个 100MB 的文件 `testfile`（这是我们作业文件里配置的）：

```shell
$ du -sh *
4.0K    jobfile.fio
100M    testfile
```



下面我们开始解释上面那一大坨输出！



