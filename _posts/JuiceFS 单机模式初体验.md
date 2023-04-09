---
title: "JuiceFS 单机模式初体验"
date: 2023-04-09 09:32:00
updated: 2023-04-09 09:32:00
categories: [分布式文件系统]
tags: [JuiceFS,分布式文件系统]
---





**本文不涉及分布式系统，仅在单机模式下简单体验 JuiceFS。**



## 1. 什么是 JuiceFS ？

JuiceFS 是一款面向云原生设计的高性能分布式文件系统。

JuiceFS 是跨平台的，可以在几乎所有主流架构上的各种操作系统中运行，包括且不限于 Linux、macOS、Windows。

JuiceFS 采用「数据」与「元数据」分离存储的架构，从而实现文件系统的分布式设计。

JuiceFS 中文件数据本身会被切分保存在[对象存储](https://juicefs.com/docs/zh/community/how_to_setup_object_storage#supported-object-storage)（例如 Amazon S3），而元数据则可以保存在 Redis、MySQL、TiKV、SQLite 等多种[数据库](https://juicefs.com/docs/zh/community/databases_for_metadata)中。

JuiceFS 在 [GitHub](https://github.com/juicedata/juicefs) 上开源，开源协议为 Apache 2.0。



此小节介绍内容节选自 [JucieFS 官方文档](https://juicefs.com/docs/zh/community/introduction/)。



## 2. 安装 JuiceFS 客户端



JuiceFS 客户端只有一个二进制文件，我们可以下载官方预编译的版本直接使用，也可以用源代码手动编译。



我这里直接使用官方预编译的二进制文件。如果你想自己编译，可以看 [JuiceFS 编译]()，最终都是得到一个二进制文件。



本文在 x86_64 架构上的 AlmaLinux 8.7 中进行。



下载我们需要的 JuiceFS 版本，更多版本见 [Releases · juicedata/juicefs](https://github.com/juicedata/juicefs/releases)。



```shell
mkdir juicefs && cd juicefs
wget https://github.com/juicedata/juicefs/releases/download/v1.0.4/juicefs-1.0.4-linux-amd64.tar.gz
tar -zxvf juicefs-1.0.4-linux-amd64.tar.gz
```

解压完成后，我们当前目录应该是这个样子的：

```shell
$ ll .
total 99292
-rwxr-xr-x. 1 gukaifeng gukaifeng 76303720 Apr  6 00:24 juicefs
-rw-rw-r--. 1 gukaifeng gukaifeng 25323848 Apr  8 22:42 juicefs-1.0.4-linux-amd64.tar.gz
-rw-r--r--. 1 gukaifeng gukaifeng    11358 Apr  6 00:08 LICENSE
-rw-r--r--. 1 gukaifeng gukaifeng    13213 Apr  6 00:08 README_CN.md
-rw-r--r--. 1 gukaifeng gukaifeng    14163 Apr  6 00:08 README.md
```

这里面有用的就是第一个，二进制文件 `juicefs`，我们将其移动到 `/usr/local/bin/` 目录下：



```shell
sudo mv juicefs /usr/local/bin/
```



现在直接在命令行输入 `juicefs`，输出帮助信息即为安装成功：



```
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



\-

卸载 JuiceFS 非常简单，因为 JuiceFS 只有一个二进制文件，我们刚刚把它放在了 `/usr/local/bin/` 目录下。

所以只要把这个二进制文件删掉，就算卸载了：

```shell
rm /usr/local/bin/juicefs
```





## 3. 创建文件系统



JuiceFS 文件系统由[「对象存储」](https://juicefs.com/docs/zh/community/how_to_setup_object_storage)和[「数据库」](https://juicefs.com/docs/zh/community/databases_for_metadata)共同驱动。

除了对象存储，JuiceFS 还支持使用本地磁盘、WebDAV 和 HDFS 等作为底层存储。

因此，可以使用本地磁盘和 SQLite 数据库快速创建一个单机文件系统，用以了解和体验 JuiceFS。



即在这里，**我们的 JuiceFS 文件系统将数据存在本地磁盘，将元数据存在 SQLite 数据库中。**





\-



JuiceFS 使用 [`format`](https://juicefs.com/docs/zh/community/command_reference#format) 子命令创建文件系统命令，基本格式为：

```shell
juicefs format [command options] META-URL NAME
```



1. `[command options]`：设定文件系统的存储介质，缺省则**默认使用本地磁盘**，Linux 中的默认路径为 `"$HOME/.juicefs/local"`。
2. `META-URL`：用来设置元数据存储，即数据库相关的信息，通常是数据库的 URL 或文件路径。
3. `NAME`：是文件系统的名称。



在我们目前简单体验的需求下，留空 `[command options]`，所以我们只需要设定两个值，即元数据存储`META-URL` 和文件系统名称 `NAME`。



\-



我这里为了干净一点，创建一个新文件夹并在之中操作后续的内容（读者怎么做看自己心情，并没有太大的影响）：



```shell
mkdir ~/juicefs-experience && cd ~/juicefs-experience
```





输入命令：

```shell
juicefs format sqlite3://myjfs.db myjfs
```

```
$ juicefs format sqlite3://myjfs.db myjfs
2023/04/08 23:18:47.985351 juicefs[2545] <INFO>: Meta address: sqlite3://myjfs.db [interface.go:401]
2023/04/08 23:18:47.986513 juicefs[2545] <INFO>: Data use file:///home/gukaifeng/.juicefs/local/myjfs/ [format.go:434]
2023/04/08 23:18:48.007846 juicefs[2545] <INFO>: Volume is formatted as {
  "Name": "myjfs",
  "UUID": "41c1b525-e685-48ed-8dc9-3ff596458b7a",
  "Storage": "file",
  "Bucket": "/home/gukaifeng/.juicefs/local/",
  "BlockSize": 4096,
  "Compression": "none",
  "TrashDays": 1,
  "MetaVersion": 1
} [format.go:471]
```

输出形如上面的信息，即表示 JuiceFS 文件系统创建成功了。



我们逐步分析上面的输出，其实主要就是 3 个 `<INFO>`，我们看看里都有什么。



* `Meta address`：元数据地址。元数据存储在 SQLite 数据库中，数据库文件是当前目录下的 `myjfs.db`。
* `Data`：数据库存储信息。由于我们没有指定存储介质，所以默认是本地存储。数据库目录为 `/home/gukaifeng/.juicefs/local/myjfs/`。
* `Volume is formatted as {...}`：卷的初始格式。这里给出了初始的元数据内容。



第三个 INFO 打印出的信息，就是 `myjfs.db` 数据库内表 `jfs_setting` 中 `format` 字段的值：



![myjfs.db](D:\Profession\hexo\_posts\JuiceFS 单机模式初体验\1.png)





到这里我们就创建好一个 JuiceFS 文件系统了。



\-



如果想要删除掉这个文件系统的话，根据上面的分析，只要分别删除元数据和数据就可以了。

在这里即为 SQLite 数据库文件 `myjfs.db` 和数据存储目录 `/home/gukaifeng/.juicefs/local/myjfs/`。



## 4. 挂载文件系统

JuiceFS 使用 [`mount`](https://juicefs.com/docs/zh/community/command_reference#mount)  子命令创建文件系统命令，基本格式为：

```shell
juicefs mount [command options] META-URL MOUNTPOINT
```

1. `[command options]`：挂载文件系统相关的选项。
2. `META-URL`：元数据存储。即数据库相关的信息，通常是数据库的 URL 或文件路径；
3. `MOUNTPOINT`：指定文件系统的挂载点。



由于 SQLite 是单文件数据库，挂载时要注意数据库文件的的路径，JuiceFS 同时支持相对路径和绝对路径。



我这里创建一个新目录作为即将的挂载点：

```shell
mkdir ~/jfs
```

然后挂载：

```shell
$ juicefs mount sqlite3:///home/gukaifeng/juicefs-experience/myjfs.db ~/jfs
2023/04/09 00:31:12.036698 juicefs[3206] <INFO>: Meta address: sqlite3:///home/gukaifeng/juicefs-experience/myjfs.db [interface.go:401]
2023/04/09 00:31:12.038812 juicefs[3206] <INFO>: Data use file:///home/gukaifeng/.juicefs/local/myjfs/ [mount.go:431]
2023/04/09 00:31:12.039007 juicefs[3206] <INFO>: Disk cache (/home/gukaifeng/.juicefs/cache/41c1b525-e685-48ed-8dc9-3ff596458b7a/): capacity (102400 MB), free ratio (10%), max pending pages (15) [disk_cache.go:94]
2023/04/09 00:31:12.045556 juicefs[3206] <INFO>: Create session 7 OK with version: 1.0.4+2023-04-06.f1c475d [base.go:289]
2023/04/09 00:31:12.046403 juicefs[3206] <INFO>: Prometheus metrics listening on 127.0.0.1:9567 [mount.go:161]
2023/04/09 00:31:12.046484 juicefs[3206] <INFO>: Mounting volume myjfs at /home/gukaifeng/jfs ... [mount_unix.go:181]
2023/04/09 00:31:12.046621 juicefs[3206] <WARNING>: setpriority: permission denied [fuse.go:427]
2023/04/09 00:31:12.542599 juicefs[3206] <INFO>: OK, myjfs is ready at /home/gukaifeng/jfs [mount_unix.go:45]

```

> 我这里出现了一个简单的错误：
>
> ```shell
> <FATAL>: fuse: fuse: exec: "/bin/fusermount": stat /bin/fusermount: no such file or directory [mount_unix.go:184]
> ```
>
> 就是缺少一个包 `fuse`，解决方法也很简单：
>
> ```shell
> sudo yum install fuse
> ```



上面的操作是前台挂载的，关掉窗口就没了，通常我们加参数 `-d` 或 `-background` 来后台挂载，通过日志查看输出：

```shell
juicefs mount -d sqlite3:///home/gukaifeng/juicefs-experience/myjfs.db ~/jfs
```

\-



卸载比较简单，使用子命令 `umount` 即可，例如：

```shell
juicefs umount ~/jfs
```





## 5. 查看我们的挂载信息

命令 `df -Th` 可以显示文件系统的类型、使用情况、挂载位置等信息（`T` 显示文件系统类型，`h` 表示把大小转成易读的单位）：

```shell
$ df -Th
Filesystem                 Type          Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs      3.8G     0  3.8G   0% /dev
tmpfs                      tmpfs         3.8G     0  3.8G   0% /dev/shm
tmpfs                      tmpfs         3.8G  9.0M  3.8G   1% /run
tmpfs                      tmpfs         3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/almalinux-root xfs            17G  2.1G   15G  13% /
/dev/sda1                  xfs          1014M  168M  847M  17% /boot
tmpfs                      tmpfs         774M     0  774M   0% /run/user/1000
JuiceFS:myjfs              fuse.juicefs  1.0P     0  1.0P   0% /home/gukaifeng/jfs
```

可以看到最后一行就是我们刚刚挂载的 JuiceFS 文件系统。在 `/home/gukaifeng/jfs` 目录内读写，即为在我们的 JuiceFS 文件系统上读写。



（Size 的值是 1.0P，我的磁盘是没有这么大的，结合官方的“这种形式等同于将容量几乎无限的对象存储接入到了本地计算机”描述，我认为可以把这个值理解为一种逻辑上的大小，或者是一个可以随时修改的上限值设定，而不是实际的物理存储大小。）



我们也可以看看我们的 JuiceFS 文件系统 `myjfs` 内有什么：

```shell
$ tree ~/jfs -ap
/home/gukaifeng/jfs
├── [-r--------]  .accesslog
├── [-r--------]  .config
├── [-r--r--r--]  .stats
└── [dr-xr-xr-x]  .trash

1 directory, 3 files
```

可以看到 `~/jfs` 目录并不是看空的，从名字上很容易看出文件或目录的含义：

* `.accesslog`：文件系统访问日志。
* `.config`：文件系统配置，json 格式的。
* `.stats`：文件系统统计信息。
* `.trash`：回收站，是个目录。



## 6. 进阶：将本地存储替换为对象存储

元数据仍由 SQLite 数据库存储，把数据的本地存储替换为对象存储，是一个更有实用价值的方案。





### 6.1. 准备一个对象存储



通常，创建对象存储需要两步：

1. 创建 **Bucket** 存储桶，得到 **Endpoint** 地址；
2. 创建 **Access Key ID** 和 **Access Key Secret**，即对象存储 API 的访问密钥。



我这里创建了一个[阿里云 OSS 对象存储](https://www.aliyun.com/product/oss)。



我这里选择的是外网访问的 Bucket Endpoint，因为我的 JuiceFS 机器不在阿里云上这个 OSS 的内网里。



我的 Bucket 名字叫做 juicefs-myjfs-test，读写权限为私有（即必须有 Access Key ID 和 Access Key Secret，且用户有此 Bucket 的权限）。



具体的 Bucket 创建操作、Access Key ID/Secret 的获取以及用户赋权问题，这里就不说了，如果你不明白，建议查看对应云厂商的操作手册。



我们最终应当持有形如下面这样的信息：

- **Bucket Endpoint**： juicefs-myjfs-test.oss-cn-beijing.aliyuncs.com
- **Access Key ID**：ABCDEFGHIJKLMNopqXYZ
- **Access Key Secret**：ZYXwvutsrqpoNMLkJiHgfeDCBA

（我这里写出的 Access Key ID 和 Access Key Secret 是无效的，你应该有你自己的。）



### 6.2. 创建并挂载文件系统

*注意哦，这里的创建的 JuiceFS 文件系统已经和之前的没有关系了，现在是创建一个新的了。*

*因为前面已经详细介绍过流程了，这里就加快速度。*



还是一样，为了不同的项目不会混在一起，我还是创建一个新文件夹，在里面做后面的事。

```shell
mkdir ~/juicefs-oss && cd ~/juicefs-oss
```

创建 JuiceFS 文件系统：

```shell
juicefs format --storage oss \
    --bucket  juicefs-myjfs-test.oss-cn-beijing.aliyuncs.com \
    --access-key ABCDEFGHIJKLMNopqXYZ \
    --secret-key ZYXwvutsrqpoNMLkJiHgfeDCBA \
    sqlite3:///home/gukaifeng/juicefs-oss/myjfs-oss.db myjfs-oss
```

在上述命令中，  
元数据数据库为 `/home/gukaifeng/juicefs-oss/myjfs-oss.db`，  
文件系统名称为 `myjfs-oss`，  
增加了对象存储相关的信息：

- `--storage`：设置存储类型，比如 `oss`、`s3` 等。
- `--bucket`：设置对象存储的 Endpoint 地址。
- `--access-key`：设置对象存储 API 访问密钥 Access Key ID。
- `--secret-key`：设置对象存储 API 访问密钥 Access Key Secret。



> 我这里创建文件系统的时候遇到了一个错误：
>
> 
>
> `The difference between the request time and the current time is too large`
>
> 
>
> **原因：**请求发起的时间与 OSS 服务器当前时间相差过大，OSS 判定该请求无效，返回报错。
>
> 
>
> **解决方案：**校对系统时间（OSS 的时区采用的是 GMT）。具体做法如下：
>
> 
>
> **Step 1：查看当前时区/时间信息**
>
> ```shell
> $ timedatectl
>                Local time: Sun 2023-04-09 04:19:28 EDT
>            Universal time: Sun 2023-04-09 08:19:28 UTC
>                  RTC time: Sun 2023-04-09 12:45:00
>                 Time zone: America/New_York (EDT, -0400)
> System clock synchronized: no
>               NTP service: active
>           RTC in local TZ: no
> ```
>
> **Step 2：校对时间**
>
> 
>
> 首先安装 `chrony`：
>
> ```shell
> sudo yum install chrony
> ```
>
> 修改 `chrony` 配置文件，注释掉 CentOS 的时间服务器，改成阿里的时间服务器：
>
> ```shell
> sudo vim /etc/chrony.conf
> ```
>
> ```text
> # pool 2.almalinux.pool.ntp.org iburst
> server ntp1.aliyun.com iburst
> server ntp2.aliyun.com iburst
> server ntp3.aliyun.com iburst
> server ntp4.aliyun.com iburst
> server ntp5.aliyun.com iburst
> server ntp6.aliyun.com iburst
> server ntp7.aliyun.com iburst
> ```
>
> 设置 `chronyd` 开机启动：
>
> ```text
> sudo systemctl enable chronyd
> ```
>
> 手动启动 `chronyd`：
>
> ```text
> sudo systemctl start chronyd
> ```
>
> 可以通过命令查看 `chronyd` 服务的状态，可以看到 `active (running)`，说明服务正常：
>
> ```text
> $ systemctl status chronyd
> ● chronyd.service - NTP client/server
>    Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
>    Active: active (running) since Sun 2023-04-09 16:33:34 CST; 6h ago
>      Docs: man:chronyd(8)
>            man:chrony.conf(5)
>   Process: 3658 ExecStopPost=/usr/libexec/chrony-helper remove-daemon-state (code=exited, status=0/SUCCESS)
>   Process: 3669 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
>   Process: 3663 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
>  Main PID: 3667 (chronyd)
>     Tasks: 1 (limit: 49276)
>    Memory: 1.0M
>    CGroup: /system.slice/chronyd.service
>            └─3667 /usr/sbin/chronyd
> 
> ```
>
> 
>
> **Step 3：修改时区为上海（可选）**
>
> ```shell
> timedatectl set-timezone Asia/Shanghai
> ```
>
> 这一步主要是为了自己看着方便，只要时间校对好，不用一定改成 GMT 时区的。
>
> 
>
> **Step 4：验证**
>
> 这里需要重新登录一下（重启系统、退出重登 SSH 都行）。
>
> 然后像 Step 1 一样使用 `timedatectl` 查看时区/时间信息：
>
> ```shell
> $ timedatectl
>                Local time: Sun 2023-04-09 22:45:48 CST
>            Universal time: Sun 2023-04-09 14:45:48 UTC
>                  RTC time: Sun 2023-04-09 14:45:48
>                 Time zone: Asia/Shanghai (CST, +0800)
> System clock synchronized: yes
>               NTP service: active
>           RTC in local TZ: no
> ```
>
> 可以看到时间已经校对好了，时区也成功改为上海了，并且 `System clock synchronized: yes` 表示时间会自动校对。
>
> 
>
> Over ~





创建文件系统的输出如下：

```shell
$ juicefs format --storage oss \
>     --bucket  juicefs-myjfs-test.oss-cn-beijing.aliyuncs.com \
>     --access-key ABCDEFGHIJKLMNopqXYZ \
>     --secret-key ZYXwvutsrqpoNMLkJiHgfeDCBA \
>     sqlite3:///home/gukaifeng/juicefs-oss/myjfs-oss.db myjfs-oss
2023/04/09 23:03:10.233010 juicefs[3819] <INFO>: Meta address: sqlite3:///home/gukaifeng/juicefs-oss/myjfs-oss.db [interface.go:401]
2023/04/09 23:03:10.234200 juicefs[3819] <INFO>: Data use oss://juicefs-myjfs-test/myjfs-oss/ [format.go:434]
2023/04/09 23:03:16.048988 juicefs[3819] <INFO>: Volume is formatted as {
  "Name": "myjfs-oss",
  "UUID": "e1fe6a19-3813-4aed-b34c-f11f813362d1",
  "Storage": "oss",
  "Bucket": "juicefs-myjfs-test.oss-cn-beijing.aliyuncs.com",
  "AccessKey": "LTAI5tFrCchDYqXdzmpmsovg",
  "SecretKey": "removed",
  "BlockSize": 4096,
  "Compression": "none",
  "KeyEncrypted": true,
  "TrashDays": 1,
  "MetaVersion": 1
} [format.go:471]
```

到这里就成功了。



现在我们回到阿里云的 OSS 控制台，找到我们这个 Bucket，可以看到里面有一个文件夹，名为 myjfs-oss，这正是我们上面命令里给此文件系统起的名字。

文件夹内还有一个名为 juicefs_uuid 的文件，这也是 JuiceFS 自己生成的（这里就不截图了）。



![2](D:\Profession\hexo\_posts\JuiceFS 单机模式初体验\2.png)





然后我们把文件系统 myjfs-oss 挂载到本地，首先还是创建一个用于挂载的新文件夹：

```shell
mkdir ~/jfs-oss
```

挂载：

```shell
juicefs mount -d sqlite3:///home/gukaifeng/juicefs-oss/myjfs-oss.db ~/jfs-oss
```

挂载成功会有输出信息提示。



然后使用 `df -Th` 命令检查挂载情况：

```shell
$ df -Th
Filesystem                 Type          Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs      3.8G     0  3.8G   0% /dev
tmpfs                      tmpfs         3.8G     0  3.8G   0% /dev/shm
tmpfs                      tmpfs         3.8G  9.0M  3.8G   1% /run
tmpfs                      tmpfs         3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/almalinux-root xfs            17G  2.1G   15G  13% /
/dev/sda1                  xfs          1014M  168M  847M  17% /boot
tmpfs                      tmpfs         774M     0  774M   0% /run/user/1000
JuiceFS:myjfs              fuse.juicefs  1.0P     0  1.0P   0% /home/gukaifeng/jfs
JuiceFS:myjfs-oss          fuse.juicefs  1.0P     0  1.0P   0% /home/gukaifeng/jfs-oss
```

可以看到最后一行，就是我们新挂载好的文件系统，到这里就成功了。





### 6.3. 简单测试一下



我们的文件系统挂载好以后，理论上，修改目录 `~/jfs-oss/` 中的内容后，阿里云 OSS 的 Bucket 中的内容应当有所变化。

在我们这个 myjfs-oss 文件系统中，JuiceFS 将元数据存储在 SQLite 数据库 `myjfs-oss.db` 中，将数据存储在我们的这个阿里云 OSS 的 Bucket 中。

  

我们简单创建一个文件：

```shell
$ cd ~/jfs-oss
$ vim juicefs_oss_test.txt
```

里面写上了 "Hello world !"，退出并保存。

打开阿里云 OSS 的这个 Bucket 的文件列表，可以看到是有变化的：



![3](D:\Profession\hexo\_posts\JuiceFS 单机模式初体验\3.png)



元数据存储数据库 `myjfs-oss.db` 中存储的信息必然也是有变化的。



\-



另外，关于 “OSS 中具体是怎样存储我们的数据的” 以及 “SQLite 数据库内是如何存储我们的元数据的” 这两个问题，本文是不关注的。

所以本文到这里就结束了 ~

