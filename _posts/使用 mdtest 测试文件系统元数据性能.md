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



TODO



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

这两类参数很好区别。Flags 参数是没有值的，例如 `-C`；而 Optional arguments 是有值的，例如 `-a=STRING  `。我们下面逐个介绍。



\-



**Flags：**



| No   | Flag                       | 含义                                                         |
| ---- | -------------------------- | ------------------------------------------------------------ |
| 1    | `-C`                       | 仅创建文件/目录。即无此 Flag 时，`mdtest` 会将测试时创建的文件/目录删除，有此 Flag 则会保留这些文件/目录。 |
| 2    | `-T`                       | 仅统计文件/目录。                                            |
| 3    | `-E`                       | 仅读取文件/目录。                                            |
| 4    | `-r`                       | 仅删除由之前的测试留下的文件或目录。注意使用时要保留之前测试的参数，才能准确删除期望的内容。 |
| 5    | `-D`                       | 对目录进行性能测试（不涉及文件），否则每个目录内会一半是文件一半是目录。注意是删除了文件测试，而不是把文件测试改为目录测试（比如原来某个目录下有 10 个文件、10 个目录和 1 个下一层结构的父目录，那么设置此参数后，将变为 10 个目录和 1 个下一层结构的父目录，不再有文件，而不是 20 个目录）。 |
| 6    | `-F`                       | 对文件进行性能测试（不涉及目录），否则每个目录内会一半是文件一半是目录。注意点同上。 |
| 7    | `-k`                       | 使用 `mknod` 创建文件。                                      |
| 8    | `-L`                       | 文件仅在目录树的叶子层。                                     |
| 9    | `-P`                       | 打印速率和时间。                                             |
| 10   | `--print-all-procs`        | 所有进程都打印其结果的摘要。                                 |
| 11   | `-R`                       | 随机访问文件（仅用于统计）。                                 |
| 12   | `-S`                       | 共享文件访问（只有文件，没有目录）。                         |
| 13   | `-c`                       | collective creates: task 0 does all creates.                 |
| 14   | `-t`                       | time unique working directory overhead.                      |
| 15   | `-u`                       | 每个任务一个工作目录。                                       |
| 16   | `-v`                       | 增加输出的详细程度。增加命令行上 `-v` 的数量会使详细程度更高，一共 6 个级别。 <br />* `0`：默认值；只显示基本要素。<br />* `1`：max clock deviation, participating tasks, free space, access pattern, commence/verify access notification with time.<br />* `2`：rank/hostname, machine name, timer used, individual repetition performance results, timestamp used for data signature.<br />* `3`：full test details, transfer block/offset compared, individual data checking errors, environment variables, task writing/reading file name, all test operation times.<br />* `4`：task id and offset for each transfer.<br />* `5`：each 8-byte data signature comparison (WARNING: more data to STDOUT than stored in file, use carefully). |
| 17   | `-X` <br />`--verify-read` | 验证读取的数据。                                             |
| 18   | `--verify-write`           | 写入后立即读回数据来验证数据。                               |
| 19   | `-y`                       | 写入完成后 sync 文件。                                       |
| 20   | `-Y`                       | 在每个阶段后调用 sync 命令（包含在计时中；注意这会导致从你的结点 flush 所有 IO）。 |
| 21   | `-Z`                       | 打印时间，而不是速率。                                       |
| 22   | `--allocateBufferOnGPU`    | 在 GPU 上分配缓冲区。                                        |
| 23   | `--warningAsErrors`：      | 任何警告都会导致错误（即将所有警告都视为错误）。             |
| 24   | `--showRankStatistics`     | 包括每个排名的统计信息。                                     |



\-

**Optional arguments：**

