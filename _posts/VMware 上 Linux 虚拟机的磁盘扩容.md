---
title: "VMware 上 Linux 虚拟机的磁盘扩容"
date: 2023-04-12 00:34:00
updated: 2023-04-12 00:34:00
categories: [技术杂谈]
tags: [Linux,虚拟机]
---







本文适用 VMware 虚拟机，系统为 CentOS/RHEL 系或与之兼容的发行版，我的操作系统为 AlmaLinux 8.7。

下面各项输出中所有包含 "almalinux" 字样的内容，均可替换为你的系统名字。比如你的系统是 CentOS，那么把我这里的输出信息中的 "almalinux" 都替换成 "centos" 就是你应该有的输出。

## 1. 扩容过程大纲



为了说明的清晰一些，我这里将 VMware 内 Linux 虚拟机的扩容过程分为了以下几步：



先是在 VMware 上的操作：

1. 关闭并**备份**虚拟机。
2. 删除虚拟机的全部快照。
3. 扩容虚拟机。
4. 启动虚拟机。





然后是在 Linux 虚拟机内的操作：

1. 查看当前的磁盘分区信息与挂载信息。
1. 给磁盘 `/dev/sda` 增加一个分区。
1. 设置新分区格式。
1. 写入分区表。
1. 格式化新分区。
1. 进入 `lvm` 合并磁盘。
1. 增加 `xfs` 磁盘空间。





## 2. VMware 上的操作



VMware 上的操作很简单，我们按照前文说的步骤进行就可以：



1. 关闭并备份虚拟机。备份主要是避免把虚拟机搞坏了，毕竟磁盘操作挺危险的，就把虚拟机文件夹拷贝一份就行。

3. 删除虚拟机的全部快照。这是 VMware 的要求，没有快照的虚拟机才可以扩容。

