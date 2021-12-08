---
title: RocksDB -- WAL(Write Ahead Log) 浅析
date: 2021-12-08 13:22:45
updated: 2021-12-08 13:22:45
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



## 1. 什么是 WAL

WAL(Write Ahead Log) 顾名思义，写前日志。

每次对 RocksDB 的更新都被写入两个地方：

1. 内存中一个名为 memtable 的数据结构（稍后将被 flush 到 SST 文件中）；
2. WAL。



当发生故障时，可以使用 WAL 来完全恢复 memtable 中的数据，这对于将数据库恢复到原始状态是必要的。

在默认配置中，RocksDB 通过在每个用户写入后 flush WAL 来保证进程崩溃的一致性。

当 RocksDB 干净地关闭时，所有未提交的数据都在关闭之前提交，因此始终保证一致性。当 RocksDB 进程被杀死或机器重新启动时，RocksDB 需要将自己恢复到一致的状态。

<!--more-->

WAL 是 RocksDB 架构中重要的一部分，如果你还不了解 RocksDB 的架构，可以查看 [RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/) 这篇文章。



## 2. WAL 的生命周期

让我们使用一个示例来说明 WAL 的生命周期。

-

```cpp
DB *db;
std::vector<ColumnFamilyDescriptor> column_families;
column_families.push_back(ColumnFamilyDescriptor(
    kDefaultColumnFamilyName, ColumnFamilyOptions()));
column_families.push_back(ColumnFamilyDescriptor(
    "new_cf", ColumnFamilyOptions()));
std::vector<ColumnFamilyHandle *> handles;
s = DB::Open(DBOptions(), kDBPath, column_families, &handles, &db);
```

一个 RocksDB 实例数据库有两个列族 "new_cf" 和 "default"。

一旦打开 db，将在磁盘上创建一个新的 WAL 来持久化所有的写操作（WAL 在所有列族之间共享）。

-

```cpp
db->Put(WriteOptions(), handles[1], Slice("key1"), Slice("value1"));
db->Put(WriteOptions(), handles[0], Slice("key2"), Slice("value2"));
db->Put(WriteOptions(), handles[1], Slice("key3"), Slice("value3"));
db->Put(WriteOptions(), handles[0], Slice("key4"), Slice("value4"));
```

此时，WAL 应该已经记录了所有写入。

WAL 将保持打开状态，并继续记录未来的写操作，直到它的大小达到 `DBOptions::max_total_wal_size`。

如果用户决定 flush 列族 "new_cf"，将会发生下面几件事

1. new_cf (key1 和 key3) 的数据 flush 到一个新的 SST 文件；
2. 创建一个新的 WAL，未来所有写入所有列族的操作都将被记录在新的 WAL 中；
3. 旧的 WAL 不再接受写入新的记录，但是删除可能会延迟。



-

```cpp
db->Flush(FlushOptions(), handles[1]);  // 只 flush 了 "new_cf" 列族
// key5 and key6 will appear in a new WAL
db->Put(WriteOptions(), handles[1], Slice("key5"), Slice("value5"));
db->Put(WriteOptions(), handles[0], Slice("key6"), Slice("value6"));
```



此时将有两个 WAL，旧的 WAL 包含 key1 到 key4，新的 WAL 包含 key5 和 key6。

因为旧的 WAL 仍然包含至少一个列族("default")的实时数据，所以还不能删除它。

只有当用户最终决定 flush "default" 列族时，旧的 WAL 才能被存档并自动从磁盘中删除。

-

```cpp
db->Flush(FlushOptions(), handles[0]);
// 旧的 WAL 将分别存档，然后清除
```

-

总的来说，当下面两件事发生时，会创建一个 WAL。

1. 打开一个新的 DB；
2. flush 一个列族时。



当所有列族的 flush 超过了 WAL 中包含的最大序列号，或者换句话说，WAL 中的所有数据都持久化到 SST 文件时，将删除 WAL（如果启用了存档，则将存档）。随后存档的 WAL 将被移动到一个单独的位置，稍后将从磁盘中清除。

