



参考文档：

* https://docs.ceph.com/en/pacific/rados/operations/pools/

* https://docs.ceph.com/en/pacific/rados/operations/erasure-code/





## 1. 主要概念



**池（Pool）**是用于存储对象的逻辑分区。



池提供以下几个特性：

1. **弹性（Resilience，指恢复数据的能力）：**

   * 对于副本池，弹性的定义为，可以保证发生故障而不丢失任何数据的 OSD 的数量。在多副本策略的池中，这个保证不丢失数据的 OSD 数量就是副本数量。
   * 对于纠删码池，弹性的定义为，编码块的数量（例如，纠删码配置文件中的 m = 2）。

2. **PG（Placement Group，放置组）：**

   * 一个池存储了很多很多对象，为了更好的管理这些对象，Ceph 会把这些对象进行分组，每个组就是一个 PG。

   * 一个池可以有很多 PG，每个 PG 只能归属一个池。

     当多个池使用相同的 OSD 时，我们需要确保每个 OSD 的 PG 副本总和处于每个 OSD 所需的 PG 数量范围内。

   * 典型配置的目标是每个 OSD 大约 100 个 PG，从而在不消耗大量计算资源的情况下提供最佳平衡。

   * 当集群内有多个池时，需哟小心为每个池和整个集群设置合理数量的 PG。

3. **CRUSH 规则：**

   * 当数据存储在池中时，集群中对象及其副本（或 chunks，在纠删码池的情况下）的放置由 CRUSH 规则控制。
   * 如果默认规则不适合我们的用场景，可以为池创建自定义 CRUSH 规则。

4. **快照：**可以给池创建快照。命令 `ceph osd pool mksnap`。



## 2. 全局参数





下面这些全局参数，都可以使用下面的命令读取或修改：



* 获取参数值：

  ```shell
  ceph config get osd <parameter_name>
  ```

* 修改参数值：

  ```shell
  ceph config set osd <parameter_name>
  ```



