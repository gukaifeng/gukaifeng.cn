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









![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_structure.png)









![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_files.png)









![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/level_targets.png)









![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l0_compaction.png)











![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l0_compaction.png)











![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l1_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l1_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/pre_l2_compaction.png)















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/post_l2_compaction.png)













![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/multi_thread_compaction.png)

















![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/subcompaction.png)













![](https://gukaifeng.cn/posts/rocksdb-leveled-compaction/dynamic_level.png)