出于复制的目的，实际的删除可能会延迟，请参阅事务日志迭代器(Transaction Log Iterator)。

> 事务日志迭代器(Transaction Log Iterator)提供了一种在 RocksDB 实例之间复制数据的方法。
>
> 若由于列族 flush 而对 WAL 进行了存档，被存档的 WAL 不会被立即删除。目标是允许事务日志迭代器继续读取 WAL，并将其发送给跟随者进行重放。

## 3. WAL 配置

下面是几个与 WAL 有关选项，都可以在 `option.h` 头文件中找到。

1. `DBOptions::wal_dir`: 设置 RocksDB 存储 WAL 文件的目录，这允许将 WAL 存储在与实际数据相独立的目录中。
2. `DBOptions::WAL_ttl_seconds, DBOptions::WAL_size_limit_MB`: 这两个字段会影响已经存档的 WAL 被删除的速度。非零值表示触发存档 WAL 删除的时间和磁盘空间阈值。
3. `DBOptions::max_total_wal_size`: 为了限制 WALs 的大小，RocksDB 使用 `DBOptions::max_total_wal_size` 作为列族 flush 的触发器。一旦 WALs 超过这个大小，RocksDB 将开始强制 flush 列族，以允许删除一些最老的 WALs。当列族以非均匀频率更新时，此配置非常有用。如果没有大小限制，当不经常更新的列族有一段时间没有 flush 时，用户可能需要保留真正的旧 walls。
4. `DBOptions::avoid_flush_during_recovery`: 这个配置的作用从名字就能看出来，避免在恢复数据库时 flush，bool 类型，默认为 false。
5. `DBOptions::manual_wal_flush`: 这个选项决定 WAL 的 flush 是在每次写操作后自动执行还是纯手动执行（用户必须调用 `FlushWAL()` 方法来触发 WAL 刷新）。
6. `DBOptions::wal_filter`: 通过 `DBOptions::wal_filter`，用户可以提供一个过滤器对象，以便想在恢复数据库期间处理 WALs 时调用。注意：在 ROCKSDB_LITE 模式下不支持。
7. `WriteOptions::disableWAL`: 禁用 WAL。当用户依赖于其他日志记录或不关心数据丢失时，很有用。



## 4. WAL 文件管理

WAL 将 memtable 操作序列化为日志文件。在发生故障时，通过日志重构 memtable，可以使用 WAL 文件将数据库恢复到其一致状态。当 memtable 被安全地 flush 到持久存储介质时，相应的 WAL 日志将被废弃并存档。最终，存档的 WAL 日志会在一段时间后从磁盘中清除。

WAL 文件是在 WAL 目录下按序号递增生成的。为了重建数据库的状态，按照序列号的顺序读取这些文件。WAL 管理器(WAL Manager)提供了将 WAL 文件作为单个单元读取的抽象。在内部，它使用 Reader 或 Writer 抽象打开和读取文件。

Writer 提供了将日志记录追加到日志文件的抽象。特定存储介质写入的内部实现细节由 `WriteableFile()` 接口处理。

类似地，Reader 提供了从日志文件中连续读取日志记录的抽象。特定存储介质读取的内部实现细节由 `SequentialFile()` 接口处理。



## 5. WAL 文件格式

