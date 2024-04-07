





## 1. 测试准备



### 1.1. 清理系统缓存



在每次进行测试以前，我们清理下系统缓存，避免系统缓存对测试结果造成影响：

```shell
sync ; echo 3 | sudo tee /proc/sys/vm/drop_caches
```



### 1.2. 创建测试池



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



这里不讲创建池时各个参数的意义。我这里先创建一个副本池，专门用于 Rados 性能测试用：

```shell
ceph osd pool create test_pool
```





## 2. 测试方法



Ceph 官方提供了 Rados 的性能测试工具，是 `rados` 的子命令：

```shell
rados bench
```

这是一个秒基准测试，测试过程持续数秒即完成。使用方法：

```shell
rados bench -p <pool_name> <seconds> <mode> [options]
```

**必填**参数说明如下 ：

* `pool_name`：测试操作的池名字。
* `seconds`：测试持续时长（单位为秒）。
* `mode`：测试模式。取值有 `write`、`seq` 和 `rand`。其中 `write` 是写（Rados 没有顺序写或随机写之分），`seq` 和 `rand` 分别表示顺序读和随机读。

**选填**参数 `options` 如下：

* `-c`：Ceph 配置文件路径。如果使用的不是默认的 Ceph 配置文件 `/etc/ceph/ceph.conf`，则需要指定。
* `-b`：`write` 模式中操作的块大小（block size），单位字节。默认为 4194304（4 MB）。
* `-O`：`write` 模式中操作的对象大小（object size），单位字节。默认等于 `-b`。
* `--max-objects`：`write` 模式中的最大的对象数量。设定此参数后，写到最大数量测试就会停止（可能会超出一些），否则会继续执行直到达到 `seconds` 时间。
* `-t`：并发操作数。默认为 16。
* `--no-cleanup`：`write` 模式中，在写入测试完成后删除不测试数据。不加此参数时，`write` 测试完成后测试数据会被删除，如果后面还要进行读测试的话，需要加这个参数保留数据。
* `-f`：执行输出的格式。取值有 `plain`（默认）、`json` 和 `json-pretty`。
* `--run-name`：本次测试的名字，这个名字和测试时拼接的 key 名字有关。默认为 "benchmark_last_metadata"。
* `--no-hints`：。
* `--reuse-bench`：`write` 模式中复用上一次的测试对象。不同的测试会有不同的对象前缀，加这个参数以后还用上一次的前缀，但前提是上一次测试加了 `--no-cleanup` 参数。
* `--show-time`：在每一条输出前都加上日期/时间 `data/time` 前缀。
* `--no-verify`：读对象的时候不验证内容。
* `--write-object`：指定向对象中写入的内容。
* `--write-omap`：指定向 omap 中写入的内容。
* `--write-xattr`：指定向 extended attributes 中写入的内容。



---



如果我们在写测试中添加了 `--no-cleanup` 参数不清理数据，那么后面当我们不需要的时候，还需要使用子命令 cleanup 来删除这些数据：

```shell
rados cleanup [--run-name run_name] [--prefix prefix]
```

* `--run-name` ：写入测试时如果添加了这个参数，这里要添加一样的。默认也是 "benchmark_last_metadata"。

* `--prefix`：要清理对象的前缀。

  每次进行写入测试的时候，写入的对象都会有一个前缀，例如 `benchmark_data_myhostname_3742558`，前面是固定的 `benchmark_data_`，中间是机器的 host 名字，末尾是一串数字（看着应该是跟时间有关，因为可以排序），这个前缀会在 `write` 测试时会打印出来。

  当多次执行带有 `--no-cleanup` 的写入测试后，池里就会存在很多个不同的前缀的对象。可以通过指定前缀，清理特定某一次测试写入的数据。如果不指定前缀，则默认清理最后一次写入测试的数据。



## 3. 结果说明



### 3.1. `write`



我这里执行一次 `write` 测试：

```shell
rados bench -p test_pool 10 write -t 16 -b 131072 --no-cleanup
```

输出是下面这样的：

```shell
hints = 1
Maintaining 16 concurrent writes of 131072 bytes to objects of size 131072 for up to 10 seconds or 0 objects
Object prefix: benchmark_data_tj5-s1-v6-tj5-128473-2yqgrsr3_3788036
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    1      16      6653      6637    829.58   829.625  0.00375874  0.00240574
    2      16     13960     13944   871.423   913.375   0.0012494  0.00229252
    3      16     21345     21329   888.635   923.125  0.00188994  0.00224849
    4      16     28714     28698   896.735   921.125  0.00165139  0.00222769
    5      16     36048     36032    900.72    916.75    0.002355   0.0022182
    6      16     43456     43440    904.92       926  0.00147598  0.00220816
    7      15     50729     50714   905.526    909.25   0.0014616  0.00220662
    8      15     58090     58075    907.34   920.125  0.00216009  0.00220221
    9      16     65474     65458   909.055   922.875  0.00238127  0.00219797
   10      13     72671     72658    908.14       900   0.0021631  0.00220023
Total time run:         10.0021
Total writes made:      72671
Write size:             131072
Object size:            131072
Bandwidth (MB/sec):     908.199
Stddev Bandwidth:       28.6869
Max bandwidth (MB/sec): 926
Min bandwidth (MB/sec): 829.625
Average IOPS:           7265
Stddev IOPS:            229.495
Max IOPS:               7408
Min IOPS:               6637
Average Latency(s):     0.0022003
Stddev Latency(s):      0.000743341
Max latency(s):         0.0183621
Min latency(s):         0.000940606
```



