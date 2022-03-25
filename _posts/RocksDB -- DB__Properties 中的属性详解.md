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

关于示例代码中 `"rocksdb.num-files-at-level2"` 的介绍详见 [1. kNumFilesAtLevelPrefix](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#1-kNumFilesAtLevelPrefix)

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

若 `L<N>` 没有打开的文件，则此属性值为 `"-1.0"`.



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

这个属性其实就是 [5. kCFStats](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#5-kCFStats) 和 [8. kDBStats](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#8-kDBStats) 的组合，内容是一模一样的，所以我们在下面具体的属性内专门再说。



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

SST 文件的**简略**信息。是一个多行 stirng。

具体信息形如：

```
--- level 0 --- version# 1 ---
 179:30428945[27533969 .. 28561320]['6B65792D746573743132323430383435' seq:27533969, type:1 .. '6B65792D746573743133323638313936' seq:28561320, type:1](0)
 177:30427512[26506664 .. 27533968]['6B65792D746573743131323133353430' seq:26506664, type:1 .. '6B65792D746573743132323430383434' seq:27533968, type:1](0)
 171:30429788[25479280 .. 26506663]['6B65792D746573743130313836313536' seq:25479280, type:1 .. '6B65792D746573743131323133353339' seq:26506663, type:1](0)
--- level 1 --- version# 1 ---
 328:30430428[84038683 .. 85066084]['6B65792D746573743638373435353539' seq:84038683, type:1 .. '6B65792D746573743639373732393630' seq:85066084, type:1](0)
 335:30430540[87120855 .. 88148260]['6B65792D746573743731383237373331' seq:87120855, type:1 .. '6B65792D746573743732383535313336' seq:88148260, type:1](0)
...
 414:30430099[113832283 .. 114859673]['6B65792D746573743938353339313539' seq:113832283, type:1 .. '6B65792D746573743939353636353439' seq:114859673, type:1](0)
--- level 2 --- version# 1 ---
 224:30430799[43971552 .. 44998968]['6B65792D746573743238363738343238' seq:43971552, type:1 .. '6B65792D746573743239373035383434' seq:44998968, type:1](0)
 250:30431766[53217472 .. 54244918]['6B65792D746573743337393234333438' seq:53217472, type:1 .. '6B65792D746573743338393531373934' seq:54244918, type:1](0)
 279:30428744[65545776 .. 66573120]['6B65792D746573743530323532363532' seq:65545776, type:1 .. '6B65792D746573743531323739393936' seq:66573120, type:1](0)
 299:30429959[72737469 .. 73764854]['6B65792D746573743537343434333435' seq:72737469, type:1 .. '6B65792D746573743538343731373330' seq:73764854, type:1](0)
--- level 3 --- version# 1 ---
 205:67568579[0 .. 0]['616765' seq:0, type:1 .. '6B65792D746573743134383534363630' seq:0, type:1](0)
 206:67571849[0 .. 0]['6B65792D746573743134383534363631' seq:0, type:1 .. '6B65792D746573743136393235303634' seq:0, type:1](0)
...
 120:26837417[0 .. 0]['6B657938393436343430' seq:0, type:1 .. '6E616D65' seq:0, type:1](0)
--- level 4 --- version# 1 ---
--- level 5 --- version# 1 ---
--- level 6 --- version# 1 ---
```

可以看到，每个 level 都有描述，即便没有 sst 文件的 levle 4,5,6 中也同样有对应的标题。

\-

我们这里取其中一条来分析：

```
 328:30430428[84038683 .. 85066084]['6B65792D746573743638373435353539' seq:84038683, type:1 .. '6B65792D746573743639373732393630' seq:85066084, type:1](0)
```

这是上面 level 1 的第 1 条信息。

* **`328`**

信息一开始的 `328` 表示 sst 文件的编号，你可以在数据库目录里找到对应的 sst 文件，就像下图这样。

这里这个 sst 文件名至少是 6 位的，所以我们编号 `328` 的 `sst` 文件实际文件名是 `000328.sst`，当我们有更多 sst 文件时，这个文件名编号是可以超过 6 位的。

![000328.sst](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie_1.png)

* **`30430428`**

这个表示 sst 文件的大小，单位字节。这里代表 `000328.sst` 这个文件的大小是 30430428 字节。

* **`[84038683 .. 85066084]`**

`84038683 ` 是这个 sst 文件的最小序列号(sequence number)，`85066084` 是这个 sst 文件的最大序列号。

* **`['6B65792D746573743638373435353539' seq:84038683, type:1 .. '6B65792D746573743639373732393630' seq:85066084, type:1]`**

`'6B65792D746573743638373435353539' seq:84038683, type:1` 是最小的内部键解析后的信息，  
`'6B65792D746573743639373732393630' seq:85066084, type:1` 是最大的内部键解析后的信息。

这里以最小的内部键解析后的信息举例：

\- `'6B65792D746573743638373435353539'` 

\- `seq:84038683` 是这个键的序列号，与上述的 sst 文件的最小序列号是一样的。

\- `type:1` 是这个键的值的类型，`1` 表示这是一个正常的值。具体关于此值的定义详见 [`dbformat.h`](https://github.com/facebook/rocksdb/blob/v6.25.3/db/dbformat.h#L39-L74)，很清楚，这里就不多赘述了。

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



