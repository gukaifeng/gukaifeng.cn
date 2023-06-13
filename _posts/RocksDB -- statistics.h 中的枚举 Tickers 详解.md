


> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>





RocksDB 的 Tickers 枚举定义在 `statistics.h` 中，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L21-L417)。

`statistics.h` 中还声明了一个 Tickers 枚举值与 rocksdb 属性的一一对应关系，相关代码在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/statistics.h#L419-L421)，而具体的定义在 `statistics.cc` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/monitoring/statistics.cc#L20-L217)。

下面按照 `statistics.h` 中 Tickers 枚举的属性定义顺序，来逐个介绍。



#### 1. BLOCK_CACHE_MISS

`Tickers::BLOCK_CACHE_MISS` 对应的 rocksdb 属性是 `rocksdb.block.cache.miss`。



#### 2. BLOCK_CACHE_HIT

`Tickers::BLOCK_CACHE_HIT` 对应的 rocksdb 属性是 `rocksdb.block.cache.hit`。

