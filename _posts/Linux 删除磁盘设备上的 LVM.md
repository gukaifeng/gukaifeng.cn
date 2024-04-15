我这里以设备 `/dev/sdc` 为例：

```shell
# lsblk -o PATH,NAME,FSTYPE,FSSIZE,FSAVAIL,FSUSE%,MOUNTPOINT
PATH                                                                                                            NAME                          FSTYPE         FSSIZE FSAVAIL FSUSE% MOUNTPOINT
...
/dev/sdc                                                                                                        sdc                           LVM2_member                          
/dev/mapper/ceph--3ed272cd--3dd6--4e30--8b22--e8df598f454c-osd--block--4046edde--cdac--4ab9--8951--c4d25893e5ca └─ceph--3ed272cd--3dd6--4e30--8b22--e8df598f454c-osd--block--4046edde--cdac--4ab9--8951--c4d25893e5ca
                                                                                                                                              ceph_bluestore
...
```

使用 `pvs` 命令，先看下 LVM 的情况：

```shell
# pvs
  PV         VG                                        Fmt  Attr PSize  PFree
...
  /dev/sdc   ceph-3ed272cd-3dd6-4e30-8b22-e8df598f454c lvm2 a--  <3.64t    0 
...
```

这里会有个 VG 名字，我们使用命令 `vgremove` 从 LVM 系统中移除这个 VG：

```shell
# vgremove ceph-3ed272cd-3dd6-4e30-8b22-e8df598f454c
Do you really want to remove volume group "ceph-3ed272cd-3dd6-4e30-8b22-e8df598f454c" containing 1 logical volumes? [y/n]: y
Do you really want to remove and DISCARD active logical volume ceph-3ed272cd-3dd6-4e30-8b22-e8df598f454c/osd-block-4046edde-cdac-4ab9-8951-c4d25893e5ca? [y/n]: y
```

使用命令 `pvremove` 从 PV 中移除 LVM：

```shell
# pvremove /dev/sdc
  Labels on physical volume "/dev/sdc" successfully wiped.
```