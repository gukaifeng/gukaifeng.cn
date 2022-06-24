---
title: YCSB 的介绍与使用方法
date: 2022-06-23 18:54:08
updated: 2022-06-23 18:54:08
categories: [数据库]
tags: [数据库,YCSB,Benchmark]
---





> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>





## 1. 什么是 YCSB



**YCSB** 全称 **Yahoo! Cloud Serving Benchmark**，即是一个雅虎云基准测试工具。



根据官方 wiki 描述，其现支持以下数据库：

1. [HBase](http://hbase.apache.org/)
2. [Hypertable](http://hypertable.org/)
3. [Cassandra](http://incubator.apache.org/cassandra/)
4. [Couchbase](http://couchbase.com/)
5. [Voldemort](http://www.project-voldemort.com/)
6. [MongoDB](http://www.mongodb.org/)
7. [OrientDB](http://www.orientdb.org/)
8. [Infinispan](http://www.infinispan.org/)
9. [Redis](http://redis.io/)
10. [GemFire](http://www.vmware.com/products/application-platform/vfabric-gemfire/overview.html)
11. [DynamoDB ](http://aws.amazon.com/dynamodb/)
12. [Tarantool](http://tarantool.org/)
13. [Memcached ](http://memcached.org/)

… 等等更多（上述列出的数据库类型均为 NoSQL）。



我们很难知道哪个存储系统更适合我们的应用程序，一部分原因是不同的系统有不同特性，一部分原因是比较存储系统的性能比较困难。而 YCSB 主要用于解决后者 —— 测试某些存储系统的性能。

YCSB 项目的目标是开发一套框架和通用的工作负载，以评估不同的 “key-value” 和 “云” 服务的性能。该项目包括两件事：

1. YCSB 客户端，一个可扩展的工作负载生成器。
2. 核心工作负载，一组将由生成器执行的工作负载方案。



尽管 YCSB 核心工作负载提供了对一个系统性能的全面分析，但客户端是可扩展的，因此你可以定义新的不同的工作负载来检查核心工作负载未充分涵盖的系统方面或应用场景。同样，客户端可扩展以支持对不同数据库进行基准测试。

YCSB 中已经包含了用于对 HBase、Cassandra、Infinispan 和 MongoDB 进行基准测试的示例代码，但编写一个新的接口层来对我们喜欢的数据库进行基准测试是很简单的。

YCSB 工具的常用用途是对多个存储系统进行基准测试，并进行这些系统。例如，你可以在相同的硬件配置上安装多个系统，并针对每个系统运行相同的工作负载。然后，你可以绘制每个系统的性能（例如，作为延迟与吞吐量曲线），以查看一个系统何时比另一个系统更好。

[这里](http://dl.acm.org/citation.cfm?id=1807152)是一篇描述了这个基准测试和一些结果的论文。

wiki 中还给出了一篇使用 YCSB 对 Redis 进行基准测试的文档 —— http://robertlehmann.de/img/redis.pdf。



> 注：YCSB 最新的 Release 版本是 [0.17.0](https://github.com/brianfrankcooper/YCSB/releases/tag/0.17.0)，发布时间是 2019 年 10 月 6 日。





## 2. 开始使用

> 本文将**略过**在 <font color=red>Windows</font> 上使用 YCSB 的方法，仅在 <font color=green>Linux</font>。  
> 如果你需要在 Windows 上使用 YCSB，请自行查看[官方 wiki](https://github.com/brianfrankcooper/YCSB/wiki/Getting-Started#1-obtain-ycsb) 中的内容。



### 2.1. 获取 YCSB

目前最新版就是 0.17.0，我们直接下载这个版本就好。

```shell
curl -O --location https://github.com/brianfrankcooper/YCSB/releases/download/0.17.0/ycsb-0.17.0.tar.gz
tar xfvz ycsb-0.17.0.tar.gz
cd ycsb-0.17.0
```

（这个 `ycsb-0.17.0.tar.gz` 足足有 675 MB，可以自己根据实际情况调整下载方案。)

运行 `./bin/ycsb` 命令但不给出任何参数，将打印用法。

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z ycsb-0.17.0]$ ./bin/ycsb
usage: ./bin/ycsb command database [options]

Commands:
    load           Execute the load phase
    run            Execute the transaction phase
    shell          Interactive mode

Databases:
    accumulo       https://github.com/brianfrankcooper/YCSB/tree/master/accumulo
    accumulo1.6    https://github.com/brianfrankcooper/YCSB/tree/master/accumulo1.6
    accumulo1.7    https://github.com/brianfrankcooper/YCSB/tree/master/accumulo1.7
    accumulo1.8    https://github.com/brianfrankcooper/YCSB/tree/master/accumulo1.8
    aerospike      https://github.com/brianfrankcooper/YCSB/tree/master/aerospike
    arangodb       https://github.com/brianfrankcooper/YCSB/tree/master/arangodb
    arangodb3      https://github.com/brianfrankcooper/YCSB/tree/master/arangodb3
    asynchbase     https://github.com/brianfrankcooper/YCSB/tree/master/asynchbase
    azurecosmos    https://github.com/brianfrankcooper/YCSB/tree/master/azurecosmos
    azuretablestorage https://github.com/brianfrankcooper/YCSB/tree/master/azuretablestorage
    basic          https://github.com/brianfrankcooper/YCSB/tree/master/basic
    basicts        https://github.com/brianfrankcooper/YCSB/tree/master/basicts
    cassandra-cql  https://github.com/brianfrankcooper/YCSB/tree/master/cassandra
    cassandra2-cql https://github.com/brianfrankcooper/YCSB/tree/master/cassandra2
    cloudspanner   https://github.com/brianfrankcooper/YCSB/tree/master/cloudspanner
    couchbase      https://github.com/brianfrankcooper/YCSB/tree/master/couchbase
    couchbase2     https://github.com/brianfrankcooper/YCSB/tree/master/couchbase2
    crail          https://github.com/brianfrankcooper/YCSB/tree/master/crail
    dynamodb       https://github.com/brianfrankcooper/YCSB/tree/master/dynamodb
    elasticsearch  https://github.com/brianfrankcooper/YCSB/tree/master/elasticsearch
    elasticsearch5 https://github.com/brianfrankcooper/YCSB/tree/master/elasticsearch5
    elasticsearch5-rest https://github.com/brianfrankcooper/YCSB/tree/master/elasticsearch5
    foundationdb   https://github.com/brianfrankcooper/YCSB/tree/master/foundationdb
    geode          https://github.com/brianfrankcooper/YCSB/tree/master/geode
    googlebigtable https://github.com/brianfrankcooper/YCSB/tree/master/googlebigtable
    googledatastore https://github.com/brianfrankcooper/YCSB/tree/master/googledatastore
    griddb         https://github.com/brianfrankcooper/YCSB/tree/master/griddb
    hbase098       https://github.com/brianfrankcooper/YCSB/tree/master/hbase098
    hbase10        https://github.com/brianfrankcooper/YCSB/tree/master/hbase10
    hbase12        https://github.com/brianfrankcooper/YCSB/tree/master/hbase12
    hbase14        https://github.com/brianfrankcooper/YCSB/tree/master/hbase14
    hbase20        https://github.com/brianfrankcooper/YCSB/tree/master/hbase20
    hypertable     https://github.com/brianfrankcooper/YCSB/tree/master/hypertable
    ignite         https://github.com/brianfrankcooper/YCSB/tree/master/ignite
    ignite-sql     https://github.com/brianfrankcooper/YCSB/tree/master/ignite
    infinispan     https://github.com/brianfrankcooper/YCSB/tree/master/infinispan
    infinispan-cs  https://github.com/brianfrankcooper/YCSB/tree/master/infinispan
    jdbc           https://github.com/brianfrankcooper/YCSB/tree/master/jdbc
    kudu           https://github.com/brianfrankcooper/YCSB/tree/master/kudu
    maprdb         https://github.com/brianfrankcooper/YCSB/tree/master/maprdb
    maprjsondb     https://github.com/brianfrankcooper/YCSB/tree/master/maprjsondb
    memcached      https://github.com/brianfrankcooper/YCSB/tree/master/memcached
    mongodb        https://github.com/brianfrankcooper/YCSB/tree/master/mongodb
    mongodb-async  https://github.com/brianfrankcooper/YCSB/tree/master/mongodb
    nosqldb        https://github.com/brianfrankcooper/YCSB/tree/master/nosqldb
    orientdb       https://github.com/brianfrankcooper/YCSB/tree/master/orientdb
    postgrenosql   https://github.com/brianfrankcooper/YCSB/tree/master/postgrenosql
    rados          https://github.com/brianfrankcooper/YCSB/tree/master/rados
    redis          https://github.com/brianfrankcooper/YCSB/tree/master/redis
    rest           https://github.com/brianfrankcooper/YCSB/tree/master/rest
    riak           https://github.com/brianfrankcooper/YCSB/tree/master/riak
    rocksdb        https://github.com/brianfrankcooper/YCSB/tree/master/rocksdb
    s3             https://github.com/brianfrankcooper/YCSB/tree/master/s3
    solr           https://github.com/brianfrankcooper/YCSB/tree/master/solr
    solr6          https://github.com/brianfrankcooper/YCSB/tree/master/solr6
    tablestore     https://github.com/brianfrankcooper/YCSB/tree/master/tablestore
    tarantool      https://github.com/brianfrankcooper/YCSB/tree/master/tarantool

Options:
    -P file        Specify workload file
    -cp path       Additional Java classpath entries
    -jvm-args args Additional arguments to the JVM
    -p key=value   Override workload property
    -s             Print status to stderr
    -target n      Target ops/sec (default: unthrottled)
    -threads n     Number of client threads (default: 1)

Workload Files:
    There are various predefined workloads under workloads/ directory.
    See https://github.com/brianfrankcooper/YCSB/wiki/Core-Properties
    for the list of workload properties.
ycsb: error: too few arguments
```



> 注：在 ./bin 目录下，有 3 个 ycsb 相关的可执行程序(脚本)：
>
> 1. `ycsb`：这是一个 python2 的脚本，注意是 2。
> 2. `ycsb.bat`：看后缀就知道，这个是在 windows 上用的。
> 3. `ycsb.sh`：这是个 shell 脚本，不过需要系统中安装了 java。
>
> 本文使用的是第一个。



### 2.2. 运行一个工作负载



运行一个工作负载分为 6 步：

1. 设置要测试的数据库系统；
2. 选择合适的数据库接口层；
3. 选择合适的工作负载；
4. 选择合适的运行参数（如客户端线程数、目标吞吐量，等等）；
5. 加载数据；
6. 执行工作负载。



这里描述的步骤假设你正在运行一个客户端服务器。对于中小型集群（例如 10台 机器），这应该足够。对于更大的群集，你可能必须在不同的服务器上运行多个客户端才能生成足够的负载。同样，在某些情况下，使用多个客户端机器加载数据库可能会更快。有关并行运行多个客户端的更多详细信息，请参见 [Running a Workload in Parallel](https://github.com/brianfrankcooper/YCSB/wiki/Running-a-Workload-in-Parallel)。



#### 2.2.1. 步骤 1. 设置要测试的数据库系统



#### 2.2.2. 步骤 2. 选择合适的数据库接口层



#### 2.2.3. 步骤 3. 选择合适的工作负载



#### 2.2.4. 步骤 4. 选择合适的运行参数



#### 2.2.5. 步骤 5. 加载数据



#### 2.2.6. 步骤 6. 执行工作负载
