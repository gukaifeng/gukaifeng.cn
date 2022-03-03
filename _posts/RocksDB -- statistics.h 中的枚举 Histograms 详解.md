---
title: RocksDB -- statistics.h 中的枚举 Histograms 详解
date: 2022-03-02 16:45:47
updated: 2022-03-02 16:45:47
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



RocksDB 的 Histograms 枚举定义在 `statistics.h` 中，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L423-#L520)。

`statistics.h` 中还声明了一个 Histograms 枚举值与 rocksdb 属性的一一对应关系，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L522)，而具体的定义在 `statistics.cc` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/monitoring/statistics.cc#L219-L274)。

下面按照 `statistics.h` 中 Histograms 枚举的属性定义顺序，来逐个介绍。







#### 1. DB_GET

`Histograms::DB_GET` 对应的 rocksdb 属性是 `rocksdb.db.get.micros`。



#### 2. DB_WRITE

`Histograms::DB_WRITE` 对应的 rocksdb 属性是 `rocksdb.db.write.micros`。

