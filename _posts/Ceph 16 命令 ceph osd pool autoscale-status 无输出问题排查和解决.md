今天想在一个集群看下 pg 自动扩缩的情况，结果发现没有任何输出：

```shell
# ceph osd pool autoscale-status
# 
```



## 1. 问题排查

确认 pg 自动扩缩这个功能已经打开了：

```shell
# ceph mgr module enable pg_autoscaler
module 'pg_autoscaler' is already enabled (always-on)
```

```shell
# ceph osd pool get noautoscale
noautoscale is off
```

确认每个池都启用了 pg 自动扩缩功能（省略了不需要的输出）：

```shell
# ceph osd pool ls detail
pool 1 'device_health_metrics' replicated ... autoscale_mode on ...
pool 2 'rp' replicated ... autoscale_mode on ...
pool 3 'ec' erasure ... autoscale_mode on ...
```

所以设置方面是没什么问题的，肯定是被其他东西影响了。

跟踪检查了下 mgr 日志，发现了下面这一段 debug 内容：

```shell
# docker logs -f `docker ps | grep ceph-mgr | cut -d ' ' -f 1`
...
debug 2024-04-28T10:53:26.156+0000 7f613c019700  0 [pg_autoscaler WARNING root] pool ec won't scale due to overlapping roots: {-1, -2}
debug 2024-04-28T10:53:26.156+0000 7f613c019700  0 [pg_autoscaler WARNING root] Please See: https://docs.ceph.com/en/latest/rados/operations/placement-groups/#automated-scaling
debug 2024-04-28T10:53:26.160+0000 7f613c019700  0 [pg_autoscaler WARNING root] pool 1 contains an overlapping root -1... skipping scaling
debug 2024-04-28T10:53:26.160+0000 7f613c019700  0 [pg_autoscaler WARNING root] pool 2 contains an overlapping root -1... skipping scaling
debug 2024-04-28T10:53:26.160+0000 7f613c019700  0 [pg_autoscaler WARNING root] pool 3 contains an overlapping root -2... skipping scaling
...
```

mgr 的 debug 日志，说池 ec 因为重叠的 roots {-1, -2} 而导致无法自动扩展 PG，然后是说池 1、2、3，即所有的池都因为包含重叠的 root 导致跳过了 PG 扩展。这看起来就是问题的主要原因了。

前面 `ceph osd pool ls detail` 的输出可以看到，我这里有三个池：

1. `device_health_metrics`：Ceph 内建的池。
2. `rp`：一个副本池。
3. `ec`：一个纠删码池。



我查了一下集群中的 CRUSH 规则：

```shell
[
    {
        "rule_id": 0,
        "rule_name": "replicated_rule",
        "ruleset": 0,
        "type": 1,
        "min_size": 1,
        "max_size": 10,
        "steps": [
            {
                "op": "take",
                "item": -1,
                "item_name": "default"
            },
            {
                "op": "chooseleaf_firstn",
                "num": 0,
                "type": "host"
            },
            {
                "op": "emit"
            }
        ]
    },
    {
        "rule_id": 1,
        "rule_name": "ec",
        "ruleset": 1,
        "type": 3,
        "min_size": 3,
        "max_size": 6,
        "steps": [
            {
                "op": "set_chooseleaf_tries",
                "num": 5
            },
            {
                "op": "set_choose_tries",
                "num": 100
            },
            {
                "op": "take",
                "item": -2,
                "item_name": "default~ssd"
            },
            {
                "op": "choose_indep",
                "num": 0,
                "type": "osd"
            },
            {
                "op": "emit"
            }
        ]
    }
]
```



从规则信息可以看到，规则 replicated_rule 的 `item` 为 -1，`item_name` 为 default。规则 ec 的 `item` 为 -2，`item_name` 为 default~ssd。这其实是说这两个规则分别使用设备类（device class） default 和设备类 default~ssd，但因为我这个集群中只有一种设备 ssd，所以这两个规则的 default 和 default~ssd 是等价的，所以出现了 -1 和 -2 这两个不同的 root 就是冲突的。更进一步的理解涉及 CRUSH 算法，这里暂不讨论。

