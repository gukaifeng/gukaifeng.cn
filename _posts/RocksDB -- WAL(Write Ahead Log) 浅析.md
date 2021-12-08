---
title: RocksDB -- WAL(Write Ahead Log) 浅析
date: 2021-12-08 13:22:45
updated: 2021-12-08 13:22:45
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



## 1. 什么是 WAL



每次对 RocksDB 的更新都被写入两个地方：

1. 内存中一个名为 memtable 的数据结构（稍后将被 flush 到 SST 文件中）；
2. WAL。



当发生故障时，可以使用 WAL 来完全恢复 memtable 中的数据，这对于将数据库恢复到原始状态是必要的。

在默认配置中，RocksDB 通过在每个用户写入后 flush WAL 来保证进程崩溃的一致性。

WAL(Write Ahead Log) 是 RocksDB 架构中重要的一部分，如果你还不了解 RocksDB 的架构，可以查看 [RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/) 这篇文章。



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

这篇文章主要讲 WAL 文件是什么，关于具体的记录格式，查看 [Write Ahead Log File Format](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log-File-Format)。

