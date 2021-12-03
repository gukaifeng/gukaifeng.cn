---
title: RocksDB -- 高级架构
date: 2021-06-30
updated: 2021-06-30
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---





前文说过，RocksDB 是一个有 kv 接口的存储引擎库，并且 kv 是任意大小的字节流。  
RocksDB 把所有的数据有序组织起来，并且提供了一套通用的操作：`Get(key)`、`NewIterator()`、`Put(key, val)`、`Delete(key)` 和 `SingleDelete(key)`。

RocksDB 中最基础的三个结构是 **[memtable](https://github.com/facebook/rocksdb/wiki/MemTable)**、**[sstfile](https://github.com/facebook/rocksdb/wiki/Rocksdb-BlockBasedTable-Format)** 和 **[logfile (aka. Write Ahead Log(WAL))](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log)**。

memtable 是一个内存数据结构。

新的写操作会被插入到 memtable 中，也可以选择是否同时插入到 logfile。logfile 是在存储介质上一个连续写入的文件。

当 memtable 被写满了以后，RocksDB 就会把其变成一个不可变的 memtable，不可变的 memtable 将等待 flush。

flush 操作将把不可变的 memtable flush 到一个存储介质上的 sstfile 中，之后与之对应的 logfile 就可以安全的删除了。

为了加速 key 的查找效率，sstfile 中的数据是被排序了的。

<!--more-->

默认的 sstfile 格式是 BlockBasedTable，[点此了解更多](https://gukaifeng.me/2021/05/19/RocksDB-BlockBasedTable-%E5%88%86%E6%9E%90/)。

![](/posts/rocksdb-gao-ji-jia-gou/RocksDB--gao-ji-jia-gou_1.png)


## 参考资料

https://github.com/facebook/rocksdb/wiki/RocksDB-Overview
