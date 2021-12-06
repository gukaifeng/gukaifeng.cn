---
title: RocksDB -- MemTable 浅析
date: 2021-12-06 17:57:02
updated: 2021-12-06 17:57:02
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



## 1. 什么是 MemTable



MemTable 是一个**内存**中的数据结构，在数据被 flush 到 SST 文件之前，它保存着数据。

memtable 同时服务于读和写 —— 新的写总是插入数据到 memtable，而读必须在从 SST 文件读取之前查询memtable，因为 memtable 中的数据是更新的。

一旦一个 memtable 被填满，它就成为不可变的，并被一个新的 memtable 所取代。

一个后台线程将**不可变的 memtable** 的内容 flush 到一个 SST 文件中，然后可以销毁这个 memtable。



memtable是 RocksDB 架构中非常重要的一部分。  
关于 RocksDB 的架构，查看这里 [RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/)。



## 2. 几个对 MemTable 影响较大的选项



1\. `AdvancedColumnFamilyOptions::memtable_factory`: memtable 的工厂对象。通过指定工厂对象，用户可以改变 memtable 的底层实现，并提供实现特定的选项（默认工厂: SkipListFactory）。

2\. `ColumnFamilyOptions::write_buffer_size`: 单个 memtable 的大小（默认为 64MB）。

3\. `DBOptions::db_write_buffer_size`: 跨列族的 memtable 的总大小最大值。这可以用来管理 memtable 使用的总内存。(默认值: 0(禁用))

4\. `DBOptions::write_buffer_manager`: 用户可以提供自己的写缓冲区管理器来控制整个 memtable 内存的使用，而不是指定 memtable 的总大小。覆盖 `db_write_buffer_size`。(默认值:nullptr)

5\. `AdvancedColumnFamilyOptions::max_write_buffer_number`: 在内存中建立的 memtable 的最大数量，在它们刷新到 SST 文件之前。(默认值:2)

6\. `AdvancedColumnFamilyOptions::max_write_buffer_size_to_maintain`: 要在内存中维护的写历史记录的最大大小，以字节为单位。这包括当前的 memtable 大小，密封（指的是写满后变得不可变的 memtable）但未 flush 的 memtable，以及已经 flush 完成但还未删除的 memtable。RocksDB 会尝试在内存中保留至少这么多的历史记录 —— 如果删除一个已经 flush 完成的 memtable 会导致历史记录大小低于这个阈值，那么这个 memtable 就不会被删除，即便它已经没用了。(默认值:0，0 表示不设定这个阈值，一个 memtable 被 flush 完成后会立刻被删除)





## 3. 何时进行 flush

有三种场景可以触发 memtable flush:



1. 在一次写操作后，memtable 大小超过了 `ColumnFamilyOptions::write_buffer_size`。

2. 所有列族的 memtable 总大小超过 `DBOptions::db_write_buffer_size`，或者 `DBOptions::write_buffer_manager` 中设定的大小时会进行 flush。在这种情况下，最大的 memtable 将被 flush。
3. 总 WAL 文件大小超过 `DBOptions::max_total_wal_size`。在这个场景中，包含最旧数据的 memtable 将被flush，以便清除包含该 memtable 数据的 WAL 文件。

因此，可以在 memtable 满之前刷新它。这就是生成的 SST 文件比相应的 memtable 要小的原因之一。

压缩是使 SST 文件小于 memtable 的另一个因素，因为 memtable 中的数据是未压缩的。



## 4. MemTable 的实现概述

memtable 的默认实现是基于跳表(SkipList)的。

除了默认的 memtable 实现（SkipList），用户也可以使用其他类型的 memtable 实现，例如 HashLinkList、HashSkipList 或 Vector，以加快一些查询。



### 4.1. 两个仅 SkipList MemTable 支持的特性



#### 4.1.1. 并发插入



如果不支持对 memtable 的并发插入，多个线程对 RocksDB 的并发写操作将依次应用于 memtable。

并发 memtable 插入在默认情况下是启用的，可以通过 `DBOptions::allow_concurrent_memtable_write` 选项关闭，尽管只有基于 skiplist 的 memtable 支持该特性。

> 这里额外说一下另一个特性：就地更新
>
> 可以通过切换 `bool inplace_update_support` 标志来启用就地更新。
>
> 然而，该标志默认设置为 false，因为这种线程安全的就地更新支持与并发的 memtable 写入不兼容。注意，`bool allow_concurrent_memtable_write` 默认设置为 true。



#### 4.1.2. 带提示的插入



这个地方官方 wiki 也只有个标题（似乎是忘了更），我也先只写个标题吧。。。





### 4.2. 几种 memtable 实现的比较

| MemTable 实现                | SkipList                                 | HashSkipList                                                 | HashLinkList                                           | Vector                                                 |
| ---------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| 优化场景                     | 通用                                     | 特定 key 前缀的范围查找                                      | 特定 key 前缀的范围查找，每个前缀只有少量行            | 随机写入（繁重的工作量）                               |
| 索引类型                     | 二分查找                                 | 哈希 + 二分查找                                              | 哈希 + 线性搜索                                        | 线性搜索                                               |
| 是否支持完全有序的全 db 扫描 | 原生支持                                 | 支持但代价极高（复制和排序以创建临时的完全有序的视图）       | 支持但代价极高（复制和排序以创建临时的完全有序的视图） | 支持但代价极高（复制和排序以创建临时的完全有序的视图） |
| 内存开销                     | 中等（大约每个条目 1.33 个指针）         | 高（哈希 bucket + 非空 bucket 跳过列表元数据 + 每个条目的多个指针） | 较低（每个条目的哈希 bucket + 指针）                   | 低（Vector 末尾的预分配空间）                          |
| MemTable Flush               | 快速，仅使用常量大小的额外内存           | 速度慢，临时内存占用率高                                     | 速度慢，临时内存占用率高                               | 速度慢，仅使用常量大小的额外内存                       |
| 并发插入                     | 支持                                     | 不支持                                                       | 不支持                                                 | 不支持                                                 |
| 带提示的插入                 | 支持（仅在没有启动并发插入的场景下支持） | 不支持                                                       | 不支持                                                 | 不支持                                                 |



