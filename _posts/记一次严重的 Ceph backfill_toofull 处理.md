

先看一下我这个集群的状态：

```shell
# ceph -s
  cluster:
    id:     1e2c2112-037f-11ef-b466-45c2ff826e1a
    health: HEALTH_ERR
            1 backfillfull osd(s)
            1 full osd(s)
            Low space hindering backfill (add storage if this doesn't resolve itself): 15 pgs backfill_toofull
            3 pool(s) full
 
  services:
    mon: 5 daemons, quorum c3-hadoop-ceph-st38.bj,c3-hadoop-ceph-st40,c3-hadoop-ceph-st43,c3-hadoop-ceph-st42,c3-hadoop-ceph-st41 (age 2d)
    mgr: c3-hadoop-ceph-st38.bj.urnckz(active, since 3d), standbys: c3-hadoop-ceph-st39.rgvvtd
    osd: 48 osds: 48 up (since 4m), 48 in (since 2d); 15 remapped pgs
 
  data:
    pools:   3 pools, 193 pgs
    objects: 65.91M objects, 13 TiB
    usage:   38 TiB used, 48 TiB / 86 TiB avail
    pgs:     57588000/274612068 objects misplaced (20.971%)
             178 active+clean
             15  active+remapped+backfill_toofull
 
  progress:
    Global Recovery Event (20h)
      [=========================...] (remaining: 105m)
```

可以看到这句话：

```shell
Low space hindering backfill (add storage if this doesn't resolve itself): 15 pgs backfill_toofull
```

有 15 个 pg 是 backfill_toofull 状态，**我这里是无法自行恢复的，已经持续这个状态非常久非常久了。**按照提示的做法，应该加新的存储节点，但我这个集群的问题是，当前严重不均衡，还有大量空闲节点，所以我要想办法把这些 backfill_toofull 解决，而不是加存储节点。

