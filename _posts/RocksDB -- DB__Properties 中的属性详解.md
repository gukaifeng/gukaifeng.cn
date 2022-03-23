---
title: RocksDB -- DB::Properties 中的属性详解
date: 2022-03-02 16:42:26
updated: 2022-03-02 16:42:26
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



RocksDB 中的 DB 类中定义了一个结构体 Properties，里面是许多 RocksDB 的属性。

具体代码在 `db.h` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L748-L987)查看。

结构体 Properties 中的每个成员都对应一个 rocksdb 属性，我们可以通过 `GetProperty()` 或 `GetMapProperty()` 等方法来获取其中的内容。具体方法说明详见 [获取属性值的方法](https://gukaifeng.cn/posts/rocksdb-db-lei/#18-2-获取属性值的方法)。

例如：

```cpp
#include <iostream>
#include <cassert>
#include "rocksdb/db.h"

int main(int argc, char *argv[])
{
    rocksdb::DB *db;
    rocksdb::Options options;
    options.create_if_missing = true;
    rocksdb::Status status =
        rocksdb::DB::Open(options, "./testdb", &db);
    assert(status.ok());
  
    std::string p = "rocksdb.num-files-at-level2";  // 获取该属性值
    std::string v;  // 存储待获取属性值
  
    db->GetProperty(p, &v);  // 执行 GetProperty() 方法
    std::cout << v << std::endl;  // 输出结果

    db->Close();
    delete db;

    return 0;
}
```

上面的代码演示了获取 rocksdb 属性值的方法之一。

关于示例代码中 `"rocksdb.num-files-at-level2"` 的介绍详见 [1. kNumFilesAtLevelPrefix](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#1-kNumFilesAtLevelPrefix)

我这里上面代码执行后的输出如下：

```
3
```

这表示我当前的 rocksdb 数据库 `"./testdb"` 中 level2 中一共有 3 个 sst 文件。

\-

下面我会按照这些属性在源码中的定义顺序来逐个介绍它们，较为复杂的会举例说明。

部分带有类似 `level<N>` 字样的属性中的 `<N>` 表示 level 数，例如 level2。

## 1. kNumFilesAtLevelPrefix

**定义**

```cpp
static const std::string kNumFilesAtLevelPrefix;
```

**属性**

```
rocksdb.num-files-at-level<N>
```

**含义**

表示 `L<N>` 中的全部 sst 文件数。

如 `rocksdb.num-files-at-level2` 表示 level 2 中全部的 sst 文件数。



## 2. kCompressionRatioAtLevelPrefix

**定义**

```cpp
static const std::string kCompressionRatioAtLevelPrefix;
```

**属性**

```
rocksdb.compression-ratio-at-level<N>
```

**含义**

表示 `L<N>` 的数据压缩率。

**数据压缩率 = 未压缩的数据大小 / 压缩后的文件大小**

若 `L<N>` 没有打开的文件，则此属性值为 `"-1.0"`.



## 3. kStats

**定义**

```cpp
static const std::string kStats;
```

**属性**

```
rocksdb.stats
```

**含义**

这是一个多行 string，包含了数据库里的一些数据统计信息。

这个属性其实就是 [5. kCFStats](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#5-kCFStats) 和 [8. kDBStats](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#8-kDBStats) 的组合，内容是一模一样的，所以我们在下面具体的属性内专门再说。



## 4. kSSTables

**定义**

```cpp
static const std::string kSSTables;
```

**属性**

```
rocksdb.sstables
```

**含义**





## 5. kCFStats

**定义**

```cpp
static const std::string kCFStats;
```

**属性**

```
rocksdb.cfstats
```

**含义**





## 6. kCFStatsNoFileHistogram

**定义**

```cpp
static const std::string kCFStatsNoFileHistogram;
```

**属性**

```
rocksdb.cfstats-no-file-histogram
```

**含义**





## 7. kCFFileHistogram

**定义**

```cpp
static const std::string kCFFileHistogram;
```

**属性**

```
rocksdb.cf-file-histogram
```

**含义**





## 8. kDBStats

**定义**

```cpp
static const std::string kDBStats;
```

**属性**

```
rocksdb.dbstats
```

**含义**





## 9. kLevelStats

**定义**

```cpp
static const std::string kLevelStats;
```

**属性**

```
rocksdb.levelstats
```

**含义**





## 10. kBlockCacheEntryStats

**定义**

```cpp
static const std::string kBlockCacheEntryStats;
```

**属性**

```
rocksdb.block-cache-entry-stats
```

**含义**





## 11. kNumImmutableMemTable

**定义**

```cpp
static const std::string kNumImmutableMemTable;
```

**属性**

```
rocksdb.num-immutable-mem-table
```

**含义**





## 12. kNumImmutableMemTableFlushed

**定义**

```cpp
static const std::string kNumImmutableMemTableFlushed;
```

**属性**

```
rocksdb.num-immutable-mem-table-flushed
```

**含义**





## 13. kMemTableFlushPending

**定义**

```cpp
static const std::string kMemTableFlushPending;
```

**属性**

```
rocksdb.mem-table-flush-pending
```

**含义**





## 14. kNumRunningFlushes

**定义**

```cpp
static const std::string kNumRunningFlushes;
```

**属性**

```
rocksdb.num-running-flushes
```

**含义**





## 15. kCompactionPending

**定义**

```cpp
static const std::string kCompactionPending;
```

**属性**

```
rocksdb.compaction-pending
```

**含义**





## 16. kNumRunningCompactions

**定义**

```cpp
static const std::string kNumRunningCompactions;
```

**属性**

```
rocksdb.num-running-compactions
```

**含义**





## 17. kBackgroundErrors

**定义**

```cpp
static const std::string kBackgroundErrors;
```

**属性**

```
rocksdb.background-errors
```

**含义**





## 18. kCurSizeActiveMemTable

**定义**

```cpp
static const std::string kCurSizeActiveMemTable;
```

**属性**

```
rocksdb.cur-size-active-mem-table
```

**含义**





## 19. kCurSizeAllMemTables

**定义**

```cpp
static const std::string kCurSizeAllMemTables;
```

**属性**

```
rocksdb.cur-size-all-mem-tables
```

**含义**





## 20. kSizeAllMemTables

**定义**

```cpp
static const std::string kSizeAllMemTables;
```

**属性**

```
rocksdb.size-all-mem-tables
```

**含义**





## 21. kNumEntriesActiveMemTable

**定义**

```cpp
static const std::string kNumEntriesActiveMemTable;
```

**属性**

```
rocksdb.num-entries-active-mem-table
```

**含义**





## 22. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 23. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 24. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 25. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 26. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 27. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 28. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 29. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 30. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 31. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 32. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 33. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 34. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 35. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 36. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 37. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 38. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 39. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 40. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 41. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 42. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 43. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 44. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 45. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 46. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 47. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 48. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 49. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 50. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 51. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 52. 

**定义**

```cpp

```

**属性**

```

```

**含义**





## 53. 

**定义**

```cpp

```

**属性**

```

```

**含义**



