---
title: Linux 挂载磁盘的相关操作
date: 2022-09-08 01:09:00
updated: 2022-09-08 22:13:00
categories: [技术杂谈]
tags: [Linux]
toc: true
---

## 1. 常用操作简介

先给出最常用的挂载步骤和相关命令：

1. `fdisk -l` ：显示全部的磁盘分区信息。

2. `lsblk -f`：列出所有可用块设备的信息，包括文件系统类型，而且还能显示他们之间的依赖关系，但是不会列出 RAM 的信息。我比较喜欢和 `df -Th` 搭配用。后面我都会说。

3. `df -Th`：显示文件系统的类型、使用情况、挂载位置等信息（`T` 显示文件系统类型，`h` 表示把大小转成易读的单位）。

4. `mkfs`：格式化磁盘。

5. `mount`：挂载磁盘。

最后我们还应该进行配置，使得即便机器重启后可以自动挂载，这很有必要，否则可能会丢失数据！

然后根据我本机的情况，下面给出一个流程示例。

## 2. 一个示例

我这里的系统是 ubuntu 22.04，是阿里云的一个 ECS。

不过这些都不重要，各个系统的操作都是差不多的，下面开始。

### 2.1. 显示全部的磁盘分区信息

首先我们执行一下 `fdisk -l` 命令：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# fdisk -l
Disk /dev/loop0: 61.89 MiB, 64901120 bytes, 126760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 79.95 MiB, 83832832 bytes, 163736 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 44.68 MiB, 46845952 bytes, 91496 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop3: 46.96 MiB, 49242112 bytes, 96176 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop4: 61.96 MiB, 64966656 bytes, 126888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop5: 102.98 MiB, 107986944 bytes, 210912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 44CF1A76-5042-458F-92A0-2FB23895727C

Device      Start      End  Sectors  Size Type
/dev/vda1    2048     4095     2048    1M BIOS boot
/dev/vda2    4096   413695   409600  200M EFI System
/dev/vda3  413696 83886046 83472351 39.8G Linux filesystem


Disk /dev/nvme0n1: 447 GiB, 479962595328 bytes, 117178368 sectors
Disk model: ALIBABA_NVMe_Controller                 
Units: sectors of 1 * 4096 = 4096 bytes
Sector size (logical/physical): 4096 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

可以看到，我这里一共有 8 个磁盘，分别为 /dev/loop0 - /dev/loop5、/dev/vda 和 /dev/nvme0n1。注意这里 /dev/vda 下有三个部分，分别是 */dev/vda1 - /dev/vd3*，不过这不是本文重点。

我们暂且只要知道我们一共有 8 个磁盘就好了。我这里主要关注磁盘 **/dev/nvme0n1**，因为我预先知道了这就是我即将要挂载的磁盘（挂载磁盘的人肯定知道如何识别自己要挂载的磁盘，比如我在之前就知道这台 ECS 上有一个 447 GiB 的磁盘，这里就这一个，所以肯定就是这个，错不了）。

### 2.2. 显示磁盘的挂载状态

我比较喜欢 `lsblk -f` 命令：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# lsblk -f
NAME    FSTYPE   FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0   squashfs 4.0                                                    0   100% /snap/core20/1405
loop1   squashfs 4.0                                                    0   100% /snap/lxd/22923
loop2   squashfs 4.0                                                    0   100% /snap/snapd/15534
loop3   squashfs 4.0                                                    0   100% /snap/snapd/16292
loop4   squashfs 4.0                                                    0   100% /snap/core20/1611
loop5   squashfs 4.0                                                    0   100% /snap/lxd/23541
vda                                                                              
├─vda1                                                                           
├─vda2  vfat     FAT32       F3F3-8B04                             191.7M     3% /boot/efi
└─vda3  ext4     1.0         73186861-e43b-4de4-b562-8c33820998e3   34.4G     7% /
nvme0n1 
```

我们可以看到第一列 `NAME` 对应我们上面看到过的磁盘名，第二列 `FSTYPE` 是文件系统类型，最后一列 `MOUNTPOINTS` 表示挂载点，为空表示未挂载。同样的我们关注点在 **nvme0n1**，这行所有信息都是空的，是因为我们还没格式化，这个磁盘暂不可用。

我们再用 `df -Th` 命令看一下：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# df -Th
Filesystem     Type   Size  Used Avail Use% Mounted on
tmpfs          tmpfs  3.1G  1.1M  3.1G   1% /run
/dev/vda3      ext4    40G  2.9G   35G   8% /
tmpfs          tmpfs   16G     0   16G   0% /dev/shm
tmpfs          tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/vda2      vfat   197M  5.3M  192M   3% /boot/efi
tmpfs          tmpfs   50M     0   50M   0% /usr/local/aegis/cgroup
tmpfs          tmpfs  3.1G  4.0K  3.1G   1% /run/user/0
```