---

前两行都是参数说明：

```shell
hints = 1
Maintaining 16 concurrent writes of 131072 bytes to objects of size 131072 for up to 10 seconds or 0 objects
```

然后是一行前缀输出，每次测试的对象名字前缀都不一样：

```shell
Object prefix: benchmark_data_tj5-s1-v6-tj5-128473-2yqgrsr3_3788036
```

接下来是一个实时打印的表格，每秒一行，显示了从测试开始截至这一秒的测试情况（忽略 `Cur`）：

```shell
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    1      16      6653      6637    829.58   829.625  0.00375874  0.00240574
    2      16     13960     13944   871.423   913.375   0.0012494  0.00229252
    3      16     21345     21329   888.635   923.125  0.00188994  0.00224849
    4      16     28714     28698   896.735   921.125  0.00165139  0.00222769
    5      16     36048     36032    900.72    916.75    0.002355   0.0022182
    6      16     43456     43440    904.92       926  0.00147598  0.00220816
    7      15     50729     50714   905.526    909.25   0.0014616  0.00220662
    8      15     58090     58075    907.34   920.125  0.00216009  0.00220221
    9      16     65474     65458   909.055   922.875  0.00238127  0.00219797
   10      13     72671     72658    908.14       900   0.0021631  0.00220023
```

* `sec`：当前执行的秒数。
* `Cur ops`：当前的并发操作数。这个值一般就是 `-t` 的值，可能小一点（跟执行情况有关）。
* `started`：截至当前的已经开始写入的对象数。
* `finished`：截至当前的已经完成写入的对象数。
* `avg MB/s`：平均带宽。单位 MB/s。
* `cur MB/s`：当前带宽。单位 MB/s。
* `last lat(s)`：当前延迟。单位秒。
* `avg lat(s)`：平均延迟。单位秒。



最后是测试总结：

```shell
Total time run:         10.0021
Total writes made:      72671
Write size:             131072
Object size:            131072
Bandwidth (MB/sec):     908.199
Stddev Bandwidth:       28.6869
Max bandwidth (MB/sec): 926
Min bandwidth (MB/sec): 829.625
Average IOPS:           7265
Stddev IOPS:            229.495
Max IOPS:               7408
Min IOPS:               6637
Average Latency(s):     0.0022003
Stddev Latency(s):      0.000743341
Max latency(s):         0.0183621
Min latency(s):         0.000940606
```

* `Total time run`：总测试运行时间。
* `Total writes made`：总测试写入对象数。
* `Write size`：写入时的 block 大小。单位字节。
* `Object size`：写入时的对象大小。单位字节。
* `Bandwidth (MB/sec)`：平均带宽。单位 MB/sec。
* `Stddev Bandwidth`：带宽标准差。
* `Max bandwidth (MB/sec)`：最大带宽。单位 MB/sec。
* `Min bandwidth (MB/sec)`：最小带宽。单位 MB/sec。
* `Average IOPS`：平均 IOPS。
* `Stddev IOPS`：IOPS 标准差。
* `Max IOPS`：最大 IOPS。
* `Min IOPS`：最小 IOPS。
* `Average Latency(s)`：平均延迟。单位秒。
* `Stddev Latency(s)`：延迟标准差。单位秒。
* `Max latency(s)`：最大延迟。单位秒。
* `Min latency(s)`：最小延迟。单位秒。



### 3.2. `seq` 和 `rand`



我这里执行一次 `seq` （`rand` 是一样的）测试：

```shell
rados bench -p test_pool 10 seq -t 16
```

输出是下面这样的：

```shell
hints = 1
  sec Cur ops   started  finished  avg MB/s  cur MB/s last lat(s)  avg lat(s)
    0       0         0         0         0         0           -           0
    1      16      3484      3468   433.421     433.5 0.000800011  0.00443406
    2      16      6561      6545   408.996   384.625 0.000791454  0.00474372
    3      16      9666      9650   402.023   388.125 0.000864584  0.00494705
    4      16     12942     12926   403.881     409.5 0.000980328  0.00491885
    5      16     16101     16085   402.068   394.875 0.000980993  0.00493039
    6      16     19359     19343   402.924    407.25 0.000689757  0.00493172
    7      16     22695     22679   404.929       417 0.000779945  0.00490919
    8      16     25820     25804   403.135   390.625 0.000879283  0.00493282
    9      16     29250     29234   405.974    428.75  0.00084602  0.00489478
   10      11     32263     32252   403.097    377.25  0.00129956  0.00493978
Total time run:       10.0272
Total reads made:     32263
Read size:            131072
Object size:          131072
Bandwidth (MB/sec):   402.193
Average IOPS:         3217
Stddev IOPS:          152.968
Max IOPS:             3468
Min IOPS:             3018
Average Latency(s):   0.00494957
Max latency(s):       0.11724
Min latency(s):       0.000266814
```



这里面的值都和 write 是一样的（除了 Write 相关字样换成了 Read）。

缺少的有一开始的参数说明、前缀显示（因为写才有，读没有），还有 4 个结果字段：

* `Stddev Bandwidth`
* `Max bandwidth (MB/sec)`
* `Min bandwidth (MB/sec)`
* `Stddev Latency(s)`

其他都是一样的，这里就不写了。
