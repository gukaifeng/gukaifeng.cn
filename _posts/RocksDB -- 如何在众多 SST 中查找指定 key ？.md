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

## 2. 如何查找 key 所在的 SST



> 上面第 1 点，提到了 RocksDB 是按 memtable -> inmutable memtable -> SST 文件 的顺序来查找指定 key 的。
> 
> 这篇文章主要关注最后的在 SST 文件中查找指定 key。

RocksDB 没有通过对每个 SST 文件进行扫描并检查 key 是否落入其范围，而是根据 `FileMetaData.largest` 进行二分查找，以找到可能包含目标 key 的候选文件。这将对每个 SST 文件查找的复杂度从 O(n) 降至了 O(logn)。但是这个复杂度对于底层的 levels 仍然非常大，例如，对于扇出比率 10，L3 可以有 1000 个文件，这需要 10 个比较才能找到候选文件。当你可以每秒有数百万 Get() 时，这对内存数据库来说开销是巨大的。

对此问题的一个观察结果是：LSM-Tree 构建后，一个 SST文件在其 level 中的位置就固定了。进一步说，其相对于下一个 level 的文件的顺序也是固定的。基于这个想法，我们可以执行 **分散层叠([Fractional Cascading](https://en.wikipedia.org/wiki/Fractional_cascading))** 的优化，以缩小二分查找的范围。

这是一个示例（若格式混乱可在右下角选项中开启阅读模式）：

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

1\. 查找 key = 80。基于 `FileMetaData.largest` 的二分查找会告诉我们 file 1 是候选文件。然后我们拿 80 和 file 1 的 `FileMetaData.smallest`（这里是 100） 和 `FileMetaData.largest`（这里是 200） 比较，得知 80 不在 file 1 中。然后我们继续看 level 2。通常来说，我们需要在 level 2 的所有 8 个文件上做二分查找，但由于我们已经知道 80 是小于 100（file 1 的 `FileMetaData.smallest`） 的，只有 file 1-3 满足，所以二分查找的文件数量就从 8 个减少到了 3 个。

2\. 查找 key = 230。首先在 level 1 进行二分查找，找到 file 2（这也意味着 key 230 大于 file 1 的 ` FileMetaData.largest` 200）。通过将 230 和 file 2 的 key 范围比较得知，230 小于 file 2 的  `FileMetaData.smallest`。即使我们没能在 level 1 找到目标 key 230，但我们也已经知道了这个 key 在 [200, 300] 这个区间（注意这里要从计算机的角度看待问题，即 file 1 的  `FileMetaData.largest` 200 和 file 2 的 `FileMetaData.smallest` 300）。Level 2 中任何与 [200, 300] 这个区间不重叠的都可以排除了，只剩下了 file 5-6 两个文件。

受上述概念影响，RocksDB 在 compaction 时为 level 1 的 SST 文件预构建了指向 level 2 的一个 SST 文件范围的指针。例如，在上面的示例中，为 level 1 的 file 1 构建指针，指向 level 2 的 file 3（范围左边界）和 file 4（范围右边界）；为 level 1 的 file 2 构建指针，指向 level 2 的 file 6 和 file 7。在查询时，这些指针用于根据比较结果来确定实际的二分查找范围。



## 3. 如何在一个 SST 内查找指定 key

上面介绍了如何找到可能存放目标 key 的 SST 文件，这一节说如果在这个 SST 内找到这个 key。（当然可能在这个 SST 中找不到这个 key，那就说明数据库中就是没有这个 key。）

### 3.1. 布隆过滤器(Bloom Filter)

这里不会介绍什么是布隆过滤器，可以参考 [Bloom filter - Wikipedia](https://en.wikipedia.org/wiki/Bloom_filter)。

即便你不了解布隆过滤器，也能够理解下面的内容。

\-

简单来说，布隆过滤器可以在 **O(1)** 的时间内，判断一个 key 是否在一个数据集中，当这个数据集十分庞大时，布隆过滤器能带来很大的性能优势，减少很多不必要的读。

布隆过滤器的判断只有两种结果：

1. 目标 key **可能**存在；

2. 目标 key **一定**不存在。



若启用了过滤策略，每个新创建的 SST 文件都会包含一个布隆过滤器。

当 RocksDB 找到可能存放目标 key 的 SST 文件后，可以使用布隆过滤器，快速判断这个 SST 文件中是否存在目标 key，如果**一定不存在**，就不再继续查找，如果**可能存在**，就继续查找。这样就排除了一大类查询一定不存在 key 的请求，节省了大量的资源。



### 3.2. SST 文件内查找



RocksDB 支持多种 SST 文件格式，最主要的是 Block-Based Table 格式（也是默认的 SST 文件格式），这里以此为例。

Block-Based Table 的格式如下：

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

我们这里暂时只需要关注数据块(data block) 和索引快(meta block 2: index block)。

在没有启用索引的时候，RocksDB 才用二分查找对 data block 块进行搜索。

在启用了索引的情况下，RocksDB 通过索引快（例如哈希索引）来确定一个 key 具体在哪个位置。




