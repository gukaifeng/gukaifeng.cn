---
title: RocksDB -- 高级架构
date: 2021-06-30 17:31:34
updated: 2021-12-08 17:31:34
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---





前文说过，RocksDB 是一个有 kv 接口的存储引擎库，并且 kv 是任意大小的字节流。

我们看看 RocksDB 的官方架构图：

![](/posts/rocksdb-gao-ji-jia-gou/RocksDB--gao-ji-jia-gou_1.png)

<!--more-->



上面的图中涉及到了几个 RocksDB 特有的东西，例如 MemTable，SST file，WAL(Write Ahead Log) 和 MANIFEST，其中 MemTable，SST file 和 WAL 是 RocksDB 中最基础的三个结构。

-

RocksDB 的架构体系上图已经很清楚了，这里简单说下上面提到的几个概念：

memtable 是一个内存中的数据结构，RocksDB 先将数据库写操作写入 memtable，然后再 flush memtable 中的内容到 SST 文件中。

SST 文件是 RocksDB 存储数据的最终文件，即存在磁盘上的数据文件，默认的 SST 文件格式是 Block-based Table。

WAL(Write Ahead Log) 顾名思义，是一个写前日志。写操作除了会写入 memtable，还会写入 WAL。WAL 用于在数据库故障时恢复 memtable 中的数据。

MANIFEST 是 RocksDB 存储其状态改变的事务日志文件中，RocksDB 重启时，会通过 MANIFEST 中的内容，将 RocksDB 恢复到最后已知的一致性状态。

如果你对这些结构尚不了解，建议搭配下面几篇文章一同阅读本文，不然一些概念会比较模糊。

* [RocksDB -- MemTable 浅析](https://gukaifeng.cn/posts/rocksdb-memtable-qian-xi/)
* [RocksDB -- WAL(Write Ahead Log) 浅析](https://gukaifeng.cn/posts/rocksdb-wal-write-ahead-log-qian-xi/)
* [RocksDB -- MANIFEST 浅析](https://gukaifeng.cn/posts/rocksdb-manifest-qian-xi/)

至于 SST 文件，没什么好说的，就是存储最终数据的，你可以看看其默认的格式分析 [RocksDB -- Block-based Table 浅析](#)。

