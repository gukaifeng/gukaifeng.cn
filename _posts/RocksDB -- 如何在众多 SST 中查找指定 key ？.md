---
title: RocksDB -- 如何在众多 SST 中查找指定 key ？
date: 2022-07-27 17:35:17
updated: 2022-07-27 17:35:17
categories: [数据库]
tags: [数据库,RocksDB]
---



## 1. 预备知识

1. 对于一个 Get() 请求，RocksDB 是按 memtable -> inmutable memtable -> SST 文件 的顺序来查找指定 key 的。

2. SST 文件以 level 的形式组织。
3. 在 L0，SST 文件的顺序即 flush 这些文件的时间顺序。
4. L0 的 SST 文件的 key 范围通常是重叠的（一个 SST 文件的 key 的范围的上、下限分别在 `FileMetaData.smallest`、`FileMetaData.largest` 中定义），所以我们查找一个 key 时，需要查找每个 L0 的 SST 文件。
5. Compaction 以从上层选择 SST 文件，并将其与下层的 SST 文件合并（这里认为层数越小，层级越高。例如，L0 是 L1 的上层）。从结果来看，L0 的 SST 文件会逐渐地向 LSM-Tree 下层移动。
6. Compaction 会排序 K/V 并把这些 K/V 划分到多个 SST 文件中。从 L1 开始到更下层，SST 文件依据 key 来排序，且各个 SST 文件的 key 范围是互斥的。





## 2. 如何查找

> 上面第 1 点，提到了 RocksDB 是按 memtable -> inmutable memtable -> SST 文件 的顺序来查找指定 key 的。
>
> 这篇文章主要关注最后的在 SST 文件中查找指定 key。



RocksDB 没有通过对每个 SST 文件进行扫描并检查 key 是否落入其范围，而是根据 `FileMetaData.largest` 进行二分查找，以找到可能包含目标 key 的候选文件。这将对每个 SST 文件查找的复杂度从 O(n) 降至了 O(logn)。但是这个复杂度对于底层的 levels 仍然非常大，例如，对于扇出比率 10，L3 可以有 1000 个文件，这需要 10 个比较才能找到候选文件。当你可以每秒有数百万 Get() 时，这对内存数据库来说开销是巨大的。



对此问题的一个观察结果是：LSM-Tree 构建后，一个 SST文件在其 level 中的位置就固定了。进一步说，其相对于下一个 level 的文件的顺序也是固定的。基于这个想法，我们可以执行**分散层叠([Fractional Cascading](https://en.wikipedia.org/wiki/Fractional_cascading))**的优化，以缩小二分查找的范围。

这是一个示例（若格式混乱请开启阅读模式，在右下角有选项）：

```
                                         file 1                                          file 2
                                      +----------+                                    +----------+
level 1:                              | 100, 200 |                                    | 300, 400 |
                                      +----------+                                    +----------+
           file 1     file 2      file 3      file 4       file 5       file 6       file 7       file 8
         +--------+ +--------+ +---------+ +----------+ +----------+ +----------+ +----------+ +----------+
level 2: | 40, 50 | | 60, 70 | | 95, 110 | | 150, 160 | | 210, 230 | | 290, 300 | | 310, 320 | | 410, 450 |
         +--------+ +--------+ +---------+ +----------+ +----------+ +----------+ +----------+ +----------+
```

示例中， L1 有 2 个文件，L2 有 8 个文件。下面根据这个示例，演示两个实际的查找流程。

1\. 查找 key = 80。

2\. 查找 key = 230。