| NO   | Optional arguments                    | 含义                                                         |
| ---- | ------------------------------------- | ------------------------------------------------------------ |
| 1    | `-a`=STRING                           | I/O 的 API，取值 `POSIX` 或 `DUMMY`。                        |
| 2    | `-b`=1                                | 层次目录结构的分支因子。                                     |
| 3    | `-d`=./out                            | 运行测试的目录，可以有多个，用`@` 隔开，如 `-d=./out1@test/out2@~/out3`。 |
| 4    | `-B`=0                                | no barriers between phases.                                  |
| 5    | `-e`=0                                | 从每个文件读取的字节数。                                     |
| 6    | `-f`=1                                | 测试将运行的任务的起始编号。                                 |
| 7    | `-G`=-1                               | 读/写缓冲区中数据的偏移量，如果未设置，则使用随机值。        |
| 8    | `-i`=1                                | 测试将运行的迭代次数。                                       |
| 9    | `-I`=0                                | 每个目录中的项目数。                                         |
| 10   | `-l`=0                                | 测试将运行的任务的最后编号。                                 |
| 11   | `-n`=0                                | 每个进程都会 创建/统计/读取/删除 目录和文件。                |
| 12   | `-N`=0                                | 每个文件/目录操作之间的任务步长（local=0；设置为 1 以避免客户端缓存）。 |
| 13   | `-p`=0                                | 迭代前延迟（以秒为单位）。                                   |
| 14   | `--random-seed`=0                     | `-R` 的随机数种子。                                          |
| 15   | `-s`=1                                | stride between the number of tasks for each test.            |
| 16   | `-V`=0                                | 详细程度值。与上面 Flag 中的 `-v` 一样，只是直接设定数字。   |
| 17   | `-w`=0                                | 创建每个文件后写入每个文件的字节数。                         |
| 18   | `-W`=0                                | 以秒为单位的数字；stonewall 计时器，写入尽可能多的秒数并确保所有进程执行相同数量的操作（目前仅在创建阶段和文件期间停止） |
| 19   | `-x`=STRING                           | StoneWallingStatusFile，这是一个包含创建阶段迭代次数的文件，可用于在多次运行中拆分阶段。 |
| 20   | `-z`=0                                | 层次目录结构的深度。                                         |
| 21   | `--dataPacketType`=t                  | 将要创建的数据包类型，`[offset|incompressible|timestamp|random|o|i|t|r]`。 |
| 22   | `--run-cmd-before-phase`=STRING       | 在每个阶段之前调用此外部命令（不计入计时）。                 |
| 23   | `--run-cmd-after-phase`=STRING        | 在每个阶段之后调用此外部命令（计入计时）。                   |
| 24   | `--saveRankPerformanceDetails`=STRING | 将各个排名信息保存到此 CSV 文件中。                          |



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



**注：**  `mpiexec` 命令由 MPI 标准规定，并且是 MPI-2 和 MPI-3 标准的一部分。`mpirun` 命令是一种常见的 `mpiexec` 的实现，由某些 MPI 发行版提供。在新版本的 OpenMPI 中（我们上边安装的就是新版），这两个命令已经完全等价了，但在旧版本中，`mpiexec` 能够支持一些 `mpirun` 不支持的选项。本文以 `mpiexec` 作示例。

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



| 序号 | 参数                         | 含义                           |
| ---- | ---------------------------- | ------------------------------ |
| 1    | `-c` `-np` `--np` `-n` `--n` | 指定启动的 MPI 进程数。        |
| 2    | `-h`                         | 打印帮助信息。                 |
| 3    | `-q`                         | 安静模式，有用的信息也不输出。 |
| 4    | `-v`                         | 冗余模式，会打印更多的信息。   |
| 5    | `-V`                         | 打印版本信息。                 |





### 4.3. 使用 mpiexec 启动 IOR



TODO

### 4.4. 使用 mpiexec 启动 mdtest



使用 `mpiexec` 启动 `mdtest`，只需要在 `mpiexec` 的参数后接 `mdtest` 再接 `mdtest` 的参数即可。例如：



```shell
mpiexec -n 4 ./mdtest -I 10 -z 3 -u
```







## 5. mdtest 最常用的方法





### 5.1. 单独使用 mdtest

其实之前的小节列出的一大堆 `mdtest` 的参数，都不是很常用，最常用的应当只有下面这几个，掌握下面这些就足够了。





| No   | Flag | 含义                                                         |
| ---- | ---- | ------------------------------------------------------------ |
| 1    | `-C` | 仅创建文件/目录。即无此 Flag 时，`mdtest` 会将测试时创建的文件/目录删除，有此 Flag 则会保留这些文件/目录。 |
| 2    | `-r` | 仅删除由之前的测试留下的文件或目录。注意使用时要保留之前测试的参数，才能准确删除期望的内容。 |
| 3    | `-D` | 对目录进行性能测试（不涉及文件），否则每个目录内会一半是文件一半是目录。注意是删除了文件测试，而不是把文件测试改为目录测试（比如原来某个目录下有 10 个文件、10 个目录和 1 个下一层结构的父目录，那么设置此参数后，将变为 10 个目录和 1 个下一层结构的父目录，不再有文件，而不是 20 个目录）。 |
| 4    | `-F` | 对文件进行性能测试（不涉及目录），否则每个目录内会一半是文件一半是目录。注意点同上。 |
| 5    | `-L` | 文件仅在目录树的叶子层。                                     |
|      |      |                                                              |
|      |      |                                                              |



### 5.2. 与 mpiexec 结合使用



其实观察前面提到的 `mdtest` 参数就知道，`mdtest` 没有提供并发参数（如设置线程数），也就是说 `mdtest` 是单线程的。



`mpiexec` 基本上只用来并行执行 `mdtest`，即仅用到 `mpiexec` 的 `-n` 参数。





如：



```shell
mpiexec -n 2 mdtest -I 10 -z 3 -u
```