| 序号 | 参数                                  | 描述                                                         | 类型                    | 默认值                                                       |
| ---- | ------------------------------------- | ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| 1    | `mon_max_pool_pg_num`                 | 每个 Pool 的 PG 的最大数量。                                 | Integer                 | 65536                                                        |
| 2    | `mon_pg_create_interval`              | 在同一 Ceph OSD 守护进程中创建 PG 的间隔（单位：秒）。       | Float                   | 30.0                                                         |
| 3    | `mon_pg_stuck_threshold`              | PG 被视为卡住的秒数。                                        | 32-bit Integer          | 300                                                          |
| 4    | `mon_pg_min_inactive`                 | 如果不活动时间超过 `mon_pg_stuck_threshold` 的 PG 数量超过此设置，则引发 `HEALTH_ERR`。非正数意味着禁用，永远不会进入 ERR。 | Integer                 | 1                                                            |
| 5    | `mon_pg_warn_min_per_osd`             | 如果每个 OSD 中 PG 平均数量低于此数字，则引发 `HEALTH_WARN`。非正数会禁用此功能。 | Integer                 | 30                                                           |
| 6    | `mon_pg_warn_min_objects`             | 如果集群中 RADOS 对象的总数低于此数字，则不发出警告。        | Integer                 | 1000                                                         |
| 7    | `mon_pg_warn_min_pool_objects`        | 如果一个池子中的中 RADOS 对象的总数低于此数字，则不发出警告。 | Integer                 | 1000                                                         |
| 8    | `mon_pg_check_down_all_threshold`     | 状态为 `down` 的 OSD 的百分比阈值，高于该阈值 Ceph 将检查所有 PG 是否已过时。 | Float                   | 0.5                                                          |
| 9    | `mon_pg_warn_max_object_skew`         | 如果任意一个池中每个 PG 的平均 RADOS 对象数量大于 `mon_pg_warn_max_object_skew` 乘以所有池中每个 PG 的平均 RADOS 对象数量，则引发 `HEALTH_WARN`。零或非正数会禁用此功能。请注意，此选项适用于 `ceph-mgr` 守护进程。 | Float                   | 10                                                           |
| 10   | `mon_delta_reset_interval`            | 在将 PG 增量重置为 0 之前不活动的秒数。Ceph 跟踪每个Pool 的已用空间增量，因此我们可以更轻松地了解恢复进度或缓存层的性能。但是，如果某个池没有报告任何活动，我们只需重置该池的增量历史记录。 | Integer                 | 10                                                           |
| 11   | `mon_osd_max_op_age`                  | 请求可以接受的被阻塞的最长时间。如果一个请求被阻塞的时间超过此限制，则会引发 `HEALTH_WARN`。这个值应该设置为 2 的幂。 | Float                   | 32.0                                                         |
| 12   | `osd_pg_bits`                         | 每个 OSD 可以管理的 PG 数量的位数。即每个 OSD 可以管理 $2^{osd\_pg\_bits}$ 个 PG。 | 32-bit Integer          | 6                                                            |
| 13   | `osd_pgp_bits`                        | 每个 OSD 可以管理的 PGG 数量的位数。即每个 OSD 可以管理 $2^{osd\_pgp\_bits}$ 个 PGP。 | 32-bit Integer          | 6                                                            |
| 14   | `osd_crush_chooseleaf_type`           | CRUSH 规则中用于 `chooseleaf` 的存储桶类型。使用序数排名而不是名称。 | 32-bit Integer          | 1                                                            |
| 15   | `osd_crush_initial_weight`            | 新添加的 OSD 的初始 CRUSH 权重。                             | Double                  | 新添加的 OSD 的大小（以 TB 为单位）。                        |
| 16   | `osd_pool_default_crush_rule`         | 创建副本池时使用的默认 CRUSH 规则。                          | 8-bit Integer           | -1，表示“选择数字 ID 最小的规则并使用它”。这是为了在没有规则 0 的情况下创建池。 |
| 17   | `osd_pool_erasure_code_stripe_unit`   |                                                              |                         |                                                              |
| 18   | `osd_pool_default_size`               | 设置 Pool 中对象的副本数。默认值与 `ceph osd pool set {pool-name} size {size}` 相同。 | 32-bit Integer          | 3                                                            |
| 19   | `osd_pool_default_min_size`           | 设置 Pool 中对象的最小写入副本数，以便确认对客户端的写入操作。如果未满足最小值，Ceph 将不会向客户端确认写入，这可能会导致数据丢失。此设置可确保在降级模式下运行时的副本数量最少。 | 32-bit Integer          | 0，这意味着没有特定的最小值。如果为 0，则最小值为`size - (size / 2)`。 |
| 20   | `osd_pool_default_pg_num`             | Pool 的默认 PG 数量。默认值与 `mkpool` 的 `pg_num` 相同。    | 32-bit Integer          | 32                                                           |
| 21   | `osd_pool_default_pgp_num`            | Pool 的默认 PGP 数量。默认值与 `mkpool` 的 `pgp_num` 相同。 PG 和 PGP 应该相等（目前）。 | 32-bit Integer          | 8                                                            |
| 22   | `osd_pool_default_flags`              | 新 Pool 的默认 Flag。                                        | 32-bit Integer          | 0                                                            |
| 23   | `osd_max_pgls`                        | 客户端请求列出 PG 时，要列出的 PG 最大数量。请求大量数据的客户端可能会占用 Ceph OSD 守护进程。 | Unsigned 64-bit Integer | 1024（不建议修改）                                           |
| 24   | `osd_min_pg_log_entries`              | 清理日志时，PG 日志保留的最小数量。                          | 32-bit Int Unsigned     | 250                                                          |
| 25   | `osd_max_pg_log_entries`              | 清理日志时，PG 日志保留的最大数量。                          | 32-bit Int Unsigned     | 10000                                                        |
| 26   | `osd_default_data_pool_replay_window` | OSD 等待客户端重播请求的时间（以秒为单位）。                 | 32-bit Integer          | 45                                                           |
| 27   | `osd_max_pg_per_osd_hard_ratio`       | 集群允许每个 OSD 的 PG 数量的比率（超过的 PG 创建请求会被拒绝）。如果 OSD 服务的 PG 数量超过 `osd_max_pg_per_osd_hard_ratio * mon_max_pg_per_osd`，则 OSD 将停止创建新 PG。 | Float                   | 2                                                            |
| 28   | `osd_recovery_priority`               | 工作队列中恢复的优先级。                                     | Integer                 | 5                                                            |
| 29   | `osd_recovery_op_priority`            | 用于恢复操作的默认优先级（如果 Pool 不覆盖）。               | Integer                 | 3                                                            |



