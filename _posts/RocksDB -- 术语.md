---
title: RocksDB -- 术语
date: 2021-06-30
updated: 2021-06-30
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---





本篇博客解释了 RocksDB 的相关术语。  
同时，考虑到术语的特殊性，部分名词仅做解释，不做翻译。

<!--more-->

##### 1. 2PC (Two-phase commit)

表示分为两个阶段提交。悲观事务可以在两个阶段提交：首先是准备，然后是实际的提交。 See https://github.com/facebook/rocksdb/wiki/Two-Phase-Commit-Implementation

##### 2. Backup
备份。RocksDB 有一个备份工具，可以帮助用户将数据库状态备份到不同的位置，比如 HDFS。 See https://github.com/facebook/rocksdb/wiki/How-to-backup-RocksDB

##### 3. Block cache
缓存来自 SST 文件的热数据块的内存数据结构。 See https://github.com/facebook/rocksdb/wiki/Block-Cache

##### 4. BlockSST
文件的数据块。在 block-based table 的 SST 文件中，一个 block 总是被校验和检查，并且通常会被压缩存储。

##### 5. Block-based bloom filter <font color=#D3D3D3>or</font> full bloom filter
在 SST 文件中存储 bloom 过滤器的两种不同方法，这两种方法都是 block-based table 格式的特性。 See https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter#new-bloom-filter-format

##### 6. Block-Based Table
RocksDB 中默认的 SST 文件格式。 See https://github.com/facebook/rocksdb/wiki/Rocksdb-BlockBasedTable-Format

##### 7. Bloom filter
Bloom 过滤器。See https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter

##### 8. Checkpoint
检查点。一个检查点就是某个数据库的一份物理镜像，储存在文件系统中的另一个位置。 See https://github.com/facebook/rocksdb/wiki/Checkpoints

##### 9. Column Family
常译为列族/列簇。在一个 RocksDB 的数据库中，column family 是一个单独的键空间。但这个名字很有误导性，因为 RocksDB 中的 column family 和其他存储系统中的 column family 概念没有任何相似性，RocksDB 中甚至没有“列”的概念（所以这个名字起的就很迷）。 See https://github.com/facebook/rocksdb/wiki/Column-Families

##### 10. Compaction filter
压缩过滤器。一个用户插件，可以在压缩的时候修改或删除已经存在的 key。 See https://github.com/facebook/rocksdb/blob/master/include/rocksdb/compaction_filter.h

##### 11. Compaction
压缩，在后台运行。Compaction 会合并一些 SST 文件到另一些 SST 文件中。在 LevelDB 中，compaction 还包括了 flush。RocksDB 进一步区分开了 compaction 和 flush 这两个概念。  See https://github.com/facebook/rocksdb/wiki/RocksDB-Basics#multi-threaded-compactions and https://github.com/facebook/rocksdb/wiki/Compaction

##### 12. Comparator
一个可以定义键顺序的插件类。 See https://github.com/facebook/rocksdb/blob/master/include/rocksdb/comparator.h

##### 13. DB properties
一些数据库的运行状态信息，这些信息可以通过 DB::GetProperty() 获得。 See https://github.com/facebook/rocksdb/blob/master/include/rocksdb/db.h

##### 14. Flush
在后台运行，把 memtables 中的数据写入到 SST 文件中。

##### 15. Forward iterator / Tailing iterator
特殊的迭代器选项，可以针对特定用例优化。 See https://github.com/facebook/rocksdb/wiki/Tailing-Iterator

##### 16. Immutable memtable
一个已经关闭了的，并且正在等待 flush 的 memtable。

##### 17. Index
SST 文件中数据块的索引，在 SST 文件中始终有一个索引块来存放这些索引。默认的索引格式是二分查找索引。

##### 18. Iterator
迭代器。迭代器可以让用户在一个范围内按顺序查询 key。 See https://github.com/facebook/rocksdb/wiki/Basic-Operations#iteration

##### 19. Leveled Compaction or Level-Based Compaction Style
RocksDB 默认的压缩方式，即 Leveled Compaction。

##### 20. LSM level
一个数据库物理数据的逻辑组织，用以维护所需的 LSM-tree 的形态和结构。 See https://github.com/facebook/rocksdb/wiki/Compaction, particularly https://github.com/facebook/rocksdb/wiki/Compaction#lsm-terminology-and-metaphors

##### 21. LSM-tree
一种数据结构，即 [Log-structured merge-tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree)。RocksDB 是一个基于 LSM-tree 的存储引擎。

##### 22. Memtable switch
Memtable 开关。在这个过程中，当前活跃的 memtable (当前的写操作) 被关闭，变成 immutable memtable。同时，RocksDB 会关闭当前的 WAL 文件，再启动一个新的 WAL 文件。

##### 23. Memtable / write buffer
一个内存数据结构，用于存储大部分最近对数据库的改动。这些改动的存储一般是有序的，同时还包括了可以二分查找的索引。 See https://github.com/facebook/rocksdb/wiki/Basic-Operations#memtable-and-table-factories

>##### 24. Merge operator
>RocksDB supports a special operator Merge(), which is a delta record, merge operand, to the existing value. Merge operator is a user defined call-back class which can merge the merge operands. See https://github.com/facebook/rocksdb/wiki/Merge-Operator-Implementation 