这个命令的显示了已挂载的磁盘的类型、使用信息、以及挂载的位置。我们同样可以看到，这里没有我们关注的 **nvme0n1**，说明这个磁盘还没有挂载。

### 2.3. 格式化磁盘

挂载前需要格式化磁盘，目前最常用的 Linux 文件系统格式是 ext4。

你也可以格式化为其他文件系统格式，这里不是本文重点，就不说了。

格式化命令如下：

```shell
mkfs [ -V ] [ -t fstype ] [ fs-options ] filesys [ blocks ]
```

* `-V`：表示详细模式。

* `-t`：指定要格式化的文件系统类型。

* `filesys`：要格式化的文件系统。

我们这里只用到这三个参数，更详细的 `mkfs` 命令可以[看这里](https://man7.org/linux/man-pages/man8/mkfs.8.html)，这里就不多说了。

我们现在格式化 **/dev/nvme0n1**：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# mkfs -V -t ext4 /dev/nvme0n1
mkfs from util-linux 2.37.2
mkfs.ext4 /dev/nvme0n1 
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done                            
Creating filesystem with 117178368 4k blocks and 29294592 inodes
Filesystem UUID: cfdedf14-a52b-40ba-8b1f-94a4f60fd766
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
    102400000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done     
```

现在就格式化完了，可以挂载了。

上面具体的输出内容看看就好，深究的话里面包含更多的 Linux 与操作系统知识，就超出本文范畴了。

我们再看一下 `lsblk -f` 命令：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# lsblk -f
NAME    FSTYPE   FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0   squashfs 4.0                                                    0   100% /snap/core20/1405
loop1   squashfs 4.0                                                    0   100% /snap/lxd/22923
loop2   squashfs 4.0                                                    0   100% /snap/snapd/15534
loop3   squashfs 4.0                                                    0   100% /snap/snapd/16292
loop4   squashfs 4.0                                                    0   100% /snap/core20/1611
loop5   squashfs 4.0                                                    0   100% /snap/lxd/23541
vda                                                                              
├─vda1                                                                           
├─vda2  vfat     FAT32       F3F3-8B04                             191.7M     3% /boot/efi
└─vda3  ext4     1.0         73186861-e43b-4de4-b562-8c33820998e3   34.4G     7% /
nvme0n1 ext4     1.0         cfdedf14-a52b-40ba-8b1f-94a4f60fd766
```

可以注意到，最后一行关于 **nvme0n1** 的信息被补全了一些，我们也可以看到其文件系统类型为 ext4，挂载点还没有被补全，因为我们还没挂载。而**在 2.2 中我们在格式化前执行 `lsblk -f` 时，这行的信息是空的！**

### 2.4. 挂载磁盘

挂载磁盘其实就像一个映射，你需要创建一个新的空文件夹，然后把你要挂载的磁盘“映射”到这个文件夹，这个文件夹就是该磁盘的挂载点。我们访问此文件夹，就是在访问挂载的磁盘。

我们这里先创建一个文件夹，位置任意，名字任意。我这里是 /mnt/nvme0n1。

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# mkdir /mnt/nvme0n1
```

然后使用 `mount` 命令将 **/dev/nvme0n1** 挂载到 /mnt/nvme0n1。

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# mount /dev/nvme0n1 /mnt/nvme0n1
```

如果成功了的话，这行命令是没有输出的。

现在我们再看看 `df -Th` 命令：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# df -Th
Filesystem     Type   Size  Used Avail Use% Mounted on
tmpfs          tmpfs  3.1G  1.1M  3.1G   1% /run
/dev/vda3      ext4    40G  2.9G   35G   8% /
tmpfs          tmpfs   16G     0   16G   0% /dev/shm
tmpfs          tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/vda2      vfat   197M  5.3M  192M   3% /boot/efi
tmpfs          tmpfs   50M     0   50M   0% /usr/local/aegis/cgroup
tmpfs          tmpfs  3.1G  4.0K  3.1G   1% /run/user/0
/dev/nvme0n1   ext4   439G   28K  417G   1% /mnt/nvme0n1
```

可以看到，我们关注的 **/dev/nvme0n1** 已经成功挂载了，挂载在目录 /mnt/nvme0n1 上。

再看看 `lsblk -f`，会发现最后一行已经完整了：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# lsblk -f
NAME    FSTYPE   FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0   squashfs 4.0                                                    0   100% /snap/core20/1405
loop1   squashfs 4.0                                                    0   100% /snap/lxd/22923
loop2   squashfs 4.0                                                    0   100% /snap/snapd/15534
loop3   squashfs 4.0                                                    0   100% /snap/snapd/16292
loop4   squashfs 4.0                                                    0   100% /snap/core20/1611
loop5   squashfs 4.0                                                    0   100% /snap/lxd/23541
vda                                                                              
├─vda1                                                                           
├─vda2  vfat     FAT32       F3F3-8B04                             191.7M     3% /boot/efi
└─vda3  ext4     1.0         73186861-e43b-4de4-b562-8c33820998e3   34.4G     7% /
nvme0n1 ext4     1.0         cfdedf14-a52b-40ba-8b1f-94a4f60fd766  416.5G     0% /mnt/nvme0n1
```

现在我们操作 /mnt/nvme0n1 目录就是在操作磁盘 **/dev/nvme0n1** 了，搞定。

### 2.5. 配置自动挂载

我们上面的挂载完成后，如果系统重启，挂载是不会保留的。

如果我们不配置自动挂载，那么数据是有可能丢失的！

所以下面我们配置一下自动挂载。

自动挂载的配置文件是 `/etc/fstab` ，我们打开。

```shell
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda3 during curtin installation
UUID=73186861-e43b-4de4-b562-8c33820998e3 / ext4 defaults 0 1
# /boot/efi was on /dev/vda2 during curtin installation
UUID=F3F3-8B04 /boot/efi vfat errors=remount-ro 0 1
```

添加一行挂载信息，挂载信息格式如下：

```
<file system> <mount point> <type> <options> <dump> <pass>
```

我这里最后结果是：

```shell
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda3 during curtin installation
UUID=73186861-e43b-4de4-b562-8c33820998e3 / ext4 defaults 0 1
# /boot/efi was on /dev/vda2 during curtin installation
UUID=F3F3-8B04 /boot/efi vfat errors=remount-ro 0 1

/dev/nvme0n1 /mnt/nvme0n1 ext4 default 0 0
```

现在当我们的系统启动时，会自动挂载 `/etc/fstab` 描述的内容了。

> 我们也可以执行 `mount -a` 立即应用 `/etc/fstab` 的内容，这适用于再次挂载曾经配置过的磁盘（比如磁盘已经格式化了，以前挂载过，这次可以不像之前那样一步一步来了，可以直接把挂载信息写在 `/etc/fstab`，然后 `mount -a` 应用）。我们这里就不需要了。

**我们现在要做的是，重启我们的操作系统，然后再检查一下，我们挂载的磁盘是否还在！**

我这里重启后，再次执行 `df -Th`：

```shell
root@iZ2zeg9tk9plor6dcquxpmZ:~# df -Th
Filesystem     Type   Size  Used Avail Use% Mounted on
tmpfs          tmpfs  3.1G  1.1M  3.1G   1% /run
/dev/vda3      ext4    40G  2.9G   35G   8% /
tmpfs          tmpfs   16G     0   16G   0% /dev/shm
tmpfs          tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/vda2      vfat   197M  5.3M  192M   3% /boot/efi
tmpfs          tmpfs   50M     0   50M   0% /usr/local/aegis/cgroup
tmpfs          tmpfs  3.1G  4.0K  3.1G   1% /run/user/0
/dev/nvme0n1   ext4   439G   28K  417G   1% /mnt/nvme0n1
```

关于 **/dev/nvme0n1** 的信息存在，说明自动挂载配置成功了。

### 2.6. 卸载磁盘

卸载命令很简单，`umount` 就是。

最长用的方法就是：

```shell
umount [Filesystem]
```

或

```shell
umount [MOUNTPOINTS]
```

即 `umount` 后面接磁盘名，或者挂载点，都可以卸载的。

以我们上面将磁盘 **/dev/nvme0n1** 挂载到 **/mnt/nvme0n1** 为例，下面两个命令都是可以正确卸载的：

```shell
umount /dev/nvme0n1
```

```shell
umount /mnt/nvme0n1
```



## 3. 相关命令扩展

我们上面用到的命令主要有 `fdisk`、`lsblk`、`df`、`mkfs` 和 `mount`。

但其实在我们上述场景中，这几个命令都只用到了一点功能，这些命令的全部功能要多得多，所以我在这里给出下这几个命令的参考文档，供查阅。

* `fdisk`：[fdisk(8) - Linux manual page](https://man7.org/linux/man-pages/man8/fdisk.8.html)

* `lsblk`：[lsblk(8) - Linux manual page](https://man7.org/linux/man-pages/man8/lsblk.8.html)

* `df`：[df(1) - Linux manual page](https://man7.org/linux/man-pages/man1/df.1.html)

* `mkfs`：[mkfs(8) - Linux manual page](https://man7.org/linux/man-pages/man8/mkfs.8.html)

* `mount`：[mount(8) - Linux manual page](https://man7.org/linux/man-pages/man8/mount.8.html)
