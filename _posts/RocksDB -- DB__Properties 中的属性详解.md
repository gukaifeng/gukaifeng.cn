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

这个属性其实就是 [6. kCFStatsNoFileHistogram](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#6-kCFStatsNoFileHistogram) 和 [7. kCFFileHistogram](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#7-kCFFileHistogram) 和 [8. kDBStats](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#8-kDBStats) 的组合。

所以这个属性我们在下面 6. 和 7. 和 8. 小节来说，内容是一模一样的。



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

\- **`328`**

信息一开始的 `328` 表示 sst 文件的编号，你可以在数据库目录里找到对应的 sst 文件，就像下图这样。

这里这个 sst 文件名至少是 6 位的，所以我们编号 `328` 的 `sst` 文件实际文件名是 `000328.sst`，当我们有更多 sst 文件时，这个文件名编号是可以超过 6 位的。

![000328.sst](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie_1.png)

\- **`30430428`**

这个表示 sst 文件的大小，单位字节。这里代表 `000328.sst` 这个文件的大小是 30430428 字节。

\- **`[84038683 .. 85066084]`**

`84038683 ` 是这个 sst 文件的最小序列号(sequence number)，`85066084` 是这个 sst 文件的最大序列号。

\- **`['6B65792D746573743638373435353539' seq:84038683, type:1 .. '6B65792D746573743639373732393630' seq:85066084, type:1]`**

`'6B65792D746573743638373435353539' seq:84038683, type:1` 是最小的内部键(interval key)解析后的信息，  
`'6B65792D746573743639373732393630' seq:85066084, type:1` 是最大的内部键解析后的信息。

这里以最小的内部键解析后的信息举例：

\- \- `'6B65792D746573743638373435353539'`  是 key 的 16 进制编码，可以通过这个编码解析出原始的 key 的。这个编码的长度将是原始 key （Slice 类型）长度的两倍，原因是原来是一个字符 1 个字节（8 位），用 16 进制编码，是将原来的数据每 4 位编成一个 16 进制值，所以返回的长度将是原来数据长度的两倍。

\- \- `seq:84038683` 是这个 key 的序列号，与上述的 sst 文件的最小序列号是一样的。

\- \- `type:1` 是这个 key 的 value 的类型，`1` 表示这是一个正常的值。具体关于此值的定义详见 [`dbformat.h`](https://github.com/facebook/rocksdb/blob/v6.25.3/db/dbformat.h#L39-L74)，很清楚，这里就不多赘述了。



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

这是一个多行 string，包含了数据库中关于 column family 的一些数据统计信息。

这个属性其实就是 [6. kCFStatsNoFileHistogram](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#6-kCFStatsNoFileHistogram) 和 [7. kCFFileHistogram](https://gukaifeng.cn/posts/rocksdb-db-properties-zhong-de-shu-xing-xiang-jie/#7-kCFFileHistogram) 的组合。

所以这个属性我们在下面 6. 和 7. 小节来说，内容是一模一样的。



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

我们先看一下这个属性值的示例，再逐一来说其每个值的含义。

```
** Compaction Stats [default] **
Level    Files      Size     Score Read(GB)  Rn(GB) Rnp1(GB) Write(GB) Wnew(GB) Moved(GB) W-Amp Rd(MB/s) Wr(MB/s) Comp(sec) CompMergeCPU(sec) Comp(cnt) Avg(sec) KeyIn KeyDrop Rblob(GB) Wblob(GB)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  L0      6/5       2.12 GB   1.4      0.0     0.0      0.0  199099.8 199099.8       0.0   1.0      0.0    269.5 756504.74         250346.23    554853    1.363       0      0       0.0       0.0
  L1     38/38      1.12 GB   0.0 249127.6 199097.7  50029.9  248731.1 198701.2       0.0   1.2    134.9    134.7 1890421.55         452407.19    200791    9.415     93G   156M       0.0       0.0
  L2     60/0     979.08 MB   1.0 213539.6 198700.1  14839.5   86964.5  72125.0       0.0   0.4     13.0      5.3 16830463.52         691433.34   4870877    3.455     89G   580M       0.0       0.0
  L3    309/2       4.05 GB   1.0  28156.2 19799.1   8357.2   23635.3  15278.1   52325.9   1.2      9.3      7.8 3109912.67         407217.85    268347   11.589     29G  2260M       0.0       0.0
  L4    982/15     16.19 GB   1.0  48411.4 25620.9  22790.4   33402.8  10612.4   41983.0   1.3      9.5      6.6 5221179.63         716863.16    758257    6.886     52G    13G       0.0       0.0
  L5    205/2      29.18 GB   0.5 146746.1 52595.2  94151.0   94151.0      0.0       0.0   1.8     10.1      6.5 14814832.53        2756096.66    494153   29.980    182G    58G       0.0       0.0
 Sum   1600/62     53.61 GB   0.0 685981.0 495813.0 190168.0  685984.5 495816.4   94308.9   3.4     16.5     16.5 42623314.64        5274364.44   7147278    5.964    446G    74G       0.0       0.0
 Int      0/0       0.00 KB   0.0      0.4     0.4      0.1       0.7      0.6       0.0   2.0     10.5     17.3     41.22              6.14         3   13.740    449K    43K       0.0       0.0

** Compaction Stats [default] **
Priority Files      Size     Score Read(GB)  Rn(GB) Rnp1(GB) Write(GB) Wnew(GB) Moved(GB) W-Amp Rd(MB/s) Wr(MB/s) Comp(sec) CompMergeCPU(sec) Comp(cnt) Avg(sec) KeyIn KeyDrop Rblob(GB) Wblob(GB)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Low      0/0       0.00 KB   0.0 685981.0 495813.0 190168.0  486884.7 296716.6       0.0   0.0     16.8     11.9 41866809.91        5024018.21   6592425    6.351    446G    74G       0.0       0.0
High      0/0       0.00 KB   0.0      0.0     0.0      0.0  199099.8 199099.8       0.0   0.0      0.0    269.5 756504.74         250346.23    554853    1.363       0      0       0.0       0.0

Blob file count: 0, total size: 0.0 GB

Uptime(secs): 4728090.8 total, 14.0 interval
Flush(GB): cumulative 199099.803, interval 0.356
AddFile(GB): cumulative 0.000, interval 0.000
AddFile(Total Files): cumulative 0, interval 0
AddFile(L0 Files): cumulative 0, interval 0
AddFile(Keys): cumulative 0, interval 0
Cumulative compaction:  669.906709 TB write,  148.57 MB/s write, 685980.985360 GB read,  148.57 MB/s read, 42623314.6 seconds
Interval   compaction:    0.694616 GB write,   50.67 MB/s write,    0.424185 GB read,   30.94 MB/s read,    41.2 seconds
Stalls(count): 2273408 level0_slowdown, 1603632 level0_slowdown_with_compaction, 0 level0_numfiles, 0 level0_numfiles_with_compaction, 0 stop for pending_compaction_bytes, 10738 slowdown for pending_compaction_bytes, 0 memtable_compaction, 2 memtable_slowdown, interval 9 total count
```


我们可以看到上面上班部分有两个“表格”，标题都是 `** Compaction Stats [default] **`，  
两个表格第一行的表头是一样的，只有下面的内容不同，我们这里先说第一行表头的含义，`Level` 列下面不同的值会在这个属性的解释内说明。

* `Level` : 用于级别压缩 LSM 的级别。
    * `L<N>` : 表示 LSM 的级别，如 L0，L3 等。对于 universal compaction，所有文件都在 L0 中。
    * `Sum` : 表示 `L<N>` 各属性的和。
    * `Int` : 与 `Sum` 类似，也是各层的和，但只包含来自上一个报告间隔的数据。需要注意这里是单词 "Interval" 的缩写，表示“间隔”，而不是整数类型。
    * `Low`（第二个表格）: 
    * `High`（第二个表格）: 
* `Files` : 这有两个值 `a/b`，a 是此 level 中的 sst 文件数，b 是此 level 当前正在进行压缩的 sst 文件数。
* `Size` : 此 level 全部的 sst 文件总大小。
* `Score` : 当前 level 的 compaction 最高分数。  
    当有多个 level 触发 compaction 条件时，就要依靠分数选择执行 compaction 的 level，分数越高越优先。  
    分数可以是 0 或 1，分数大于 1 的 level 是需要被压缩的。  
    L0 的分数计算方法是 `num L0 files` / `level0_file_num_compaction_trigger`，或 L0 的总大小减去 `max_bytes_for_level_base`，二者取较大的。  
    L1 及以上的分数计算方法是 `Level files size` / `MaxBytesForLevel`。
* `Read(GB)` : 
* `Rn(GB)` : 
* `Rnp1(GB)` : 
* `Write(GB)` : 
* `Wnew(GB)` : 
* `Moved(GB)` : 
* `W-Amp` : 
* `Rd(MB/s)` : 
* `Wr(MB/s)` : 
* `Comp(sec)` : 
* `CompMergeCPU(sec)` : 
* `Comp(cnt)` : 
* `Avg(sec)` : 
* `KeyIn` : 
* `KeyDrop` : 
* `Rblob(GB)` : 
* `Wblob(GB)` : 






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

下面是一个真实世界的 rocksdb 实例的此属性输出，我们先看一下此属性的值，再分析其含义。

```
** File Read Latency Histogram By Level [default] **
** Level 0 read latency histogram (micros):
Count: 5548530 Average: 0.2112  StdDev: 8.12
Min: 0  Median: 0.5042  Max: 15736
Percentiles: P50: 0.50 P75: 0.76 P99: 1.00 P99.9: 13.41 P99.99: 53.80
------------------------------------------------------
[       0,       1 ]  5501962  99.161%  99.161% ####################
(       1,       2 ]    22660   0.408%  99.569% 
(       2,       3 ]     3815   0.069%  99.638% 
(       3,       4 ]      859   0.015%  99.653% 
(       4,       6 ]     3149   0.057%  99.710% 
(       6,      10 ]     8415   0.152%  99.862% 
(      10,      15 ]     3111   0.056%  99.918% 
(      15,      22 ]     1953   0.035%  99.953% 
(      22,      34 ]     1316   0.024%  99.977% 
(      34,      51 ]      700   0.013%  99.989% 
(      51,      76 ]      314   0.006%  99.995% 
(      76,     110 ]       93   0.002%  99.997% 
(     110,     170 ]       61   0.001%  99.998% 
(     170,     250 ]       31   0.001%  99.998% 
(     250,     380 ]       30   0.001%  99.999% 
(     380,     580 ]       22   0.000%  99.999% 
(     580,     870 ]       13   0.000% 100.000% 
(     870,    1300 ]       15   0.000% 100.000% 
(    1300,    1900 ]        7   0.000% 100.000% 
(    1900,    2900 ]        2   0.000% 100.000% 
(    6600,    9900 ]        1   0.000% 100.000% 
(   14000,   22000 ]        1   0.000% 100.000% 

** Level 1 read latency histogram (micros):
Count: 90331511 Average: 0.1316  StdDev: 8.13
Min: 0  Median: 0.5028  Max: 46108
Percentiles: P50: 0.50 P75: 0.75 P99: 1.00 P99.9: 18.06 P99.99: 46108.00
------------------------------------------------------
[       0,       1 ] 89833074  99.448%  99.448% ####################
(       1,       2 ]   123161   0.136%  99.585% 
(       2,       3 ]    59917   0.066%  99.651% 
(       3,       4 ]     7118   0.008%  99.659% 
(       4,       6 ]    54551   0.060%  99.719% 
(       6,      10 ]   100879   0.112%  99.831% 
(      10,      15 ]    46145   0.051%  99.882% 
(      15,      22 ]    37344   0.041%  99.923% 
(      22,      34 ]    29940   0.033%  99.956% 
(      34,      51 ]    14982   0.017%  99.973% 
(      51,      76 ]     6872   0.008%  99.981% 
(      76,     110 ]     2243   0.002%  99.983% 
(     110,     170 ]     1058   0.001%  99.984% 
(     170,     250 ]      405   0.000%  99.985% 
(     250,     380 ]      244   0.000%  99.985% 
(     380,     580 ]      156   0.000%  99.985% 
(     580,     870 ]      117   0.000%  99.985% 
(     870,    1300 ]      109   0.000%  99.985% 
(    1300,    1900 ]       48   0.000%  99.985% 
(    1900,    2900 ]       81   0.000%  99.986% 
(    2900,    4400 ]       89   0.000%  99.986% 
(    4400,    6600 ]       29   0.000%  99.986% 
(    6600,    9900 ]        9   0.000%  99.986% 
(    9900,   14000 ]        4   0.000%  99.986% 
(   33000,   50000 ]        1   0.000%  99.986% 

** Level 2 read latency histogram (micros):
Count: 10272 Average: 0.1310  StdDev: 0.59
Min: 0  Median: 0.5025  Max: 22
Percentiles: P50: 0.50 P75: 0.75 P99: 0.99 P99.9: 9.73 P99.99: 18.40
------------------------------------------------------
[       0,       1 ]    10221  99.504%  99.504% ####################
(       1,       2 ]       21   0.204%  99.708% 
(       2,       3 ]        6   0.058%  99.766% 
(       3,       4 ]        1   0.010%  99.776% 
(       4,       6 ]        9   0.088%  99.864% 
(       6,      10 ]        4   0.039%  99.903% 
(      10,      15 ]        8   0.078%  99.981% 
(      15,      22 ]        2   0.019% 100.000% 

** Level 3 read latency histogram (micros):
Count: 7024 Average: 0.0209  StdDev: 0.51
Min: 0  Median: 0.5005  Max: 32
Percentiles: P50: 0.50 P75: 0.75 P99: 0.99 P99.9: 1.00 P99.99: 29.79
------------------------------------------------------
[       0,       1 ]     7017  99.900%  99.900% ####################
(       1,       2 ]        2   0.028%  99.929% 
(       2,       3 ]        1   0.014%  99.943% 
(       3,       4 ]        1   0.014%  99.957% 
(       6,      10 ]        1   0.014%  99.972% 
(      22,      34 ]        2   0.028% 100.000% 

** Level 4 read latency histogram (micros):
Count: 17312 Average: 0.0155  StdDev: 0.17
Min: 0  Median: 0.5003  Max: 14
Percentiles: P50: 0.50 P75: 0.75 P99: 0.99 P99.9: 1.00 P99.99: 4.54
------------------------------------------------------
[       0,       1 ]    17300  99.931%  99.931% ####################
(       1,       2 ]        7   0.040%  99.971% 
(       2,       3 ]        1   0.006%  99.977% 
(       3,       4 ]        2   0.012%  99.988% 
(       4,       6 ]        1   0.006%  99.994% 
(      10,      15 ]        1   0.006% 100.000% 

** Level 5 read latency histogram (micros):
Count: 3888 Average: 0.1821  StdDev: 9.55
Min: 0  Median: 0.5009  Max: 595
Percentiles: P50: 0.50 P75: 0.75 P99: 0.99 P99.9: 2.11 P99.99: 595.00
------------------------------------------------------
[       0,       1 ]     3881  99.820%  99.820% ####################
(       1,       2 ]        3   0.077%  99.897% 
(       2,       3 ]        1   0.026%  99.923% 
(       3,       4 ]        1   0.026%  99.949% 
(      15,      22 ]        1   0.026%  99.974% 
(     580,     870 ]        1   0.026% 100.000% 
```





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



