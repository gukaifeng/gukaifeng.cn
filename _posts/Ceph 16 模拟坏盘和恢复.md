## 1. 准备工作



为了方便模拟坏盘和恢复，我们需要先记下一些信息。

可以先看下 OSD 和设备的对应关系：

```shell
ceph device ls
```

或：

```shell
ceph device ls-by-host <hostname>
```



首先我们要决定模拟坏盘的 OSD 和对应的设备，我这里选择 `osd.1`，对应的设备为 `/dev/sdc`。

然后，我们需要看下选择设备的位置，命令如下：

```shell
ll /sys/block/sdX
```

记下这个软连接中的 host 编号，方便后面恢复坏盘，我这里是 `host0`：

```shell
ll /sys/block/sdc
lrwxrwxrwx 1 root root 0 Apr 17 16:08 /sys/block/sdc -> ../devices/pci0000:17/0000:17:02.0/0000:18:00.0/host0/target0:2:3/0:2:3:0/block/sdc/
```





## 2. 模拟坏盘



从系统中删除设备，模拟坏盘。命令：

```shell
echo 1 > /sys/block/sdX/device/delete
```

我这里是：

```shell
echo 1 > /sys/block/sdc/device/delete
```



如果集群有写入，对应的 OSD 就很快 `down` 掉了：

```shell
# ceph osd tree | grep osd.1
1    hdd    3.63820          osd.1                            down   1.00000  1.00000
```



再过一会儿不恢复，就会自动 `out` 并回填数据：

```shell
# ceph osd dump | grep osd.1
osd.1 down out weight 0 ...
```

```shell
# ceph -s
  cluster:
    id:     c5e3da58-ee3f-11ee-a744-e3cf29ca2a71
    health: HEALTH_WARN
            Failed to apply 1 service(s): osd.all-available-devices
            1 failed cephadm daemon(s)
            Degraded data redundancy: 3934275/110518560 objects degraded (3.560%), 4 pgs degraded, 4 pgs undersized
 
  services:
    mon: 3 daemons, quorum tj5-s1-v6-tj5-128473-2yqgrsr3.kscn,tj5-s1-v6-tj5-128473-6eognfvg,tj5-s1-v6-tj5-128473-zbwf3fhp (age 2d)
    mgr: tj5-s1-v6-tj5-128473-6eognfvg.hdmmes(active, since 21h), standbys: tj5-s1-v6-tj5-128473-2yqgrsr3.kscn.nopizg
    osd: 36 osds: 35 up (since 47m), 35 in (since 37m); 4 remapped pgs
 
  data:
    pools:   2 pools, 33 pgs
    objects: 36.84M objects, 8.8 TiB
    usage:   26 TiB used, 101 TiB / 127 TiB avail
    pgs:     3934275/110518560 objects degraded (3.560%)
             28 active+clean
             4  active+undersized+degraded+remapped+backfilling
             1  active+clean+scrubbing+deep
 
  io:
    recovery: 7.5 MiB/s, 29 objects/s
```



>如果我们想保留 OSD 的编号，比如我这里复用 id 1，那么需要先 destory 这个 OSD，然后其编号就会被复用。
>
> ```shell
> ceph osd destroy <osd.id> [--force]
> ```
>
>因为已经坏盘了，所以也没必要去判断 `ceph osd safe-to-destroy <osd.id>` 了。 



## 3. 恢复坏盘



首先将磁盘恢复。命令：

```shell
echo '- - -' > /sys/class/scsi_host/hostX/scan
```

注意这里的 host 编号，换成我们一开始查的那个，我这里是：

```shell
echo '- - -' > /sys/class/scsi_host/host0/scan
```



然后看一下磁盘是否正确加载了：

