---
title: RocksDB -- Leveled Compaction
date: 2022-05-10 15:16:40
updated: 2022-05-10 15:16:40
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>





## 1. 文件结构



磁盘上的文件以多个级别(levels)组织。我们称它们为 level-1，level-2 等，或者 L1，L2 等。特殊的 level-0（或简称 L0）包含刚从内存中写缓冲区（memtable）flush 的文件。每个级别（除 level-0）都是一个数据排序运行(sorted run)，如下：



![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_structure.png)



在每个级别（除级 level-0）内，数据范围分区为多个 SST 文件：



![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_files.png)

level 是排序运行，因为每个 SST 文件中的 key 都是排了序的（请参阅 [Block-based Table Format](https://github.com/facebook/rocksdb/wiki/Rocksdb-BlockBasedTable-Format) 作为示例）。为了确定一个 key 的位置，我们首先对所有文件的开始/结束 key 进行二分查找，以识别哪个文件可能包含该 key，然后在文件内进行二分查找以找到确切的位置。总而言之，它是对 level 中所有 key 的二分查找。



所有非 0 level 都有目标大小。Compaction 的目标是将这些 level 的数据大小限制在目标之下。目标大小通常呈指数增长：



![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_targets.png)





## 2. 压缩(Compactions)



![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l0_compaction.png)











![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l0_compaction.png)











![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l1_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l1_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l2_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l2_compaction.png)













![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/multi_thread_compaction.png)

















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/subcompaction.png)













![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/dynamic_level.png)











