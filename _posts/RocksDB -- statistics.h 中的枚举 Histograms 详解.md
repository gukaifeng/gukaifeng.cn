


> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>





RocksDB 的 Histograms 枚举定义在 `statistics.h` 中，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L423-#L520)。

`statistics.h` 中还声明了一个 Histograms 枚举值与 rocksdb 属性的一一对应关系，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L522)，而具体的定义在 `statistics.cc` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/monitoring/statistics.cc#L219-L274)。

下面按照 `statistics.h` 中 Histograms 枚举的属性定义顺序，来逐个介绍。







#### 1. DB_GET

`Histograms::DB_GET` 对应的 rocksdb 属性为 `rocksdb.db.get.micros`。



#### 2. DB_WRITE

`Histograms::DB_WRITE` 对应的 rocksdb 属性为 `rocksdb.db.write.micros`。





```
{DB_GET, "rocksdb.db.get.micros"},
{DB_WRITE, "rocksdb.db.write.micros"},
{COMPACTION_TIME, "rocksdb.compaction.times.micros"},
{COMPACTION_CPU_TIME, "rocksdb.compaction.times.cpu_micros"},
{SUBCOMPACTION_SETUP_TIME, "rocksdb.subcompaction.setup.times.micros"},
{TABLE_SYNC_MICROS, "rocksdb.table.sync.micros"},
{COMPACTION_OUTFILE_SYNC_MICROS, "rocksdb.compaction.outfile.sync.micros"},
{WAL_FILE_SYNC_MICROS, "rocksdb.wal.file.sync.micros"},
{MANIFEST_FILE_SYNC_MICROS, "rocksdb.manifest.file.sync.micros"},
{TABLE_OPEN_IO_MICROS, "rocksdb.table.open.io.micros"},
{DB_MULTIGET, "rocksdb.db.multiget.micros"},
{READ_BLOCK_COMPACTION_MICROS, "rocksdb.read.block.compaction.micros"},
{READ_BLOCK_GET_MICROS, "rocksdb.read.block.get.micros"},
{WRITE_RAW_BLOCK_MICROS, "rocksdb.write.raw.block.micros"},
{STALL_L0_SLOWDOWN_COUNT, "rocksdb.l0.slowdown.count"},
{STALL_MEMTABLE_COMPACTION_COUNT, "rocksdb.memtable.compaction.count"},
{STALL_L0_NUM_FILES_COUNT, "rocksdb.num.files.stall.count"},
{HARD_RATE_LIMIT_DELAY_COUNT, "rocksdb.hard.rate.limit.delay.count"},
{SOFT_RATE_LIMIT_DELAY_COUNT, "rocksdb.soft.rate.limit.delay.count"},
{NUM_FILES_IN_SINGLE_COMPACTION, "rocksdb.numfiles.in.singlecompaction"},
{DB_SEEK, "rocksdb.db.seek.micros"},
{WRITE_STALL, "rocksdb.db.write.stall"},
{SST_READ_MICROS, "rocksdb.sst.read.micros"},
{NUM_SUBCOMPACTIONS_SCHEDULED, "rocksdb.num.subcompactions.scheduled"},
{BYTES_PER_READ, "rocksdb.bytes.per.read"},
{BYTES_PER_WRITE, "rocksdb.bytes.per.write"},
{BYTES_PER_MULTIGET, "rocksdb.bytes.per.multiget"},
{BYTES_COMPRESSED, "rocksdb.bytes.compressed"},
{BYTES_DECOMPRESSED, "rocksdb.bytes.decompressed"},
{COMPRESSION_TIMES_NANOS, "rocksdb.compression.times.nanos"},
{DECOMPRESSION_TIMES_NANOS, "rocksdb.decompression.times.nanos"},
{READ_NUM_MERGE_OPERANDS, "rocksdb.read.num.merge_operands"},
{BLOB_DB_KEY_SIZE, "rocksdb.blobdb.key.size"},
{BLOB_DB_VALUE_SIZE, "rocksdb.blobdb.value.size"},
{BLOB_DB_WRITE_MICROS, "rocksdb.blobdb.write.micros"},
{BLOB_DB_GET_MICROS, "rocksdb.blobdb.get.micros"},
{BLOB_DB_MULTIGET_MICROS, "rocksdb.blobdb.multiget.micros"},
{BLOB_DB_SEEK_MICROS, "rocksdb.blobdb.seek.micros"},
{BLOB_DB_NEXT_MICROS, "rocksdb.blobdb.next.micros"},
{BLOB_DB_PREV_MICROS, "rocksdb.blobdb.prev.micros"},
{BLOB_DB_BLOB_FILE_WRITE_MICROS, "rocksdb.blobdb.blob.file.write.micros"},
{BLOB_DB_BLOB_FILE_READ_MICROS, "rocksdb.blobdb.blob.file.read.micros"},
{BLOB_DB_BLOB_FILE_SYNC_MICROS, "rocksdb.blobdb.blob.file.sync.micros"},
{BLOB_DB_GC_MICROS, "rocksdb.blobdb.gc.micros"},
{BLOB_DB_COMPRESSION_MICROS, "rocksdb.blobdb.compression.micros"},
{BLOB_DB_DECOMPRESSION_MICROS, "rocksdb.blobdb.decompression.micros"},
{FLUSH_TIME, "rocksdb.db.flush.micros"},
{SST_BATCH_SIZE, "rocksdb.sst.batch.size"},
{NUM_INDEX_AND_FILTER_BLOCKS_READ_PER_LEVEL,
"rocksdb.num.index.and.filter.blocks.read.per.level"},
{NUM_DATA_BLOCKS_READ_PER_LEVEL, "rocksdb.num.data.blocks.read.per.level"},
{NUM_SST_READ_PER_LEVEL, "rocksdb.num.sst.read.per.level"},
{ERROR_HANDLER_AUTORESUME_RETRY_COUNT,
"rocksdb.error.handler.autoresume.retry.count"},
```

#### 47. FLUSH_TIME

`Histograms::FLUSH_TIME` 对应的 rocksdb 属性为 `rocksdb.db.flush.micros`。

含义：flush 操作的时间，单位 μs。

注意，rocksdb 在内部统计中将 flush 视为 level 0 的 compaction。