3. 扩容虚拟机：

   首先打开虚拟机设置，选择 磁盘 -> 扩展：

   ![](https://gukaifeng.cn/posts/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong_1.png)

   指定新的磁盘大小，我这里是从 20 GB 扩容到 60 GB，然后点击“扩展”：

   ![](https://gukaifeng.cn/posts/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong_2.png)

   这里会有个提示，告诉我们在虚拟机内需要对磁盘重新分区和扩展文件系统（这就是我们下一节要做的事）：

   ![](https://gukaifeng.cn/posts/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong/vmware-shang-linux-xu-ni-ji-de-ci-pan-kuo-rong_3.png)

   

5. 启动虚拟机。





启动虚拟机以后，在虚拟机内进行下一步的操作。



## 3. Linux 虚拟机内的操作

Linux 虚拟机内的过程是比较复杂的，但是不用担心，按下面的步骤慢慢来就好了。

### 3.1. 查看当前的磁盘分区信息与挂载信息

这一步并不是我们扩容操作的一部分，但是要先看一下我们扩容之前的磁盘情况，有一些信息是有必要知道的。







首先使用 `fdisk -l` 命令查看磁盘分区情况：

```
$ sudo fdisk -l
[sudo] password for gukaifeng: 
Disk /dev/sda: 60 GiB, 64424509440 bytes, 125829120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ff31de0

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 41943039 39843840  19G 8e Linux LVM


Disk /dev/mapper/almalinux-root: 17 GiB, 18249416704 bytes, 35643392 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/almalinux-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

注意看上面我们需要的一些信息：

* 我们有一个磁盘 `/dev/sda`，大小为 60 GiB，这是我们扩容后的磁盘。
* 磁盘 `/dev/sda` 有两个分区，编号为 1 和 2，分别为 `/dev/sda1` 和 `/dev/sda2`。
* `/dev/mapper/almalinux-root` 是我们主要的存储磁盘，当前大小为 17 GiB，这个对应的是 `/dev/sda2`。
* `/dev/sda2` 的十六进制 ID 是 `8e`，代表类型 Linux LVM。我们后面需要以这个类型创建新分区。



然后查看扩容前的磁盘容量：

```
$ df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs                      tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs                      tmpfs     3.8G  9.0M  3.8G   1% /run
tmpfs                      tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/almalinux-root xfs        17G  2.1G   15G  13% /
/dev/sda1                  xfs      1014M  168M  847M  17% /boot
tmpfs                      tmpfs     774M     0  774M   0% /run/user/1000
```

我们需要的信息如下：

* `/dev/mapper/almalinux-root`，这对应上面我们 `fdisk -l` 命令中查到的 `/dev/sda2`。
* `/dev/mapper/almalinux-root` 磁盘格式为 `xfs`。
* `/dev/mapper/almalinux-root` 的挂载点为根目录 `/`，也就是我们 Linux 的存储根目录。



从这些信息我们也可以看得出来，虽然我们在 VMware 上给虚拟机扩容了，但对里面的 Linux 虚拟机而言，磁盘只是扩大了，但可用大小并没有变化，原因就是新增加的空间没有分区和格式化。



我们记得这些信息以后，就可以进行下一步了。



### 3.2. 给磁盘 `/dev/sda` 增加一个分区。

我们使用 `fdisk` 命令编辑我们的磁盘 `/dev/sda`：

```shell
fdisk /dev/sda
```

然后会进入一个交互式页面，输入 `m` 可以查看帮助信息：

```shell
$ sudo fdisk /dev/sda
[sudo] password for gukaifeng: 

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

这里就不解释上面的帮助了，只说我们用得到的。

输入 `n` 添加一个分区，这个就是把我们给磁盘扩容的那些空间分到这个将要添加的新分区里：

```shell
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (3,4, default 3): 
First sector (41943040-125829119, default 41943040): 
Last sector, +sectors or +size{K,M,G,T,P} (41943040-125829119, default 125829119): 

Created a new partition 3 of type 'Linux' and of size 40 GiB.
```

我这里先是输入 `n`，然后后面一直回车选择默认值。具体含义分别是：

* `Partition type`：分区类型。选择默认的 `p`，即创建主分区。
* `Partition number`：分区号。选择默认的 3，因为我们之前的是到 2（`/dev/sda1` 和 `/dev/sda2`），连续比较顺眼。
* `First sector`：起始扇区。这里给了一个范围 41943040-125829119，这就是我们磁盘扩容部分的起始扇区和结束扇区，我们直接默认值就好，从第一个扇区开始。
* `Last sector`：结束扇区。同上，直接默认，选择最后一个扇区，这样不浪费任何空间。

然后可以看到提示，我们创建了一个新的分区 3，大小是 40 GiB，这就是我们之前在 VMware 上扩容的 40 GiB。



### 3.3. 设置新分区格式



接上面的交互式页面。



输入 `t` 为我们的新分区 3 设置格式：

```shell
Command (m for help): t
Partition number (1-3, default 3): 
Hex code (type L to list all codes): 8e

Changed type of partition 'Linux' to 'Linux LVM'.
```

先选择一下分区，选择 3（默认值）。

然后设置下分区类型，想一下我们在 3.1 小节查看的信息，`/dev/sda2` 的十六进制 ID 是 `8e`，代表类型 Linux LVM。所以这里我们也输入 `8e`，为新分区设置类型为 Linux LVM。



### 3.4. 写入分区表

接上面的交互式页面。

输入 `w`，将分区信息写入分区表，并退出。

```shell
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

我们用 `fdisk -l` 命令查看一下现在磁盘信息：

```shell
$ sudo fdisk -l
[sudo] password for gukaifeng: 
Disk /dev/sda: 60 GiB, 64424509440 bytes, 125829120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ff31de0

Device     Boot    Start       End  Sectors Size Id Type
/dev/sda1  *        2048   2099199  2097152   1G 83 Linux
/dev/sda2        2099200  41943039 39843840  19G 8e Linux LVM
/dev/sda3       41943040 125829119 83886080  40G 8e Linux LVM


Disk /dev/mapper/almalinux-root: 17 GiB, 18249416704 bytes, 35643392 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/almalinux-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到已经有我们刚刚创建的 `/dev/sda3` 了。

但是 `df -Th` 命令还查不到，因为我们这个新分区还没有挂载的，是不可用的。这里就不放输出了，可以自己敲一下。



### 3.5. 格式化新分区

我们在 3.1 知道了我这里 `/dev/sda2` 的磁盘格式为 `xfs`（读者的可能和我的是不同的），所以我需要把 `/dev/sda3` 也格式化为 `xfs`。



首先输入命令重读一下分区表，让内核知道我们刚刚的改动：

```shell
partprobe
```

然后格式化 `/dev/sda3` 为 `xfs`：

```shell
mkfs.xfs /dev/sda3
```



### 3.6. 进入 `lvm` 合并磁盘



输入命令 `lvm` 进入交互式页面：

```shell
$ sudo lvm
lvm> 
```

输入命令 `pvcreate` 初始化 `/dev/sda3`：

```shell
 pvcreate /dev/sda3
```

```shell
lvm> pvcreate /dev/sda3
WARNING: xfs signature detected on /dev/sda3 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sda3.
  Physical volume "/dev/sda3" successfully created.
```



这里我们需要使用命令 `vgdisplay -v` 先查看一下当前卷的详情（节选）：

```shell
lvm> vgdisplay -v
  --- Volume group ---
  VG Name               almalinux
  System ID             
...
...
```

这里面我们主要看 `Volume group` 的名字 `VG Name`，我这里是 "almalinux"，你要是CentOS 系统就是 "centos"。



我们要把新的分区加入到这个 `Volume group` 中，使用命令 `vgextend `：

```shell
vgextend almalinux /dev/sda3
```

```shell
lvm> vgextend almalinux /dev/sda3
  Volume group "almalinux" successfully extended
```



然后再次查看卷详情：



```shell
lvm> vgdisplay -v
  --- Volume group ---
  VG Name               almalinux
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               58.99 GiB
  PE Size               4.00 MiB
  Total PE              15102
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       10239 / <40.00 GiB
  VG UUID               mPM6Oh-tKMA-BLkG-HQdw-dymm-852d-6usdbi
   
  --- Logical volume ---
  LV Path                /dev/almalinux/swap
  LV Name                swap
  VG Name                almalinux
  LV UUID                sTmb3V-GlEE-bGEe-gjBY-aPjU-GtYe-iY6NJA
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2023-04-08 05:59:54 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/almalinux/root
  LV Name                root
  VG Name                almalinux
  LV UUID                yCSqBp-P7Hl-jGJT-FelI-WF0e-okzg-Y7nlqU
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2023-04-08 05:59:55 +0800
  LV Status              available
  # open                 1
  LV Size                <17.00 GiB
  Current LE             4351
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
   
  --- Physical volumes ---
  PV Name               /dev/sda2     
  PV UUID               OHdlZx-0rLN-03kY-9Yna-WPbS-HuUQ-5QqcgO
  PV Status             allocatable
  Total PE / Free PE    4863 / 0
   
  PV Name               /dev/sda3     
  PV UUID               9lTdhS-PyBq-Xebh-JiQd-GLKf-2Wqk-gJ66Em
  PV Status             allocatable
  Total PE / Free PE    10239 / 10239
```

注意最后的 `Physical volumes` 的最后，有一个 `/dev/sda3`，这是我们刚刚创建的分区。最后有个 `Total PE`，我这里为 10239，记住这个数字。





最后将系统盘 `/dev/mapper/almalinux-root` 与 `/dev/sda3` 的 10239 空余容量合并，输入如下命令：

```shell
lvextend -l +10239 /dev/mapper/almalinux-root
```

```shell
lvm> lvextend -l +10239 /dev/mapper/almalinux-root
  Size of logical volume almalinux/root changed from <17.00 GiB (4351 extents) to 56.99 GiB (14590 extents).
  Logical volume almalinux/root successfully resized.
```



（3.1 说过 `/dev/mapper/almalinux-root` 就是挂载到 `/` 的，是我们 Linux 系统的根目录）



然后输入 `quit` 退出 `lvm`：

```shell
lvm> quit
  Exiting.
```

### 3.7. 增加 `xfs` 磁盘空间



输入命令：

```shell
xfs_growfs /dev/mapper/almalinux-root
```

这个命令就是将 `/dev/mapper/almalinux-root` 的空间真正增加了，将刚刚合并进来的 `/dev/sda3` 利用起来。



```shell
$ sudo xfs_growfs /dev/mapper/almalinux-root
meta-data=/dev/mapper/almalinux-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 14940160
```



## 4. 验证

输入命令 `df -Th`：

```shell
$ df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs                      tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs                      tmpfs     3.8G  9.0M  3.8G   1% /run
tmpfs                      tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/almalinux-root xfs        57G  2.4G   55G   5% /
/dev/sda1                  xfs      1014M  168M  847M  17% /boot
tmpfs                      tmpfs     774M     0  774M   0% /run/user/1000
```

可以看到我们的 `/dev/mapper/almalinux-root` 扩容已经成功了，`Size` 是预期的大小。



也可以顺便再用 `fdisk -l` 看一下磁盘分区情况，我们的 `/dev/sda3` 安然在列：

```shell
$ sudo fdisk -l
Disk /dev/sda: 60 GiB, 64424509440 bytes, 125829120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4ff31de0

Device     Boot    Start       End  Sectors Size Id Type
/dev/sda1  *        2048   2099199  2097152   1G 83 Linux
/dev/sda2        2099200  41943039 39843840  19G 8e Linux LVM
/dev/sda3       41943040 125829119 83886080  40G 8e Linux LVM


Disk /dev/mapper/almalinux-root: 57 GiB, 61194895360 bytes, 119521280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/almalinux-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

到这里我们的扩容操作就全部完成了 ~
