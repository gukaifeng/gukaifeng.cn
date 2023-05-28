---
title: "使用 mdtest 测试文件系统元数据性能"
date: 2023-05-28 18:36:00
updated: 2023-05-29 00:30:00
categories: [技术杂谈]
tags: [ior,mdtest]
---





## 1. 什么是 IOR 和 mdtest ？



IOR（Integrated Benchmark of Parallel I/O）是一种基准测试应用程序，旨在测试并评估并行文件系统的性能。它包含多个测试模式，可以测量一系列文件 I / O 操作的速度和可扩展性。IOR 主要使用 MPI 软件库来实现并行 I/O 操作。它可以测试并行读取和写入各种文件访问模式（例如单个大文件、多个小文件）时的并行文件系统性能。 



mdtest 是一款基准测试程序，用来测试文件系统元数据操作的性能。它可以模拟大规模目录和文件的创建、重命名、删除等操作，通常是在高性能计算环境中使用。使用 mdtest 可以帮助用户评估存储系统文件操作的性能，以便更好地优化存储系统的设计和配置。它通常作为文件系统测试套件中的一部分，如 IOR、IOZone 和 FIO 等工具。 



* IOR 通常被用来测试大文件的读写性能。例如，可以使用 IOR 测试并行文件系统的顺序读写能力、随机访问能力、并行度和带宽。这对于优化大规模应用程序和大规模数据中心中存储层的性能非常重要。
* MDTest 通常被用来测试并行文件系统在处理许多小文件时的性能。例如，可以使用 mdtest 测试并行文件系统的每秒创建文件数，每秒删除文件数和每秒访问文件数，这对于大规模数据中心中的元数据性能优化很重要。



## 2. 安装 IOR 和 mdtest

本文环境为 CentOS 8.4。

### 2.1. 克隆 IOR 仓库



首先克隆 IOR 仓库，这个仓库里已经同时包含了 IOR 和 mdtest 工具：

```shell
git clone git@github.com:hpc/ior.git
```



并进入目录：



```shell
cd ior/
```





### 2.2. 切换到最新版本分支



截止撰写本文时（2023 年 5 月 28 日），IOR 的最新版本为 4.0，我们切到这个版本的分支：



```shell
git switch 4.0
```



### 2.3. 安装依赖

#### 2.3.1. 从 yum 源直接安装某些依赖



一些依赖包直接从 yum 源安装就可以：

```shell
yum install autoconf \
            automake \
            libtool  \
            pkg-config
```



#### 2.3.2. 安装 OpenMPI



还有一个依赖是，需要一个支持 MPI 的编译器，例如 OpenMPI。

可以尝试运行以下命令确认是否已安装 MPI：

   ```shell
   which mpicc
   which mpic++
   ```

如果看到系统内已经安装了 MPI 的话，就可以跳过这一步，否则就需要安装支持 MPI 的编译器，这里安装的是 OpenMPI。



