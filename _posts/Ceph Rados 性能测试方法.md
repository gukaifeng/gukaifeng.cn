

## 1. 测试工具



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
* `-O`：`write` 模式中操作的对象大小（object size），单位字节。默认和 `-b` 一样。
* `--max-objects`：`write` 模式中的最大的对象数量。设定此参数后，写到最大数量测试就会停止（可能会超出一些），否则会继续执行直到达到 `seconds` 时间。
* `-t`：并发操作数。默认为 16。
* `--no-cleanup`：`write` 模式中，在写入测试完成后删除不测试数据。不加此参数时，`write` 测试完成后测试数据会被删除，如果后面还要进行读测试的话，需要加这个参数保留数据。
* `-f`：执行输出的格式。取值有 `plain`（默认）、`json` 和 `json-pretty`。
* `--run-name`：本次测试的名字，这个名字和测试时拼接的 key 名字有关。默认为 "benchmark_last_metadata"。
* `--no-hints`：。
* `--reuse-bench`：。
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

  每次进行写入测试的时候，写入的对象都会有一个前缀，例如 `benchmark_data_myhostname_3742558`，前面是固定的 `benchmark_data_`，中间是机器的 host 名字，末尾是一串数组（看着应该是跟时间有关，因为可以排序），这个前缀会在 `write` 测试时会打印出来。

  当多次执行带有 `--no-cleanup` 的写入测试后，池里就会存在很多个不同的前缀的对象。如果不指定前缀，则默认清理最后一次写入测试的数据。





## 2. 测试准备



### 2.1. 清理系统缓存



在每次进行测试以前，我们清理下系统缓存，避免缓存对测试结果造成影响：

```shell
sync ; echo 3 | sudo tee /proc/sys/vm/drop_caches
```



### 2.2. 创建测试池



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



这里不讲池怎么创建以及创建池时各个参数的意义。

我这里先创建一个副本池，专门用于 Rados 性能测试用：

```shell
ceph osd pool create test_pool
```