>##### 25. Partitioned Filters
>Partitioning a full bloom filter into multiple smaller blocks. See https://github.com/facebook/rocksdb/wiki/Partitioned-Index-Filters.

>##### 26. Partitioned Index
>The binary search index block partitioned to multiple smaller blocks. See https://github.com/facebook/rocksdb/wiki/Partitioned-Index-Filters

##### 27. perf context
用于测量线程本地统计数据的内存数据结构，常用于测量每个查询的统计数据。 See https://github.com/facebook/rocksdb/wiki/Perf-Context-and-IO-Stats-Context

##### 28. Pessimistic Transactions
悲观事务。使用锁隔离多个并发事务，默认写策略为“WriteCommitted”。

##### 29. PlainTable
SST 文件另一个可选的格式，这个格式针对 ramfs 进行了优化。 See https://github.com/facebook/rocksdb/wiki/PlainTable-Format

##### 30. Point lookup
点查询。在 RocksDB 中，Point lookup 指的是通过 Get() 或着 MultiGet() 读取一个 key。

>##### 31. Prefix bloom filter
>a special bloom filter that can be limitly used in iterators. Some file reads are avoided if an SST file or memtable doesn't contain the prefix of the lookup key extracted by the prefix extractor. See https://github.com/facebook/rocksdb/wiki/Prefix-Seek-API-Changes

##### 32. Prefix extractor
一个可以提取 key 前缀部分的回调类，最常用作 prefix bloom filter 中的前缀。 See https://github.com/facebook/rocksdb/blob/master/include/rocksdb/slice_transform.h

##### 33. Range lookup
范围查找，使用迭代器读取一组键。

##### 34. Rate limiter
限速器。通过 flush 和 compaction 操作来限制写入文件系统的字节速率。 See https://github.com/facebook/rocksdb/wiki/Rate-Limiter

##### 35. Recovery
恢复。指的是在数据库出错或着被关闭后，重启启动的过程。

##### 36. Sequence number (SeqNum / Seqno)
序号。每个写数据库的操作都会被分配一个 ID number 序号，这些序号自动递增。在 WAL 文件、memtable 和 SST 文件中，序号附加在键值对上。序号可用于实现压缩过程中的快照读取和垃圾回收，在事务中的 MVCC，以及其他一些目的。

##### 37. Single delete
一个特殊的删除操作，只在用户从不更新一个现存 key 时有效。 See: https://github.com/facebook/rocksdb/wiki/Single-Delete

##### 38. Snapshot
快照。快照是运行中数据库的逻辑一致的时间点视图。 See https://github.com/facebook/rocksdb/wiki/RocksDB-Basics#gets-iterators-and-snapshots

##### 39. SST File (Data file / SST table)
SST 意思是 Sorted Sequence Table，排序序列表。SST 是存储数据的持久化的文件，在 SST 文件中，keys 一般是有序组织起来的。所以一个 key 或者迭代位置可以通过二分查找确定。

##### 40. Statistics
内存数据结构，包含了活动数据库的累积统计信息。 https://github.com/facebook/rocksdb/wiki/Statistics

##### 41. Super Version
超级版本，RocksDB 内部概念。一个 super version 包含了 SST 和 blob 的文件列表 (一个版本)，以及在某个时间点的活跃的 memtables 列表。Flush、compaction 和 memtable switch 操作都会导致一个新的 super version 被创建，正在进行的读请求可以继续使用旧的 super version。当旧的 super version 不再需要时，会被垃圾回收掉。

##### 42. Table Properties
表属性。元数据存储在每个 SST 文件 (统计块) 中，其中包括由 RocksDB 生成的系统属性和由用户定义的回调计算的用户定义表属性。 See https://github.com/facebook/rocksdb/blob/master/include/rocksdb/table_properties.h

##### 43. Universal Compaction Style
另一种压缩算法。 See https://github.com/facebook/rocksdb/wiki/Universal-Compaction

##### 44. Version
RocksDB 的内部概念。一个 version 包含了在某个时间点上全部活跃的 SST 和 blob (如果使用了 BlobDB 的话) 文件。当 flush 或 compaction 操作完成时，一个新的 version 就会被创建，因位活跃 SST/blob 文件列表被修改了。一个旧的 version 可以继续被已经进行中的读请求或压缩操作使用。旧的 version 最终会被垃圾回收。

>##### 45. Write stall
>When flush or compaction is backlogged, RocksDB may actively slowdown writes to make sure flush and compaction can catch up. See https://github.com/facebook/rocksdb/wiki/Write-Stalls


##### 46. Write-Ahead-Log (WAL) or log
一个日志文件，用来在恢复数据库时，恢复那些还没 flush 到 SST 文件中的数据。 See https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log-File-Format

##### 47. WriteCommitted
悲观事务的默认写策略。先把写操作缓冲到内存，再在事务提交的时候写入数据库。 

>##### 48. WritePrepared
>A write policy in pessimistic transactions that buffers the writes in memory and write them into the DB upon prepare if it is a 2PC transaction or commit otherwise. See https://github.com/facebook/rocksdb/wiki/WritePrepared-Transactions

##### 49. WriteUnprepared
悲观事务的一个写策略。在事务发送数据时将数据写入数据库，以避免需要更大的内存缓冲区。 See https://github.com/facebook/rocksdb/wiki/WritePrepared-Transactions





## 50. 参考资料

https://github.com/facebook/rocksdb/wiki/Terminology