## 3. 常用操作



### 3.1. 选择 Pool 的 PG 和 PGP 数量



官方建议是根据 OSD 的数量来选择 Pool 的 PG / PGP 数量的，从 OSD 为 50 做了个分界。



> 在我的理解中，PG 的合理数量选择是不太好估算的，下面要介绍的官方说法也是经验学说。
>
> 使用官方算法计算出的 PG 数量，或者是一些官方样例给出的 PG 数量，也经常没有在官方建议的合理区间。
>
> 我感觉这个 PG 数量的值，到底应该设置多少，还是需要在生产环境中积累经验，逐渐调整。



#### 3.1.1. OSD 数量 ≤ 50



在 OSD 数量比较少时（≤ 50），官方建议使用自动调整。

```shell
# 调整现有 Pool
ceph osd pool set <pool-name> pg_autoscale_mode <mode>
```

```shell
# 调整全局设置，只对后面新创建的 Pool 生效
ceph config set global osd_pool_default_pg_autoscale_mode <mode>
```

`mode` 的取值有三种：

1. `off`：禁用自动调整池的 PG 数量，由管理员手动设定。
2. `on`：启用自动调整池的 PG 数量。
3. `warn`：使用与 `on` 模式同样的算法，但 Ceph 认为应当调整时，发出健康警报，但不会自动调整。

可以通过命令查看自动调整的状态：

```shell
ceph osd pool autoscale-status
```

```
POOL    SIZE  TARGET SIZE  RATE  RAW CAPACITY   RATIO  TARGET RATIO  EFFECTIVE RATIO BIAS PG_NUM  NEW PG_NUM  AUTOSCALE BULK
a     12900M                3.0        82431M  0.4695                                          8         128  warn      True
c         0                 3.0        82431M  0.0000        0.2000           0.9884  1.0      1          64  warn      True
b         0        953.6M   3.0        82431M  0.0347                                          8              warn      False
```



> 官方还提供了一个 flag `noautoscale`，表示禁用自动调整功能。
>
> 与上面设置 Pool 的是否自动调整参数不同，`noautoscale` 是在全局把这个功能关了，`pg_autoscale_mode`  是对 Pool 设定是否启用这个功能。
>
> `noautoscale` 默认值为 `off`，即启用自动调整功能。
>
> ```shell
> # 启用自动调整功能，noautoscale 设为 off
> ceph osd pool set noautoscale
> ```
>
> ```shell
> # 禁用自动调整功能，noautoscale 设为 on
> ceph osd pool unset noautoscale
> ```
>
> ```shell
> # 查看当前 noautoscale 的值
> ceph osd pool get noautoscale
> ```



#### 3.1.2. OSD 数量 ＞ 50



官方的建议是这样的：

1. 每个 OSD 有 50 - 100 个的 PG 是比较合理的。
2. PG 和 PGP 的数量应该是一样的。
3. PG 和 PGP 的数量应该始终是 2 的幂数（取最近的）。



官方给出的 PG / PGP 的数量计算方法是这样的：
$$
\frac{osd\_num \times 100}{pool\_size}
$$

