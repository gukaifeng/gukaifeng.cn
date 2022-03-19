---
title: RocksDB -- DB::Properties 中的属性详解
date: 2022-03-02 16:42:26
updated: 2022-03-02 16:42:26
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



RocksDB 中的 DB 类中定义了一个结构体 Properties，里面是许多 RocksDB 的属性。

具体代码在 `db.h` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L748-L987)查看。下面按照代码中的定义顺序，来逐一介绍每个属性。