这篇文章主要讲 WAL 文件是什么，关于具体的记录格式分析，查看 [RocksDB -- WAL 记录格式](https://gukaifeng.cn/posts/rocksdb-wal-ji-lu-ge-shi/) 这篇文章。



## 6. WAL 的恢复模式



一个重要的恢复操作是在 WAL 中**重放(replay)**未提交的记录。不同的 WAL 恢复模式定义了 WAL 重放的行为。

WAL 恢复模式共计有 4 种，其定义在 `option.h` 头文件中，[点此查看](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/options.h#L323-L360)。

下面分别简单介绍一下 4 中恢复模式。



1. `kTolerateCorruptedTailRecords`: 在这个模式下，WAL 重放将忽略在日志尾部发现的任何错误。其原因是，在不干净的关闭中，日志尾部可能会有不完整的写操作。这是一种启发式模式，系统无法区分日志尾部的损坏和未完成的写入。任何其他 IO 错误，将被认为是数据损坏。这种模式对大多数应用程序来说都是可以接受的，因为它提供了在非正常关闭后启动 RocksDB 和一致性之间的合理权衡。
2. `kAbsoluteConsistency`: 在这个模式下，任何在 WAL 重放过程中的 IO 错误都被认为是数据损坏。这种模式非常适合那些不能丢失甚至单个记录的应用程序，或有其他方法恢复未提交数据的应用程序。
3. `kPointInTimeRecovery`: 在这个模式下，WAL 重放在遇到 IO 错误后停止。系统恢复到与当前时间一致的时间点。这对于具有副本的系统非常理想。来自另一个副本的数据可以用于重放系统恢复到的“时间点”。(这是 RocksDB v6.6 版本以后的默认值。注：本站关于 RocksDB 的文章，无特别说明，均依据 v6.25.3。)
4. `kSkipAnyCorruptedRecords`: 在这个模式下，读取日志时遇到的任何 IO 错误都将被忽略，系统将试图恢复尽可能多的数据，这是灾难恢复的理想选择。





## 7. WAL 性能表现

### 7.1. non-sync 模式

当 `WriteOptions.sync = false` (默认值)，表示不将 WAL 写入同步到磁盘。除非操作系统认为它必须 flush 数据(例如，太多脏页面)，用户不需要等待任何 I/O 来写。

如果用户甚至想要减少 CPU 由于写入操作系统页面缓存而带来的延迟，可以选择 `Options.manual_wal_flush = true`。有了这个选项，WAL 写操作甚至不会 flush 到文件系统页面缓存中，而是保存在 RocksDB 中。用户需要调用 `DB::FlushWAL()` 来让缓冲条目进入文件系统。

用户可以通过调用 `DB::SyncWAL()` 强制 fsync WAL 文件。该函数不会阻塞在其他线程中执行的写操作。

在这种模式下，WAL 写入不是崩溃安全的。



### 7.2. sync 模式

当 `WriteOptions.sync = true`，在返回给用户之前对 WAL 文件进行 fsync。



### 7.3. 组提交(Group Commit)

和大多数依赖日志的程序一样，RocksDB 也支持组提交来提高 WAL 写吞吐量和写放大。

RocksDB 的组提交是用一种简单的方式实现的：当不同的线程同时写同一个数据库时，所有有资格合并的未完成的写操作将被合并到一起，通过一个 fsync 向 WAL 写入一次。这样，同样数量的 I/O 可以完成更多的写操作。



具有不同写选项的写操作可能会取消组合的资格。

最大的组大小是 1MB。RocksDB 不会尝试通过主动延迟写操作来增加组大小。





### 7.4. 每次写的 I/O 次数

如果 `Options.recycle_log_file_num = false` (默认值)。RocksDB 总是为新的 WAL 段创建新文件。

每次 WAL 写都会改变数据和文件的大小，所以每次 fsync 都会生成至少两个 I/O，一个用于数据，一个用于元数据。

注意，RocksDB 调用 `fallocate()` 来为文件预留足够的空间，但它并不阻止 fsync 中的元数据 I/O。



 `Options.recycle_log_file_num = true` 将保留一个 WAL 文件池并尝试重用它们。当写入现有日志文件时，从大小为 0 开始使用随机写入。在写入到达文件末尾之前，文件大小不会改变，因此可能会避免元数据的 I/O (也取决于文件系统挂载选项)。假设大多数 WAL 文件具有相似的大小，那么元数据所需的 I/O 将是最小的。



### 7.5. 写放大

请注意，对于某些用例，同步的 WAL 可能引入非常大的的写放大。

当写操作很小时（因为可能需要更新完整的块/页），即使写操作非常小，我们也可能会有两次 4KB 的写操作（一次用于数据，一次用于元数据）。

如果 write 仅为 40 字节，则更新 8KB，则写入放大值为 8KB/40B~= 200。它可以很容易地比 LSM-tree 的写放大功能还要大。
