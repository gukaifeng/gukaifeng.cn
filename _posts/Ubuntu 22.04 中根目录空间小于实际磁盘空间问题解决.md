## 1. 问题场景

今天创建了一个 Ubuntu 22.04 的虚拟机，分配了磁盘为 100GB，但在系统里看却只有 48 GiB。





## 2. 问题分析



我们查看一下文件系统的挂载情况：

```shell
$ df -Th
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  791M  1.7M  789M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    48G   12G   35G  25% /
tmpfs                             tmpfs  3.9G     0  3.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  129M  1.7G   8% /boot
tmpfs                             tmpfs  791M  4.0K  791M   1% /run/user/1000
```



可以看到，`/` 挂载的设备是 `/dev/mapper/ubuntu--vg-ubuntu--lv`，容量只有 48 GiB，这显然不符合预期。

于是我查看了一下磁盘的信息：

```shell
$ sudo fdisk -l
Disk /dev/loop0: 63.45 MiB, 66531328 bytes, 129944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 111.95 MiB, 117387264 bytes, 229272 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 53.26 MiB, 55844864 bytes, 109072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 64F73FB2-B4E5-4294-8D6D-09BD8509C8DD

Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   4198399   4194304   2G Linux filesystem
/dev/sda3  4198400 209713151 205514752  98G Linux filesystem


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 49 GiB, 52609155072 bytes, 102752256 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到，`/dev/mapper/ubuntu--vg-ubuntu--lv` 确实只有 49 GiB，符合我们预期的应该是 `/dev/sda3` 有 98 GiB。

这其实是因为 Ubuntu 22.04 使用了 LVM（Logical Volume Manager，逻辑卷管理器）来管理磁盘空间。

在这种情况下，`/dev/sda3` 是一个物理卷 (Physical Volume)，它被添加到一个卷组 (Volume Group) 中，然后从卷组中创建了一个逻辑卷 (Logical Volume)，即 `/dev/mapper/ubuntu--vg-ubuntu--lv`。逻辑卷是用来创建文件系统并挂载的。



我们看下卷组信息：

```shell
$ sudo vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <98.00 GiB
  PE Size               4.00 MiB
  Total PE              25087
  Alloc PE / Size       12543 / <49.00 GiB
  Free  PE / Size       12544 / 49.00 GiB
  VG UUID               yXGbkZ-Df4M-MHt2-rEPb-qLZB-4CKW-EGA7ic
```

再看下逻辑卷信息：

```shell
$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                R9z0zw-JgyS-c38B-xcvu-YSEA-T8PR-fVjjuL
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2024-03-25 05:25:36 +0000
  LV Status              available
  # open                 1
  LV Size                <49.00 GiB
  Current LE             12543
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```



可以看到，卷组 ubuntu-vg 分配了 <49.00 GiB (Alloc PE / Size) 的空间给逻辑卷 `ubuntu-lv`。卷组 ubuntu-vg 还有 49 GiB (Free  PE / Size) 是空闲的，这俩加起来的 98 GiB 就是我们预期的。





## 3. 解决方案



把另外 49 GiB 的空闲空间，扩容到这个卷组 ubuntu-vg 就好了。

运行命令：

```shell
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

注意，这里的 `/dev/ubuntu-vg/ubuntu-lv` 是我们上面命令 `lvdisplay` 输出结果中的 `LV Path` 值。

输出是这样子的：

```shell
$ sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <49.00 GiB (12543 extents) to <98.00 GiB (25087 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```

最后执行：

```shell
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

输出：

```shell
$ sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 13
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 25689088 (4k) blocks long.
```

到这里就完成了，可以再看下 `/` 的大小：

```shell
$ df -Th
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  791M  1.7M  789M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    97G   12G   81G  13% /
tmpfs                             tmpfs  3.9G     0  3.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  129M  1.7G   8% /boot
tmpfs                             tmpfs  791M  4.0K  791M   1% /run/user/1000
```

符合预期 ~