```shell
# lsblk
NAME                                                                                                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
...
sdb                                                                                                     8:16   0   3.7T  0 disk 
└─ceph--b7039b7d--da7b--450b--9a0c--3b27770afecb-osd--block--1c953852--a165--432c--bd7f--258659bd32ca 253:0    0   3.7T  0 lvm  
sdd                                                                                                     8:48   0   3.7T  0 disk 
└─ceph--0d54dcbe--1b21--4c4d--87cf--2132c65674ea-osd--block--ddf587c3--0bf3--45e5--b72c--5d97d5539e16 253:2    0   3.7T  0 lvm  
...
sdn                                                                                                     8:208  0   3.7T  0 disk 
```

可以看到我这里没有 `sdc`，但是多了个 `sdn`，这是因为磁盘目录没有复用。所以我们接下来操作 `sdn`。

先让 Ceph 重新扫描这台机器上的设备：

```shell
ceph orch host rescan <hostname>
```

然后看下我们的新设备是否被正确识别了：

```shell
# ceph orch device ls
HOST                                PATH      TYPE  DEVICE ID                                         SIZE  AVAILABLE  REFRESHED  REJECT REASONS                                                                 
...
tj5-s1-v6-tj5-128473-2yqgrsr3.kscn  /dev/sdn  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812411cad387a  3725G             44s ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
...
```



可以看到是正确识别到了的，但是因为我们这个盘上面其实是有数据的，所以这个设备暂时不可用。

我们还要抹掉数据，模拟新的空磁盘加入。主要就是把 LVM 干掉，参考 [Linux 删除磁盘设备上的 LVM](https://gukaifeng.cn/posts/linux-shan-chu-ci-pan-she-bei-shang-de-lvm/)。



> 如果是真实场景的加入磁盘，也需要操作使得新磁盘满足以下条件：
>
> 1. 该设备必须没有分区。
> 2. 设备不得具有任何 LVM 状态。
> 3. 该设备没有被挂载。
> 4. 该设备不得包含文件系统。
> 5. 设备不得包含 Ceph BlueStore OSD。
> 6. 该设备必须大于 5 GB。



操作完成后，稍等片刻，就可以看到设备已经被标记为 Yes 了，这意味着可以给 OSD 使用了。

```shell
# ceph orch device ls
HOST                                PATH      TYPE  DEVICE ID                                         SIZE  AVAILABLE  REFRESHED  REJECT REASONS                                                                 
...
tj5-s1-v6-tj5-128473-2yqgrsr3.kscn  /dev/sdn  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812411cad387a  3725G  Yes        10s ago                                                                                   
...
```



再稍等一会儿，Ceph 会自动在这个设备上创建新的 OSD。

例如我这里创建了 OSD 36（原来有 36 个 OSD，编号 0 ~ 35，这是第 37 个 OSD）：

```shell
# ceph osd tree
ID  CLASS  WEIGHT     TYPE NAME                               STATUS  REWEIGHT  PRI-AFF
-1         134.61336  root default                                                     
-3          47.29659      host tj5-s1-v6-tj5-128473-2yqgrsr3                           
...
 1    hdd    3.63820          osd.1                             down         0  1.00000
...
36    hdd    3.63820          osd.36                              up   1.00000  1.00000
...
```

到这里坏盘的替换就完成了。

没有复用 osd id 的情况下，可能还需要清一下失效了的 osd daemon，可以先命令看一下：

```
# ceph orch ps
NAME                                           HOST                                PORTS        STATUS          REFRESHED   AGE  MEM USE  MEM LIM  VERSION    IMAGE ID      CONTAINER ID  
...
osd.1                                          tj5-s1-v6-tj5-128473-2yqgrsr3.kscn               error              6m ago    2w        -    5751M  <unknown>  <unknown>     <unknown>     
...
```

删除这个 daemon 就好：

```shell
ceph orch daemon rm osd.<id> [--force]
```



> 如果前面 destroy 了 osd.1，那么这里新 osd 的 id 会复用 1。
>
> 但是复用了 id 以后，由于 osd daemon 还是以前那个，所以跑起来一样会 error。
>
> 需要重新部署：
>
> ```shell
> ceph orch daemon redeploy osd.<id>
> ```