建议从 OpenMPI 官网下载源码包，从源码安装。我们在 [Open MPI](https://www-lb.open-mpi.org/software/ompi/) 找到需要版本的 OpenMPI 的 tar 包的链接，我这里选择的是写此文章时的最新版 4.1.5：



```shell
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.5.tar.gz
```

然后解压并进入目录：



```shell
tar -zxvf openmpi-4.1.5.tar.gz 
```

```shell
cd openmpi-4.1.5/
```

配置：

```shell
./configure --prefix=/usr/local
```



编译并安装：



```shell
make -j`nproc`
```

```shell
sudo make install
```

到这里 OpenMPI 的安装就完成了，我们也可以验证一下：

```shell
$ which mpicc
/usr/local/bin/mpicc

$ which mpic++
/usr/local/bin/mpic++
```





### 2.4. 配置 configure



按照官方文档，先使用 `./bootstrap` 创建 `./configue` 文件：



```shell
./bookstarp
```

然后执行配置：



```shell
./configure
```



### 2.5. 编译并安装



依次执行：

```shell
make
```

```shell
make install
```



到这里就安装完成了，然后我们可以使用 `make check` 检查一下是否安装正确了：

```shell
$ make check
Making check in src
Making check in .
Making check in test
make  testlib testexample
  CC       lib.o
  CCLD     testlib
  CC       example.o
  CCLD     testexample
make  check-TESTS
PASS: testlib
PASS: testexample
============================================================================
Testsuite summary for ior 4.0.0rc2+dev
============================================================================
# TOTAL: 2
# PASS:  2
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
Making check in doc
make[1]: Nothing to be done for 'check'.
Making check in contrib
make[1]: Nothing to be done for 'check'.
make[1]: Nothing to be done for 'check-am'.
```

这里的输出信息有显示一共 2 个测试，通过 2 个，就说明没问题了。





### 2.6. 验证 ior 和 mdtest



我们直接在命令行输入 `ior -h` 和 `mdtest -h`，如果打印出帮助信息，就说明安装成功了。



这两个帮助信息篇幅较长，这小节就不演示了，我们在后面会详细说 `ior` 和 `mdtest` 的这些参数。





## 3. IOR 和 mdtest 的参数说明



### 3.1. IOR 的参数列表

命令 `ior -h` 可以打印出 IOR 的参数列表，如下：



```shell
$ ior -h
Synopsis ior

Flags
  -c, --collective              Use collective I/O
  -C                            reorderTasks -- changes task ordering for readback (useful to avoid client cache)
  -e                            fsync -- perform a fsync() operation at the end of each read/write phase
  -E                            useExistingTestFile -- do not remove test file before write access
  -F                            filePerProc -- file-per-process
  -g                            intraTestBarriers -- use barriers between open, write/read, and close
  -k                            keepFile -- don't remove the test file(s) on program exit
  -K                            keepFileWithError  -- keep error-filled file(s) after data-checking
  -m                            multiFile -- use number of reps (-i) for multiple file count
  -r                            readFile -- read existing file
  -R                            checkRead -- verify that the output of read matches the expected signature (used with -G)
  -u                            uniqueDir -- use unique directory name for each file-per-process
  -v                            verbose -- output information (repeating flag increases level)
  -w                            writeFile -- write file
  -W                            checkWrite -- check read after write
  -x                            singleXferAttempt -- do not retry transfer if incomplete
  -y                            dualMount -- use dual mount points for a filesystem
  -Y                            fsyncPerWrite -- perform sync operation after every write operation
  -z                            randomOffset -- access is to random, not sequential, offsets within a file
  -Z                            reorderTasksRandom -- changes task ordering to random ordering for readback
  --warningAsErrors             Any warning should lead to an error.
  --dryRun                      do not perform any I/Os just run evtl. inputs print dummy output

Optional arguments
  -a=POSIX                      API for I/O [POSIX|DUMMY|MPIIO|MMAP]
  -A=0                          refNum -- user supplied reference number to include in the summary
  -b=1048576                    blockSize -- contiguous bytes to write per task  (e.g.: 8, 4k, 2m, 1g)
  -d=0                          interTestDelay -- delay between reps in seconds
  -D=0                          deadlineForStonewalling -- seconds before stopping write or read phase
  -O stoneWallingWearOut=1           -- once the stonewalling timeout is over, all process finish to access the amount of data
  -O stoneWallingWearOutIterations=N -- stop after processing this number of iterations, needed for reading data back written with stoneWallingWearOut
  -O stoneWallingStatusFile=FILE     -- this file keeps the number of iterations from stonewalling during write and allows to use them for read
  -O minTimeDuration=0           -- minimum Runtime for the run (will repeat from beginning of the file if time is not yet over)
  -f=STRING                     scriptFile -- test script name
  -G=0                          setTimeStampSignature -- set value for time stamp signature/random seed
  -i=1                          repetitions -- number of repetitions of test
  -j=0                          outlierThreshold -- warn on outlier N seconds from mean
  -l, --dataPacketType=STRING   datapacket type-- type of packet that will be created [offset|incompressible|timestamp|random|o|i|t|r]
  -M=STRING                     memoryPerNode -- hog memory on the node  (e.g.: 2g, 75%)
  -N=-1                         numTasks -- number of tasks that are participating in the test (overrides MPI)
  -o=testFile                   testFile -- full name for test
  -O=STRING                     string of IOR directives (e.g. -O checkRead=1,GPUid=2)
  -Q=1                          taskPerNodeOffset for read tests use with -C & -Z options (-C constant N, -Z at least N)
  -s=1                          segmentCount -- number of segments
  -t=262144                     transferSize -- size of transfer in bytes (e.g.: 8, 4k, 2m, 1g)
  -T=0                          maxTimeDuration -- max time in minutes executing repeated test; it aborts only between iterations and not within a test!
  -X=0                          reorderTasksRandomSeed -- random seed for -Z option
  --randomPrefill=0             For random -z access only: Prefill the file with this blocksize, e.g., 2m
  --random-offset-seed=-1       The seed for -z
  -O summaryFile=FILE                 -- store result data into this file
  -O summaryFormat=[default,JSON,CSV] -- use the format for outputting the summary
  -O saveRankPerformanceDetailsCSV=<FILE> -- store the performance of each rank into the named CSV file.


Module POSIX

Flags
  --posix.odirect               Direct I/O Mode
  --posix.rangelocks            Use range locks (read locks for read ops)


Module DUMMY

Flags
  --dummy.delay-only-rank0      Delay only Rank0

Optional arguments
  --dummy.delay-create=0        Delay per create in usec
  --dummy.delay-close=0         Delay per close in usec
  --dummy.delay-sync=0          Delay for sync in usec
  --dummy.delay-xfer=0          Delay per xfer in usec


Module MPIIO

Flags
  --mpiio.showHints             Show MPI hints
  --mpiio.preallocate           Preallocate file size
  --mpiio.useStridedDatatype    put strided access into datatype
  --mpiio.useFileView           Use MPI_File_set_view

Optional arguments
  --mpiio.hintsFileName=STRING  Full name for hints file


Module MMAP

Flags
  --mmap.madv_dont_need         Use advise don't need
  --mmap.madv_pattern           Use advise to indicate the pattern random/sequential
```

IOR 有两类参数，一类是标记参数 Flags，一类是可选参数 Optional arguments。

这两类参数很好区别。Flags 参数是没有值的，例如 `-c`；而可选参数是有值的，例如 `-a=POSIX  `。我们下面逐个介绍。

\-



**Flags：**







\-

**Optional arguments：**







\-





剩下的参数是 IOR 和 mdtest 的通用参数，我们在 3.3 节介绍这些参数。

### 3.2. mdtest 的参数列表



命令 `mdtest -h` 可以打印出 mdtest 的参数列表，如下：



```shell
$ mdtest -h
Synopsis mdtest

Flags
  -C                            only create files/dirs
  -T                            only stat files/dirs
  -E                            only read files/dir
  -r                            only remove files or directories left behind by previous runs
  -D                            perform test on directories only (no files)
  -F                            perform test on files only (no directories)
  -k                            use mknod to create file
  -L                            files only at leaf level of tree
  -P                            print rate AND time
  --print-all-procs             all processes print an excerpt of their results
  -R                            random access to files (only for stat)
  -S                            shared file access (file only, no directories)
  -c                            collective creates: task 0 does all creates
  -t                            time unique working directory overhead
  -u                            unique working directory for each task
  -v                            verbosity (each instance of option increments by one)
  -X, --verify-read             Verify the data read
  --verify-write                Verify the data after a write by reading it back immediately
  -y                            sync file after writing
  -Y                            call the sync command after each phase (included in the timing; note it causes all IO to be flushed from your node)
  -Z                            print time instead of rate
  --allocateBufferOnGPU         Allocate the buffer on the GPU.
  --warningAsErrors             Any warning should lead to an error.
  --showRankStatistics          Include statistics per rank

Optional arguments
  -a=STRING                     API for I/O [POSIX|DUMMY]
  -b=1                          branching factor of hierarchical directory structure
  -d=./out                      directory or multiple directories where the test will run [dir|dir1@dir2@dir3...]
  -B=0                          no barriers between phases
  -e=0                          bytes to read from each file
  -f=1                          first number of tasks on which the test will run
  -G=-1                         Offset for the data in the read/write buffer, if not set, a random value is used
  -i=1                          number of iterations the test will run
  -I=0                          number of items per directory in tree
  -l=0                          last number of tasks on which the test will run
  -n=0                          every process will creat/stat/read/remove # directories and files
  -N=0                          stride # between tasks for file/dir operation (local=0; set to 1 to avoid client cache)
  -p=0                          pre-iteration delay (in seconds)
  --random-seed=0               random seed for -R
  -s=1                          stride between the number of tasks for each test
  -V=0                          verbosity value
  -w=0                          bytes to write to each file after it is created
  -W=0                          number in seconds; stonewall timer, write as many seconds and ensure all processes did the same number of operations (currently only stops during create phase and files)
  -x=STRING                     StoneWallingStatusFile; contains the number of iterations of the creation phase, can be used to split phases across runs
  -z=0                          depth of hierarchical directory structure
  --dataPacketType=t            type of packet that will be created [offset|incompressible|timestamp|random|o|i|t|r]
  --run-cmd-before-phase=STRING call this external command before each phase (excluded from the timing)
  --run-cmd-after-phase=STRING  call this external command after each phase (included in the timing)
  --saveRankPerformanceDetails=STRINGSave the individual rank information into this CSV file.


Module POSIX

Flags
  --posix.odirect               Direct I/O Mode
  --posix.rangelocks            Use range locks (read locks for read ops)


Module DUMMY

Flags
  --dummy.delay-only-rank0      Delay only Rank0

Optional arguments
  --dummy.delay-create=0        Delay per create in usec
  --dummy.delay-close=0         Delay per close in usec
  --dummy.delay-sync=0          Delay for sync in usec
  --dummy.delay-xfer=0          Delay per xfer in usec


Module MPIIO

Flags
  --mpiio.showHints             Show MPI hints
  --mpiio.preallocate           Preallocate file size
  --mpiio.useStridedDatatype    put strided access into datatype
  --mpiio.useFileView           Use MPI_File_set_view

Optional arguments
  --mpiio.hintsFileName=STRING  Full name for hints file


Module MMAP

Flags
  --mmap.madv_dont_need         Use advise don't need
  --mmap.madv_pattern           Use advise to indicate the pattern random/sequential
```

mdtest 有两类参数，一类是标记参数 Flags，一类是可选参数 Optional arguments。

这两类参数很好区别。Flags 参数是没有值的，例如 `-C`；而可选参数是有值的，例如 `-a=STRING  `。我们下面逐个介绍。



\-



**Flags：**

1. `-C`：仅创建文件/目录。即无此 Flag 时，mdtest 会将测试时创建的文件/目录删除，有此 Flag 则会保留这些文件/目录。
2. `-T`                            only stat files/dirs
3. `-E`                            only read files/dir
4. `-r`                            only remove files or directories left behind by previous runs
5. `-D`                            perform test on directories only (no files)
6. `-F`                            perform test on files only (no directories)
7. `-k`                            use mknod to create file
8. `-L`                            files only at leaf level of tree
9. `-P`                            print rate AND time
10. `--print-all-procs`             all processes print an excerpt of their results
11. `-R`                            random access to files (only for stat)
12. `-S`                            shared file access (file only, no directories)
13. `-c`                            collective creates: task 0 does all creates
14. `-t`                            time unique working directory overhead
15. `-u`                            unique working directory for each task
16. `-v`                            verbosity (each instance of option increments by one)
17. `-X`, `--verify-read`             Verify the data read
18. `--verify-write`                Verify the data after a write by reading it back immediately
19. `-y`                            sync file after writing
20. `-Y`                            call the sync command after each phase (included in the timing; note it causes all IO to be flushed from your node)
21. `-Z`                            print time instead of rate
22. `--allocateBufferOnGPU`         Allocate the buffer on the GPU.
23. `--warningAsErrors`             Any warning should lead to an error.
24. `--showRankStatistics`          Include statistics per rank





\-

**Optional arguments：**





\-





剩下的参数是 IOR 和 mdtest 的通用参数，我们在 3.3 节介绍这些参数。

### 3.3. IOR 和 mdtest 的通用参数



通过观察 ior -h 和 mdtest -h 的输出不难看出，两者下面有一部份参数是一样的，是二者皆有且含义一致的参数：



```shell
Module POSIX

Flags
  --posix.odirect               Direct I/O Mode
  --posix.rangelocks            Use range locks (read locks for read ops)


Module DUMMY

Flags
  --dummy.delay-only-rank0      Delay only Rank0

Optional arguments
  --dummy.delay-create=0        Delay per create in usec
  --dummy.delay-close=0         Delay per close in usec
  --dummy.delay-sync=0          Delay for sync in usec
  --dummy.delay-xfer=0          Delay per xfer in usec


Module MPIIO

Flags
  --mpiio.showHints             Show MPI hints
  --mpiio.preallocate           Preallocate file size
  --mpiio.useStridedDatatype    put strided access into datatype
  --mpiio.useFileView           Use MPI_File_set_view

Optional arguments
  --mpiio.hintsFileName=STRING  Full name for hints file


Module MMAP

Flags
  --mmap.madv_dont_need         Use advise don't need
  --mmap.madv_pattern           Use advise to indicate the pattern random/sequential
```





## 4. 使用 mpiexec 启动 IOR 或 mdtest



### 4.1. 什么是 mpiexec ?

MPI（Message Passing Interface）是一种编程标准和库，用于编写并行程序。MPI 实现包括多个节点的集群，这些节点可以相互通信和协调以完成计算任务。



`mpiexec` 可以启动 MPI 应用程序，并在多个节点中分配进程。它通常由 MPI 实现的一部分提供，可以在多个 MPI 实现中使用。`mpiexec` 对于运行 MPI 应用程序非常重要，因为它可以管理并行执行流程，并使编写并行程序变得更容易。



因为 IOR 和 mdtest 都是 MPI 应用程序，所以我们可以用 `mpiexec` 启动 IOR 或 mdtest。



**注：**  `mpiexec` 命令由 MPI 标准规定，并且是 MPI-2 和 MPI-3 标准的一部分。`mpirun` 命令是一种常见的 `mpiexec` 的实现，由某些 MPI 发行版提供。在我们这里（安装的 OpenMPI），使用这两个命令是完全一样的，本文倾向都使用 `mpiexec`，也会以 `mpiexec` 作示例。

### 4.2. mpiexec 的参数说明



如果你是按照本文的安装过程安装的 IOR、mdtest 以及依赖 OpenMPI，那么你的系统里应当已经有 `mpiexec` 了。





可以使用 `mpiexec -h` 命令查看其帮助信息：



```shell
$ mpiexec -h
mpiexec (OpenRTE) 4.1.5

Usage: mpiexec [OPTION]...  [PROGRAM]...
Start the given program using Open RTE

-c|-np|--np <arg0>       Number of processes to run
-h|--help <arg0>         This help message
   -n|--n <arg0>         Number of processes to run
-q|--quiet               Suppress helpful messages
-v|--verbose             Be verbose
-V|--version             Print version and exit

For additional mpirun arguments, run 'mpirun --help <category>'

The following categories exist: general (Defaults to this option), debug,
    output, input, mapping, ranking, binding, devel (arguments useful to OMPI
    Developers), compatibility (arguments supported for backwards compatibility),
    launch (arguments to modify launch options), and dvm (Distributed Virtual
    Machine arguments).

Report bugs to http://www.open-mpi.org/community/help/
```



### 4.3. 使用 mpiexec 启动 IOR





### 4.4. 使用 mpiexec 启动 mdtest
