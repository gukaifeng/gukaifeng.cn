---
title: RocksDB -- 特性
date: 2021-06-30
updated: 2021-06-30
categories: [技术杂谈]
tags: [数据库, RocksDB]
toc: true
---





### 1. Column Families

RocksDB 支持把一个数据库实例划分成多个 column families。当创建一个数据库时，如果没有指定使用那个 column family，RocksDB 就会用一个名为 "default" 的默认 column family 创建此数据库。需要注意的是，**RocksDB 中的 column family 与其他地方的此概念不同，这里指的是一个独立的键空间，**这很有误导性，这名字用的就很迷。

RocksDB 为用户在跨 column families 中保证了一致的视图，在启用 WAL 或者启用原子 flush 前提下的崩溃恢复以后也可以。RocksDB 还支持通过 `WriteBatch` API 实现跨 column families 的原子级操作。

### 2. Updates

`Put` API 往数据库中插入单个 kv 对，如果在数据库中已经存在了 key，那之前的 value 就会被覆盖掉。  
`Write` API 可以自动地插入、更新或删除多个 kv 对。RocksDB 会保证一个` Wirte`，要么完成全部的操作，要么什么都不做。如果` Wirte` 执行插入操作的时候，有 key 已经存在的话，那这个 key 之前的 value 就会被覆盖。  
[`DeleteRange`](https://github.com/facebook/rocksdb/wiki/DeleteRange)  API 可以用来删除一个范围的 keys。

<!--more-->

### 3. Gets, Iterators and Snapshots

在 RocksDB 中，key 和 value 都被视为纯字节流，并且没有限制长度。  
一个应用可以用 `Get` API 从 RocksDB 中获取单个 key/value，可以用 `MultiGet` API  获取一堆。通过 `MulitGet` 调用返回的所有 kv 彼此一致。

RocksDB 中的所有数据在逻辑上都是按一定顺序组织的。应用程序可以指定一个 key 间的比较规则，使得数据库中的数据按其理想的顺序组织。  
`Iterator` API 可以让应用程序对数据库进行一个范围扫描。`Iterator` 可以找到指定的 key，然后应用程序就能从这个 key 的位置接着扫描下一个 key。`Iterator` 也可以用来反向扫描数据库中的键。当创建一个  `Iterator` 时，一个时间一致性视图(consistent-point-in-time view)就被创建了，因此所有通过 `Iterator` 返回的键都来自一个数据库的一致视图(consistent view)。

[`Snapshot`](https://github.com/facebook/rocksdb/wiki/Snapshot) API 可以让应用程序创建一个数据库的时间点视图(point-in-time view)。`Get` 和 `Iterator` APIs 可以用来从一个指定的 snapshot 中读取数据，不过他俩的实现方式不一样。短时间的/前台的扫描最好通过 `Iterator` 完成，而长时间的/后台的扫描最好通过 `snapshot` 完成。`Itreator` 维护一个与数据库时间点视图相对应全部底层文件的引用计数，这些文件在 `Iterator` 被释放之前不会被删除。另一方面，`snapshot` 不会阻止文件的删除，相反，压缩进程明白 `snapshot` 的存在，并且不会删除任何在现有 `snapshot` 存在的键。  
`Snapshots` 不会在数据库重启时持久化，重新加载 RocksDB 库(通过服务器重启)会释放所有已存在的 `Snapshots`。



### 4. Transactions

RocksDB 支持多操作的事务，事务支持乐观和悲观两种模式。在 [Transactions](https://github.com/facebook/rocksdb/wiki/Transactions) 中了解更多。

### 5. Prefix Iterators

大部分 LSM 树引擎都不支持高效的范围扫描 API，因为这需要查看多个数据文件。但是大多数应用程序不会只纯粹的随机扫描数据库中一个范围的 keys；相反，应用程序往往用一个 key 前缀进行扫描操作。RocksDB 充分利用了这一点，应用程序可以配置一个选项 `Options.prefix_extractor` 来启用一个基于 key 前缀的 filter。当设置了 `Options.prefix_extractor` 时，一个前缀的 hash 也会被添加到 Bloom 中、一个指定了 key 前缀的(在 `ReadOptions` 里) `Iterator` 会使用 [Bloom filter](https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter) 来避免查看不包含指定前缀的 key 的数据文件。在 [Prefix-Seek](https://github.com/facebook/rocksdb/wiki/Prefix-Seek) 中了解更多。

### 6. Persistence

RocksDB 有一个写前日志([Write Ahead Log, WAL](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log))。所有的写操作(`Put`、`Delete` 和 `Merge`)都被存储在一个名为 memtable 的内存缓冲区中。并且可以选择是否同时把写操作插入到 WAL 中。在重启 RocksDB 时，会根据 WAL 中的记录重新处理所有事务。


可以通过配置使得 WAL 存储在与 SST 文件不同的位置，如果你想在非持久化的快速存储介质上存储所有的数据文件，把 WAL 存在与 SST 文件不同的位置就很有必要。同时，你可以在较慢的持久化存储介质上存放所有的事务日志以确保数据不会丢失。


每个 `Put` 操作都有一个标志。通过 `WriteOptions` 设置，这个标志可以指定是否把 `Put` 操作插入到事务日志中。`WriteOptions` 还可以指定在声明提交 `Put` 之前是否向事务日志发出 `fsync` 调用。

Internally, RocksDB uses a batch-commit mechanism to batch transactions into the log so that it can potentially commit multiple transactions using a single fsync call.

在内部，RocksDB 使用批处理提交机制将事务批处理到日志中，这样就可以使用单个 `fsync` 调用提交多个事务。

### 7. Data Checksuming

RocksDB 使用校验和来检测存储中的损坏，这些校验和针对每个 SST 文件块(通常大小在 4K 到 128K 之间)。块一旦写入存储，就永远不会被修改。RocksDB 还维护了一个 [<font color=red>Full File Checksum</font>](https://github.com/facebook/rocksdb/wiki/Full-File-Checksum)。

RocksDB 动态检测并利用 CPU 校验和卸载支持。

### 8. Multi-Threaded Compactions

如果有正在进行的写操作，就需要压缩(compaction)以提高空间效率、读(查询)效率，还要及时删除数据。compaction 会删除已被删除或覆盖的 key/value 绑定（因为被删除或覆盖的的 key，在底层可能没有马上删除），并重新组织数据以提高查询效率。如果配置了多线程压缩，压缩可以在多个线程中同时进行。

整个 RocksDB 数据库都存储在一组 sstfiles 中。当 memtable 满了的时后，其的内容就会被写入 LSM 树的 Level-0(L0) 中的文件。当 memtable 被 flush 到 L0 中的文件时，RocksDB会删除 memtable 中重复的和覆盖的键。在压缩过程中，一些文件会被定期读入并合并成更大的文件，这个更大的文件通常会进入下一个 LSM 级别（例如 L1，直到 Lmax）。

LSM 数据库总体的 write 吞吐量直接取决于发生压缩的速度，特别是当数据存储在 SSD 或 RAM 等快速存储介质中时。RocksDB 可以配置为，从多个线程发出并发压缩请求。可以观察到，当数据库在 ssd 上时，使用多线程压缩，与单线程压缩相比，持续写速率可能会提高 10 倍。

### 9. [Compaction Styles](https://github.com/facebook/rocksdb/wiki/Compaction)

`Level Style Compaction` 和 `Universal Style Compaction`  都将数据存储在数据库中固定数量的逻辑层中。较新的数据存储在 Level-0(L0) 中，较旧的数据存储在编号较高的级别中，最高 Lmax。L0 中的文件可能有重叠的键，但其他级别中的文件通常形成一个单独的排序运行。

`Level Style Compaction`(默认) 通常通过最小化每个压缩步骤中涉及的文件来优化磁盘占用空间和逻辑数据库大小(空间放大)：将 Ln 中的一个文件与 Ln+1 中的所有重叠文件合并，并用 Ln+1 中的新文件替换它们。

`Universal Style Compaction` 通常通过一次合并多个文件和级别来优化写入磁盘的总字节数和逻辑数据库大小(写放大)，这需要更多的临时空间。与 `Level Style Compaction` 相比，Universal 通常会导致更低的写放大，但是更高的空间放大和读放大。

`FIFO Style Compaction` 删除过时的旧文件，可以用于类似缓存的数据。在FIFO压缩中，所有文件都在级别 0。当数据的总大小超过配置的大小(`CompactionOptionsFIFO::max_table_files_size`)时，RocksDB 会删除最旧的表文件。

RocksDB 还允许开发人员开发和试验自定义的压缩策略。出于这个原因，RocksDB 有适当的 hooks 来关闭内置的压缩算法，并有其他 api 允许应用程序操作自己的压缩算法。如果设置了 `Options.disable_auto_compaction`，则禁用本机压缩算法。`GetLiveFilesMetaData` API允许外部组件查看数据库中的每个数据文件，并决定合并和压缩哪些数据文件。调用 `CompactFiles` 来压缩需要的文件。`DeleteFile` API允许应用程序删除被认为过时的数据文件。

### 10. Metadata storage

Manifest log 文件用于记录所有的数据库状态更改。压缩进程添加新文件并从数据库中删除现有文件，并通过在 [MANIFEST](https://github.com/facebook/rocksdb/wiki/MANIFEST) 中记录这些操作，使这些操作持久化。

### 11. Avoiding Stalls

后台压缩线程还用于将 memtable 内容 flush 到存储介质上的文件中。如果所有后台压缩线程都在忙着执行长时间运行的压缩操作，那么突然爆发的写操作可能会迅速填满 memtable，从而导致新的写操作停止。这种情况可以通过配置 RocksDB 来避免，以显式保留一小组线程，仅用于将 memtable 内容 flush 到存储介质。

### 12. Compaction Filter

压缩过滤器。有些应用程序可能希望在压缩时对键做些处理。例如，具有生存时间(TTL)固有支持的数据库可能会删除过期的 key，这可以通过应用程序定义的 [compaction-filter](https://github.com/facebook/rocksdb/wiki/Compaction-Filter) 来实现。如果应用程序希望连续删除比特定时间更早的数据，则可以使用 compaction-filter 删除已过期的记录。作为压缩进程的一部分，RocksDB compaction-filter 可以控制应用程序修改键的值或删除键。例如，应用程序可以连续运行 data sanitizer 作为压缩的一部分。

### 13. ReadOnly Mode

RocksDB 支持在 ReadOnly 模式下打开数据库，在这种模式下，数据库保证应用程序不能修改数据库中的任何内容。这将导致更高的读性能，因为经常遍历的代码路径可以完全避免锁定。

### 14. Database Debug Logs

默认情况下，RocksDB 将详细的日志写入 LOG* 文件。这些日志主要用于调试和分析正在运行的系统。用户可以选择不同的日志级别，也可以将此日志配置为按指定的周期滚动。日志接口是可插拔的，用户可以把日志接口插入不同的记录器。在 [Logger](https://github.com/facebook/rocksdb/wiki/Logger) 中了解更多。

### 15. Data Compression

RocksDB 支持 lz4、zstd、snappy、zlib 和 lz4_hc 压缩，以及 Windows 下的 xpress。RocksDB 为最底层数据（90% 的数据都存在最底层）的提供了不同的压缩算法，典型的安装可能会为最底层配置 ZSTD（如果不可用，则为 Zlib），为其他级别配置 LZ4（如果不可用，则为 Snappy）。在 [Compression](https://github.com/facebook/rocksdb/wiki/Compression) 中了解更多。


### 16. Full Backups and Replication

RocksDB 提供了备份 API `BackupEngine`。You can read more about it here: [How to backup RocksDB](https://github.com/facebook/rocksdb/wiki/How-to-backup-RocksDB).

RocksDB 本身不是复制的，但它提供了一些帮助功能，使用户能够在RocksDB上实现复制系统，see [Replication Helpers](https://github.com/facebook/rocksdb/wiki/Replication-Helpers).

### 17. Support for Multiple Embedded Databases in the same process

RocksDB 的一个常见用例是，应用程序固有地将数据集划分为逻辑分区或分片。这种技术有利于应用程序负载平衡和故障快速恢复。这意味着一个服务器进程应该能够同时操作多个 RocksDB 数据库，这是通过名为 `Env` 的环境对象完成的。其中，线程池与 `Env` 关联。如果应用程序希望在多个数据库实例之间共享一个公共线程池（用于后台压缩），那么它应该使用相同的 `Env` 对象来打开这些数据库。

类似地，多个数据库实例可以共享相同的块缓存或速率限制器。

### 18. Block Cache -- Compressed and Uncompressed Data

RocksDB 使用 LRU 缓存快([LRU cache for blocks](https://github.com/facebook/rocksdb/wiki/Block-Cache)) 来进行读操作。块缓存被划分为两个单独的缓存：第一个缓存未压缩的块，第二个缓存在 RAM 中压缩的块。如果配置了压缩块缓存，用户可能希望启用直接 I/O，以防止操作系统页面缓存中相同数据的冗余缓存。

### 19. Table Cache

表缓存是一个用来缓存打开的文件描述符的结构。这些文件描述符用于 **sstfiles**。应用程序可以指定表缓存的最大大小，或配置 RocksDB 始终保持所有文件打开，以获得更好的性能。

### 20. I/O Control

RocksDB 允许用户以不同的方式配置从 SST 文件到 SST 文件的 I/O。用户可以启用直接 I/O，这样 RocksDB 就可以完全控制 I/O 和缓存。另一种方法是利用一些选项让用户提示应该如何执行 I/O。他们可以建议 RocksDB 在读取文件时调用 `fadvise`，在追加文件时调用周期范围同步，启用直接I/O等。更多细节见 [IO](https://github.com/facebook/rocksdb/wiki/IO)。

### 21. Stackable DB

RocksDB 有一个内置的包装机制，可以在代码数据库内核之上添加功能层。这个功能是由 `StackableDB` API封装的。例如，time-to-live  功能是由 `StackableDB`实现的，而不是 RocksDB 核心 API 的一部分。这种方法保持代码模块化和整洁。

### 22. Memtables:

#### 22.1. Pluggable Memtables:

RocksDB memtable 的默认实现是 skiplist。skiplist 是一个已排序的集合，当工作负载将写与范围扫描交织在一起时，这是一个必要的构造。然而，有些应用程序不交叉写和扫描，有些应用程序根本不做范围扫描。对于这些应用程序，排序集可能无法提供最佳性能。因此，RocksDB 的 memtable 是可插拔的，提供了一些可选的实现。RocksDB 库有三个 memtable：skiplist memtable、vector memtable 和  prefix-hash memtable。vector memtable 适合于将数据批量加载到数据库中。每次 write 都在 vector 的末尾插入一个新元素；当需要 flush memtable 以存储时，vector 中的元素将被排序并写入 L0 中的文件。 prefix-hash memtable 允许高效地处理键前缀中的 get、put 和 scan。尽管 memtable 的可插拔性不是作为公共 API 提供的，但应用程序可以在私有 fork 中提供自己的 memtable 实现。

#### 22.2. Memtable Pipelining

RocksDB 支持为数据库配置任意数量的 memtable。当 memtable 满时，它变成一个 immutable memtable，后台线程开始将其内容 flush 到存储中。同时，新的写操作继续累积到新分配的 memtable 中。如果新分配的 memtable 已被填满，它也会被转换为 immutable memtable，并被插入到 flush 管道中。后台线程继续将所有管道化的 immutable memtable flush 到存储器中。这种流水线操作增加了 RocksDB 的写吞吐量，特别是当它运行在低速存储设备上时。

#### 22.3. Garbage Collection during Memtable Flush

当把 memtable flush 到存储时，一个内联的压缩进程也会被执行。垃圾以与压缩相同的方式被删除，相同 key 的重复 update 会被从输出流中删除。类似地，如果前面的 put 被后面的 delete 隐藏（前文说过，被删除的键在底层其实没有马上删除，只是被隐藏了起来），则该 put 根本不会写入输出文件。对于某些工作负载而言，该特性大大减少了存储上的数据大小和写入放大。

### 23. Merge Operator

RocksDB 原生支持三种类型的记录，`Put` 记录，`Delete` 记录和 `Merge` 记录。当压缩进程遇到 Merge 记录时，就会调用应用程序指定的方法，称为 Merge Operator。Merge Operator 可以将多个 Put 和 Merge 记录合并成一个单独的记录。这个强大的特性允许通常执行 read-modify-write 操作的应用程序完全避免 read 操作。RocksDB 允许应用程序将操作意图记录为 Merge record, RocksDB 压缩进程将该意图惰性地应用于原始值。该特性在 [Merge Operator](https://github.com/facebook/rocksdb/wiki/Merge-Operator) 中有详细描述。

### 24. DB ID

DB ID 是 RocksDB 在创建数据库时生成的全局唯一 ID，默认情况下 DB ID 存储在 DB 文件夹的 IDENTITY 文件中。用户也可以选择把 DB ID 仅存储在 MANIFEST 文件中。RocksDB 建议把 DB ID 存储在 MANIFEST 文件中。

### 25. 参考资料

https://github.com/facebook/rocksdb/wiki/RocksDB-Overview