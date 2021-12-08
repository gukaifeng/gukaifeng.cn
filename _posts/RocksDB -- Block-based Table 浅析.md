---
title: RocksDB -- Block-based Table 浅析
date: 2021-12-08 18:01:43
updated: 2021-12-08 18:01:43
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---



## 1. 什么是 Block-based Table



Block-basedTable 是 RocksDB 默认的 SST 表格式。

SST 是 RocksDB 架构的重要部分，关于 RocksDB 的架构，查看 [RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/)。

SST 文件就是数据库目录里那些以后缀 `.sst` 结尾的文件，例如 `000009.sst`，其中存储着 RocksDB 持久化的数据。

<!--more-->

## 2. Block-based Table 的格式

Block-based Table 的格式如下：

```
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]                  (see section: "filter" Meta Block)
[meta block 2: index block]
[meta block 3: compression dictionary block]  (see section: "compression dictionary" Meta Block)
[meta block 4: range deletion block]          (see section: "range deletion" Meta Block)
[meta block 5: stats block]                   (see section: "properties" Meta Block)
...
[meta block K: future extended block]  (we may add more meta blocks in the future)
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```