本来一开始就是只有 backfill_full 的，按照 [IBM 的一个博客](https://www.ibm.com/docs/en/storage-ceph/7?topic=errors-backfillfull-osds)方法，逐渐调高阈值，期望可以解除 backfill_toofull 状态，使 backfill 继续进行：

```shell
ceph osd set-backfillfull-ratio <value>
```

结果调到最后也没有满足 backfill 所需的空间，反而导致 osd 满了，触发 OSD full，直接导致集群不可用，爆炸。

我这里主要是集群严重不均衡（Reweight 有调节但还没完成就炸了）：

```shell
# ceph osd df
ID  CLASS  WEIGHT   REWEIGHT  SIZE     RAW USE   DATA      OMAP     META     AVAIL    %USE   VAR   PGS  STATUS
 2    ssd  1.81940   1.00000  1.8 TiB   305 MiB    15 MiB      0 B  290 MiB  1.8 TiB   0.02     0    0      up
11    ssd  1.81940   1.00000  1.8 TiB   616 GiB   614 GiB  504 KiB  2.5 GiB  1.2 TiB  33.08  0.75   11      up
17    ssd  1.81940   1.00000  1.8 TiB   1.5 TiB   1.5 TiB      0 B  5.6 GiB  321 GiB  82.79  1.87   25      up
23    ssd  1.81940   0.58890  1.8 TiB   1.0 TiB   1.0 TiB      0 B  4.5 GiB  822 GiB  55.89  1.26   15      up
29    ssd  1.81940   1.00000  1.8 TiB   617 GiB   615 GiB      0 B  2.4 GiB  1.2 TiB  33.12  0.75   10      up
35    ssd  1.81940   0.67078  1.8 TiB  1023 GiB  1004 GiB      0 B   18 GiB  840 GiB  54.89  1.24   20      up
41    ssd  1.81940   1.00000  1.8 TiB   617 GiB   615 GiB      0 B  2.7 GiB  1.2 TiB  33.14  0.75   10      up
47    ssd  1.81940   1.00000  1.8 TiB   1.4 TiB   1.4 TiB      0 B   14 GiB  437 GiB  76.53  1.73   32      up
 5    ssd  1.81940   0.76396  1.8 TiB   1.7 TiB   1.7 TiB      0 B   15 GiB  130 GiB  93.04  2.10   37      up
10    ssd  1.81940   1.00000  1.8 TiB   616 GiB   614 GiB      0 B  2.0 GiB  1.2 TiB  33.08  0.75   10      up
15    ssd  1.81940   0.58902  1.8 TiB   819 GiB   815 GiB      0 B  4.1 GiB  1.0 TiB  43.98  0.99   10      up
20    ssd  1.81940   0.78557  1.8 TiB   1.3 TiB   1.3 TiB  755 KiB  5.3 GiB  515 GiB  72.34  1.63   21      up
26    ssd  1.81940   1.00000  1.8 TiB   305 MiB    15 MiB      0 B  290 MiB  1.8 TiB   0.02     0    0      up
32    ssd  1.81940   0.52232  1.8 TiB   1.1 TiB   1.0 TiB      0 B   19 GiB  782 GiB  58.05  1.31   20      up
38    ssd  1.81940   1.00000  1.8 TiB   309 GiB   307 GiB      0 B  1.4 GiB  1.5 TiB  16.57  0.37    5      up
44    ssd  1.81940   0.58987  1.8 TiB   683 GiB   678 GiB      0 B  5.4 GiB  1.2 TiB  36.68  0.83    5      up
 1    ssd  1.81940   0.58955  1.8 TiB   1.5 TiB   1.5 TiB      0 B  5.2 GiB  321 GiB  82.75  1.87   25      up
 8    ssd  1.81940   0.78575  1.8 TiB   531 GiB   527 GiB      0 B  3.8 GiB  1.3 TiB  28.49  0.64    5      up
14    ssd  1.81940   1.00000  1.8 TiB   305 MiB    15 MiB      0 B  290 MiB  1.8 TiB   0.02     0    0      up
21    ssd  1.81940   1.00000  1.8 TiB   1.4 TiB   1.4 TiB      0 B   14 GiB  439 GiB  76.46  1.73   32      up
28    ssd  1.81940   0.20337  1.8 TiB   1.0 TiB  1024 GiB      0 B   21 GiB  818 GiB  56.07  1.27   20      up
34    ssd  1.81940   1.00000  1.8 TiB   617 GiB   614 GiB      0 B  3.0 GiB  1.2 TiB  33.13  0.75   10      up
40    ssd  1.81940   1.00000  1.8 TiB   925 GiB   921 GiB      0 B  3.5 GiB  938 GiB  49.64  1.12   15      up
46    ssd  1.81940   0.78619  1.8 TiB   1.2 TiB   1.2 TiB      0 B  4.2 GiB  630 GiB  66.18  1.49   20      up
 4    ssd  1.81940   0.58958  1.8 TiB   388 GiB   383 GiB      0 B  4.8 GiB  1.4 TiB  20.83  0.47    0      up
 9    ssd  1.81940   0.57719  1.8 TiB   1.1 TiB   1.1 TiB      0 B   14 GiB  758 GiB  59.31  1.34   22      up
16    ssd  1.81940   1.00000  1.8 TiB   310 GiB   308 GiB      0 B  1.9 GiB  1.5 TiB  16.63  0.38    5      up
22    ssd  1.81940   1.00000  1.8 TiB   1.2 TiB   1.2 TiB      0 B  4.3 GiB  629 GiB  66.22  1.50   20      up
27    ssd  1.81940   1.00000  1.8 TiB   616 GiB   614 GiB      0 B  2.3 GiB  1.2 TiB  33.08  0.75   10      up
33    ssd  1.81940   1.00000  1.8 TiB   617 GiB   615 GiB      0 B  2.4 GiB  1.2 TiB  33.13  0.75   10      up
39    ssd  1.81940   1.00000  1.8 TiB   926 GiB   923 GiB      0 B  3.8 GiB  937 GiB  49.72  1.12   15      up
45    ssd  1.81940   1.00000  1.8 TiB   617 GiB   615 GiB      0 B  2.5 GiB  1.2 TiB  33.12  0.75   10      up
 3    ssd  1.74660   0.56560  1.7 TiB  1009 GiB  1005 GiB  504 KiB  4.1 GiB  779 GiB  56.44  1.27   16      up
 6    ssd  1.74660   0.64401  1.7 TiB   1.3 TiB   1.3 TiB      0 B   20 GiB  458 GiB  74.40  1.68   25      up
12    ssd  1.74660   1.00000  1.7 TiB   617 GiB   614 GiB      0 B  2.7 GiB  1.1 TiB  34.51  0.78   10      up
18    ssd  1.74660   0.56595  1.7 TiB   989 GiB   985 GiB      0 B  4.2 GiB  799 GiB  55.31  1.25   15      up
24    ssd  1.74660   1.00000  1.7 TiB   1.1 TiB   1.1 TiB      0 B   14 GiB  671 GiB  62.46  1.41   27      up
30    ssd  1.74660   1.00000  1.7 TiB   305 MiB    15 MiB      0 B  290 MiB  1.7 TiB   0.02     0    0      up
36    ssd  1.74660   1.00000  1.7 TiB   309 GiB   307 GiB      0 B  1.4 GiB  1.4 TiB  17.27  0.39    5      up
43    ssd  1.74660   0.75381  1.7 TiB   925 GiB   922 GiB      0 B  3.3 GiB  863 GiB  51.75  1.17   15      up
 0    ssd  1.74660   0.75343  1.7 TiB   926 GiB   922 GiB      0 B  3.9 GiB  863 GiB  51.77  1.17   15      up
 7    ssd  1.74660   0.64394  1.7 TiB   838 GiB   820 GiB      0 B   18 GiB  950 GiB  46.88  1.06   15      up
13    ssd  1.74660   0.55382  1.7 TiB   1.7 TiB   1.7 TiB      0 B   14 GiB   54 GiB  96.99  2.19   37      up
19    ssd  1.74660   1.00000  1.7 TiB   309 GiB   307 GiB      0 B  1.5 GiB  1.4 TiB  17.25  0.39    5      up
25    ssd  1.74660   1.00000  1.7 TiB   309 GiB   307 GiB      0 B  1.9 GiB  1.4 TiB  17.27  0.39    5      up
31    ssd  1.74660   1.00000  1.7 TiB   618 GiB   615 GiB      0 B  2.7 GiB  1.1 TiB  34.55  0.78   10      up
37    ssd  1.74660   0.19479  1.7 TiB   825 GiB   805 GiB      0 B   20 GiB  963 GiB  46.14  1.04   15      up
42    ssd  1.74660   1.00000  1.7 TiB   616 GiB   614 GiB      0 B  2.5 GiB  1.1 TiB  34.44  0.78   10      up
                       TOTAL   86 TiB    38 TiB    38 TiB  1.8 MiB  308 GiB   48 TiB  44.29                   
MIN/MAX VAR: 0/2.19  STDDEV: 24.73
```

这里最严重搞得是 13 和 5 这两个 OSD：

```shell
# ceph osd df
ID  CLASS  WEIGHT   REWEIGHT  SIZE     RAW USE   DATA      OMAP     META     AVAIL    %USE   VAR   PGS  STATUS
...	
 5    ssd  1.81940   0.76396  1.8 TiB   1.7 TiB   1.7 TiB      0 B   15 GiB  130 GiB  93.04  2.10   37      up
...
13    ssd  1.74660   0.55382  1.7 TiB   1.7 TiB   1.7 TiB      0 B   14 GiB   54 GiB  96.99  2.19   37      up
...
```

尤其是 13，已经超过了 0.95 的阈值，导致集群已经 ERROR 不可用。

我尝试降低这两个 OSD 的权重：

```shell
ceph osd reweight 13 0.2
ceph osd reweight 5 0.2
```

然后集群开始迁移这两个 OSD 上的数据：

```shell
# ceph -s
  cluster:
    id:     1e2c2112-037f-11ef-b466-45c2ff826e1a
    health: HEALTH_ERR
            1 backfillfull osd(s)
            1 full osd(s)
            Low space hindering backfill (add storage if this doesn't resolve itself): 15 pgs backfill_toofull
            3 pool(s) full
 
  services:
    mon: 5 daemons, quorum c3-hadoop-ceph-st38.bj,c3-hadoop-ceph-st40,c3-hadoop-ceph-st43,c3-hadoop-ceph-st42,c3-hadoop-ceph-st41 (age 2d)
    mgr: c3-hadoop-ceph-st38.bj.urnckz(active, since 3d), standbys: c3-hadoop-ceph-st39.rgvvtd
    osd: 48 osds: 48 up (since 27m), 48 in (since 14m); 38 remapped pgs
 
  data:
    pools:   3 pools, 193 pgs
    objects: 65.91M objects, 13 TiB
    usage:   39 TiB used, 47 TiB / 86 TiB avail
    pgs:     69806767/274612068 objects misplaced (25.420%)
             155 active+clean
             23  active+remapped+backfilling
             15  active+remapped+backfill_toofull
 
  io:
    recovery: 1.6 GiB/s, 10.27k objects/s
 
  progress:
    Global Recovery Event (21h)
      [======================......] (remaining: 5h)
```

但我发现这仍然很慢，很久很久过去了，集群仍然是 ERROR 的不可用状态。所以为了快速恢复集群的可用性，将已经 full 的 OSD 13 权重设为 0，即 out 出集群。

```shell
ceph osd out 13
```

此时集群会立即恢复，变为 warn 状态。然后我继续等待集群恢复。

等待 OSD 13 的用量降低到一定水平后（不会再出现写满风险，例如 50%），将其重新加入集群即可：

```shell
ceph osd in 13
```

这样操作后，backfill 终于可以继续进行了。

我这里后面出现的新的 backfill_toofull 是可以自动恢复的，所以不再需要干预，问题解决 ~~