即 OSD 数量乘 100 再除以 Pool 的副本数：

* 副本池比较简单，就是副本数。
* 纠删码池，则等于 k + m（可由命令 `ceph osd erasure-code-profile get <ec_profile_name>` 查看）。



比如有 200 个 OSD，使用副本池，副本数为 3，那么 PG 的合理数量应该是：

200 * 100 / 3 = 6667，然后取接近的 2 次幂是 8192。

然后就这样修改配置：

```shell
ceph config set osd osd_pool_default_pg_num 8192
ceph config set osd osd_pool_default_pgp_num 8192
```

修改后，这些设置会应用到使用默认配置新创建的 Pool 上。



> 其实这里就是我前面提到的情况，官方算法的计算结果也不一定在官方建议的合理范围内。
>
> 官方说根据经验，每个 OSD 有 50 - 100 个 PG 比较合理，但这里的平均每个 OSD 有 8192 / 200 = 40.96 个 PG。



### 3.2. 创建 Pool



> 在创建 Pool 以前，建议先看下上一节，关注下如何设置 Pool 的 PG 和 PGP 数量。



Ceph 的 Pool 有两种策略，分别是副本池和纠删码池，创建方法如下：

```shell
# 副本池
ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]] [replicated] \
     [crush-rule-name] [expected-num-objects]
```

```shell
# 纠删码池
ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]]   erasure \
     [erasure-code-profile] [crush-rule-name] [expected_num_objects] [--autoscale-mode=<on,off,warn>]
```

其中 `[]` 中的项是可选参数的，不填将使用全局值。

选填参数说明如下：

| 序号 | 参数及说明                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | `pool-name`<br/>\* 描述：Pool 的名称，必须是唯一的。<br />\* 类型：字符串。<br>\* 必填：是。 |
| 2    | `pg-num`<br/>\* 描述：Pool 的 PG 总数。注意默认值不适合大多数场景，参照上面 3.1 设置。<br />\* 类型：整数。<br/>\* 必填：否。<br />\* 默认值：8。 |
| 3    | `pgp-num`<br/>\* 描述：Pool 的 PPG 总数。大部分场景下**应等于 PG 总数**，<br />\* 类型：整数<br/>\* 必填：否。<br />\* 默认值：8。 |
| 4    | `replicated` `erasure`<br/>\* 描述：Pool 类型。指定是副本池（`replicated`）还是纠删码池（`erasure`）。<br />\* 类型：字符串<br/>\* 必填：否。<br />\* 默认值：`replicated`。 |
| 5    | `crush-rule-name`<br/>\* 描述：要用于此池的 CRUSH 规则的名称。指定的规则必须存在。<br />\* 类型：字符串<br/>\* 必填：否。<br />\* 默认值：副本池由 config 变量指定，指定的规则必须存在。 纠删码池，指的是是否使用纠删码配置文件。如果没有设置过任何规则，则将隐式创建 `osd pool default crush rule erasure-code default{pool-name}`。 |
| 6    | `erasure-code-profile=profile`<br/>\* 描述：仅适用纠删码池，使用纠删码配置文件。这个必须是 `osd erasure-code-profile set` 已经定义过的存在的配置。<br />\* 类型：字符串。<br/>\* 必填：否。 |
| 7    | `--autoscale-mode=<on,off,warn>`<br/>\* 描述：自动调整 PG 数量的模式。<br />\* 类型：字符串。<br/>\* 必填：否。<br />\* 默认值：默认行为由选项 `osd_pool_default_pg_autoscale_mode` 控制。 |
| 8    | `expected-num-objects`<br/>\* 描述：池的预期对象数量。通过设置这个值（和一个负文件存储合并阈值一起），在池创建的时候就会预先划分创建好 PG 文件夹，避免在运行时增加延迟。<br />\* 类型：整数。<br/>\* 必填：否。<br />\* 默认值：0，不在池创建时划分 PG 文件夹。 |