这也就应了日志中的 ：

```shell
... [pg_autoscaler WARNING root] pool ec won't scale due to overlapping roots: {-1, -2}
```



另外，这个 default~ssd 其实是来自 ec 这个纠删码池的配置，我这里配置名字叫 eck4m2，是一个 EC 4+2 的池。查看一下配置：

```shell
# ceph osd erasure-code-profile get eck4m2
crush-device-class=ssd
crush-failure-domain=osd
crush-root=default
directory=/usr/lib64/ceph/erasure-code
jerasure-per-chunk-alignment=false
k=4
m=2
packetsize=2048
plugin=jerasure
technique=reed_sol_van
w=8
```

可以看到 default~ssd 其实是字段 `crush-root` 和 `crush-device-class` 拼起来的结果。



## 2. 问题解决



归根结底，导致 CRUSH 根重叠的原因是，多个规则使用同一设备类（配置中的 `item_name`），且设置了重叠的 root（配置中的 `item`）。正常情况下，应该是这样的：

* 当多个 CRUSH 规则使用同一设备类时，应当有不重叠的 root。
* 不同设备类之间的 root 相互独立，互不影响。



因为我的集群只有一个设备类，所以要保证多个 CRUSH 的 `item_name` 一样或者推断一样（比如我这里只有 ssd 设备，那么设备类 default 和 default~ssd 是等价的）的前提下，有不重叠的 root。

最简单的解决办法，就是新建一个 ec profile（注意不要指定 `crush-device-class` 字段，否则又会拼接出 default~*，其他字段按需设置）：

```shell
# osd erasure-code-profile set <name> [<profile>...] [--force]
ceph osd erasure-code-profile set eck4m2-new k=4 m=2
```

然后根据刚新建的 ec profile 创建新的 CRUSH 规则：

```shell
# osd crush rule create-erasure <name> [<profile>] 
ceph osd crush rule create-erasure ec-new eck4m2-new
```

最后给我们出问题的池应用新的 CRUSH 规则，我这里是给池 ec 应用规则 ec-new：

```shell
# ceph osd pool set <pool_name> crush_rule <crush_rule_name>
ceph osd pool set ec crush_rule ec-new
```



到这里问题就解决了，可以再次看下 `ceph osd pool autoscale-status` 的输出，符合预期：

```shell
# ceph osd pool autoscale-status
POOL                     SIZE  TARGET SIZE  RATE  RAW CAPACITY   RATIO  TARGET RATIO  EFFECTIVE RATIO  BIAS  PG_NUM  NEW PG_NUM  AUTOSCALE  BULK   
device_health_metrics  244.8k                3.0        88232G  0.0000                                  1.0       1              on         False  
rp                      9679G                3.0        88232G  0.3291                                  1.0     512              on         False  
ec                      3161G                1.5        88232G  0.0537                                  1.0      32              on         False 
```

至于废弃掉的旧 ec profile 和 CRUSH 规则，删不删的就随意了。



## 3. 问题复盘



这个问题本质上是 CRUSH 规则中的设备类（Crush device class）设置的不对。

那么为什么不对，经过我的排查，发现使用 Dashboard 创建**自定义配置的**纠删码池时此问题最高发。原因是在 dashboard 创建自定义配置时，`crush-device-class` 字段的值默认会被设上（在命令行创建的时候一般不会特意指定这个值）：

![](https://gukaifeng.cn/posts/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue_1.png)

我们手动给设成 Let Ceph decide 即可不再出现此问题：

![](https://gukaifeng.cn/posts/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue_2.png)



> 其实一开始的时候有搜索到红帽一个解决方案：
>
> https://access.redhat.com/solutions/6987628
>
> 但是由于这个页面只有红帽开发者才能看，等我加入红帽开发者的申请通过的时候，我已经搞定了 =。=
>
> <img width=300px src="https://gukaifeng.cn/posts/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue/ceph-16-ming-ling-ceph-osd-pool-autoscale-status-wu-shu-chu-wen-ti-pai-cha-he-jie-jue_3.jpg" />