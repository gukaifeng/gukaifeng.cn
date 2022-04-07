---
title: RocksDB -- DB::Properties 中的属性详解
date: 2022-03-02 16:42:26
updated: 2022-04-01 18:52:26
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



RocksDB 中的 DB 类中定义了一个结构体 Properties，里面是许多 RocksDB 的属性。

具体代码在 `db.h` 中，在[这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L748-L987)查看。

结构体 Properties 中的每个成员都对应一个 rocksdb 属性，我们可以通过 `GetProperty()` 或 `GetMapProperty()` 等方法来获取其中的内容。具体方法说明详见 [获取属性值的方法](https://gukaifeng.cn/posts/rocksdb-db-lei/#18-2-获取属性值的方法)。

**本文的所有属性示例均使用 `GetProperty()`，其他的方法的输出内容可能会略有不同，但大体一样。**

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


我们可以看到上面上班部分有两个“表格”，标题都是 `** Compaction Stats [default] **`（我们知道 rocksdb 中的 key 是按列族管理的，这里的 `default` 指的就是默认列族），然后是一行关于 Blob 文件的信息，最后是多行统计信息。下面我们逐一说明。

\-

我们先看第一个表格：

* `Level` : 用于级别压缩 LSM 的级别。
    * `L<N>` : 表示 LSM 的级别，如 L0，L3 等。对于 universal compaction，所有文件都在 L0 中。
    * `Sum` : 表示 `L<N>` 各属性的和。
    * `Int` : 与 `Sum` 类似，也是各层的和，但只包含来自上一个报告间隔的数据。需要注意这里是单词 "Interval" 的缩写，表示“间隔”，而不是整数类型。
* `Files` : 这有两个值 `a/b`，a 是此 level 中的 sst 文件数，b 是此 level 当前正在进行压缩的 sst 文件数。
* `Size` : 此 level 全部的 sst 文件总大小。
* `Score` : 当前 level 的 compaction 最高分数。  
    当有多个 level 触发 compaction 条件时，就要依靠分数选择执行 compaction 的 level，分数越高越优先。  
    分数 0 或 1 都是正确的值，分数大于 1 的 level 是需要被压缩的。  
    L0 的分数计算方法是 L0 的文件数除以 `level0_file_num_compaction_trigger`，或 L0 的总大小减去 `max_bytes_for_level_base`，二者取较大的。这里要注意，如果第 L0 的文件数小于 `level0_file_num_compaction_trigger`，那无论分数多高都不会执行 compaction。  
    L1 及以上的分数计算方法是 `Level files size` / `MaxBytesForLevel`。
* `Read(GB)` : 在 `L<N>` 到 `L<N+1>` 之间的 compaction 中总共读取的数据大小，包括从 `L<N>` 和从 `L<N+1>` 读的。即 `Read(GB)` = `Rn(GB)` + `Rnp1(GB)`。
* `Rn(GB)` : 在 `L<N>` 到 `L<N+1>` 之间的 compaction 中，仅包含从 `L<N>` 读取的数据大小。
* `Rnp1(GB)` : 在 `L<N>` 到 `L<N+1>` 之间的 compaction 中，仅包含从 `L<N+1>` 读取的数据大小。
* `Write(GB)` : 在 `L<N>` 到 `L<N+1>` 之间的 compaction 中总共写入的数据大小。
* `Wnew(GB)` : 写入 `L<N+1>` 的新字节数。计算方法是在 `L<N>` 的 compaction 中写入 `L<N+1>` 的字节数减去从 `L<N+1>` 读取的字节数。
* `Moved(GB)` : 在 `L<N>` 到 `L<N+1>` 之间的 compaction 中，从 `L<N>` 移动到 `L<N+1>` 的字节数。注意这里除了会更新 manifest 来表示某个文件原来在 `L<N>` 而现在在 `L<N+1>` 以外不涉及其他 I/O 操作。
* `W-Amp` : （写入 `L<N+1>` 的总字节数） / （从 `L<N>` 读的总字节数）。这是 `L<N>` 和 `L<N+1>` 之间的 compaction 的写放大。
* `Rd(MB/s)` : `L<N>` 到 `L<N+1>` 之间的 compaction 的读取速率，其值等于 `Read(GB) * 1024` / `duration`，`duration` 就是 `Comp(sec)`，不过做了一点处理，就是把 Comp(sec) 的微秒形式 + 1，以防止其作为分母时为 0。（在代码中进行这个时间计算时，单位用的是微秒，Comp(sec) 也只是把微秒换算成秒。上面说的是在原始的微秒值上 + 1，这对结果几乎没有影响。）
* `Wr(MB/s)` : `L<N>` 到 `L<N+1>` 之间的 compaction 的写入速率，其值等于 `Write(GB) * 1024` / `duration`，`duration` 含义同上。
* `Comp(sec)` : `L<N>` 到 `L<N+1>` 之间的总的 compaction 耗时。
* `CompMergeCPU(sec)` : `L<N>` 到 `L<N+1>` 之间的 compaction 占用 CPU 的时间。
* `Comp(cnt)` : `L<N>` 到 `L<N+1>` 之间进行的 compaction 数量。
* `Avg(sec)` : `L<N>` 到 `L<N+1>` 之间，平均每个 compaction 的耗时。
* `KeyIn` : `L<N>` 到 `L<N+1>` 之间 compaction 过程中比较的 key 数量。
* `KeyDrop` : 在 compaction 过程中丢弃（未写入）的 key 数量。
* `Rblob(GB)` : `L<N>` 到 `L<N+1>` 之间的 compaction 从 blob 文件中读取的数据大小。
* `Wblob(GB)` : `L<N>` 到 `L<N+1>` 之间的 compaction 写入 blob 文件的数据大小。

\-

然后我们看第二个表格：

第二个表格只有第一列和上一个表格不同，所有我们这里只说这一列。  
这一列的表头是 `Priority`，也就是优先级，下面有 2 个属性值，`Low` 和 `High`。  
事实上，受限于我们的实例，这个地方总共可以有 4 个属性值，分别为：`Bottom`、`Low`、`High`、和 `User`。  
这个表格展示的就是各个优先级的操作的统计数据，应该不难理解。

\-

关于 Blob 这一行，只有两个内容，一是 Blob 文件的数量，二是 Blob 文件的总大小。  
这里指的是当前数据库中存在的 Blob 文件的数量和总大小。

\-

最后就是这个多行的统计信息了。

下面大部分统计都有两个条目，`cumulative`（除了 `Uptime(secs)` 用的 `total`，含义一样） 和 `interval`。

`cumulative` 表示**从数据库开始运行**到现在的统计值。  
`interval` 表示本次报告距离上次报告的间隔。

这里以 `Uptime(secs)` 和 `Flush(GB)` 举例说明，后面其他的就简单说了。

* `Uptime(secs)` :
    * `total` : 数据库已运行时间（从启动到现在）。
    * `interval` : 本次报告距离上次报告的间隔时间。  
    这里表示当前显示的数据库运行时间，比上次显示的数据库运行时间，多了多少秒。  
    例如上次报告的 `total` 值是 100.3，当前 `total` 值是 102.5，那么 `interval` 就是 2.2。
* `Flush(GB)` : 
    * `cumulative` : 从数据库开始运行到现在，累计完成 flush 的数据大小。注意这里指的是从内存 flush 到 L0 的数据。
    * `interval` : 本次报告距离上次报告的间隔大小。  
    这里表示当前显示的已 flush 数据大小，比上次显示的已 flush 数据大小，多了多少。  
    计算方法同上。

下面就简单说了：

* `AddFile(GB)` : 增加的文件大小。
* `AddFile(Total Files)` : 增加的文件数量。
* `AddFile(L0 Files)` : L0 增加的文件数量。
* `AddFile(Keys)` : 增加的 key 的数量。
* `compaction`（`cumulative` 和 `interval` 在两行）: 
    * `write` : compaction 中写入的数据。有两个值，前者是写入总量，后者是写入速率。
    * `read` : compaction 中读取的数据。有两个值，前者是读取总量，后者是读取速率。
    * `seconds` : compaction 进行的时间。

最后还有个 `Stalls(count)` :

* `Stalls(count)` :
    * `level0_slowdown` : 由于 L0 的文件数达到了 `level0_slowdown_writes_trigger` 导致的写入**放缓**的次数。
    * `level0_slowdown_with_compaction` : 触发 `level0_slowdown` 时（计数 + 1 时），若当前有涉及 level 0 的 compaction 正在进行，则此计数 + 1。
    * `level0_numfiles` : 由于 L0 的文件数达到了 `level0_stop_writes_trigger` 导致的写入**停止**次数。
    * `level0_numfiles_with_compaction` : 触发 `level0_numfiles` 时（计数 + 1 时），若当前有涉及 level 0 的 compaction 正在进行，则此计数 + 1。
    * `stop for pending_compaction_bytes` : 由于待进行 compaction 的字节数达到了 `hard_pending_compaction_bytes_limit` 导致的写入**停止**次数。
    * `slowdown for pending_compaction_bytes` : 由于待进行 compaction 的字节数达到了 `soft_pending_compaction_bytes_limit` 导致的写入**放缓**次数。
    * `memtable_compaction` : 因为所有的 memtable 都满了，flush 过程无法跟上导致的写入**停止**次数。
    * `memtable_slowdown` : 因为所有的 memtable 都快满了，flush 过程无法跟上导致的写入**放缓**次数。
    * `interval n total count` : `n` 是距离上次报告增加的 Stall 总次数。其值就是前面 8 项里除了 `level0_slowdown_with_compaction` 和 `level0_numfiles_with_compaction` 的另外 6 项的“间隔”的和。

对于 `interval n total count`，其值又等于 `io_stalls.total_stop` + `io_stalls.total_slowdown`。  
`io_stalls.total_stop` 是上面 3 种导致写入**停止**的情况次数的和，  
`io_stalls.total_slowdown` 是上面与 3 种导致写入**放缓**的情况次数的和。  
这两个属性我们用 `GetMapProperty()` 获取时才会输出，而 `GetProperty()` 不会输出。



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

这个示例比较长，用来解释说明有点冗余，我们就简单看看就好。  

一开始的标题也写明了，这个属性给出的是一个 “**按 level 划分的文件读取延迟柱状图**”，延迟单位为微秒。  
`default` 同样表示默认列族。

这里选一段来，方便解释，如下：
```
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
```
这是 Level 3 的数据。

`Count` : 表示当前 Level 总共读的次数（上例为 Level 3 总共读了 7024 次）。  
`Average` : 所有的读的平均延迟（上例为 Level 3 的 7024 次读的平均延迟为 0.0209 微秒）。  
`Min`, `Median`, `Max` : 分别表示所有读延迟的最小值、中位数和最大值。  
`Percentiles`: P50, P75, P99, P99.99, P99.99 的延迟。

下面最左边是柱状图的区间，和数学中的括号类似，`(` `)` 表示开区间，`[` `]` 表示闭区间。  
第二列是数量，即当前延迟区间的读数量，总和等于上面的 `Count`。  
第三列是当前延迟区间的读数量占总读数量的比例。  
最后一列是从 0 到当前延迟区间的读书量的总占比。





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

下面是一个现实示例的该输出输出，我们在下面解释。

```
** DB Stats **
Uptime(secs): 5262850.5 total, 8.7 interval
Cumulative writes: 19G writes, 81G keys, 18G commit groups, 1.0 writes per commit group, ingest: 216850.74 GB, 42.19 MB/s
Cumulative WAL: 19G writes, 0 syncs, 19332703393.00 writes per sync, written: 216850.74 GB, 42.19 MB/s
Cumulative stall: 350:12:45.064 H:M:S, 24.0 percent
Interval writes: 31K writes, 134K keys, 31K commit groups, 1.0 writes per commit group, ingest: 364.40 MB, 41.74 MB/s
Interval WAL: 31K writes, 0 syncs, 31934.00 writes per sync, written: 0.36 GB, 41.74 MB/s
Interval stall: 00:00:0.000 H:M:S, 0.0 percent
```

`total` 和 `Cumulative` 都表示从数据库启动开始到现在的累计结果。  
`interval` 表示距离上次报告的间隔。

这样，这个属性输出就只有 4 个值：
* `Uptime(secs)` : 数据库从启动到现在的时间。
* `writes` :
    * `writes` : 写入次数。
    * `keys` : 写入的 key 数量。
    * `commit groups` : 提交写入的组数量。
    * `writes per commit group` : 平均每个组包含的 write 数量。
    * `ingest` : 有两个值，分别表示写入数据库的总数据大小，与平均写入速率。
    * `syncs` : sync 次数。
    * `writes per sync` : 每次 sync 的 write 次数。
    * `written` : 这里有两个值，分别为写入的字节数、写入速率。
* `WAL` : 
    * `writes` : 写入次数。
    * `syncs` : sync 次数。
    * `writes per sync` : 每次 sync 的 write 次数。
    * `written` : 这里有两个值，分别为已写入的总字节数、平均写入速率。
* `stall` : 有两个值。第一个是推迟时间，格式是 `时:分:秒`。第二个 `percent` 是推迟时间占数据库运行时间的百分比。


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
顾名思义，LSM 各个 Level 的统计数据。

下面给出一个真实世界的实例的此属性：

```
Level Files Size(MB)
--------------------
  0        0        0
  1       49     1304
  2       72     1050
  3      291     4280
  4      929    16625
  5      205    29882
  6        0        0
  7        0        0
  8        0        0
  9        0        0
```

很简单，就是每个 Level 的 sst 文件数，以及总大小。

这里注意下是压缩(compaction)后的大小，别的就不多解释了。



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

block cache 的使用统计数据。

因为我写这篇博客的时候还没有用到这个属性，所以这里后面再回来补。



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

尚未 flush 的不可变 memtable 数量。



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

已经 flush 完成的不可变 memtable 数量。



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

值为 0 或 1。

1 表示当前存在待 flush 的 memtable，0 反之。



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

当前正在进行的 flush 数量。



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

值为 0 或 1。

1 表示当前存在至少一个正在等待的 compaction，0 反之。



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

当前正在进行的 compaction 数量。



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

后台累计发生的错误总数。



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

当前活跃的 memtable 的大概大小（字节）。



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

全部 memtabke 总的大概大小（字节）。

包含当前活跃的 memtable，以及未 flush 的不可变 memtable。



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

全部 memtabke 总的大概大小（字节）。

包含当前活跃的 memtable、未 flush 的不可变 memtable，以及 pinned 的不可变 memtable。

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

当前活跃的 memtable 中的条目总数。



## 22. kNumEntriesImmMemTables

**定义**

```cpp
static const std::string kNumEntriesImmMemTables;
```

**属性**

```
rocksdb.num-entries-imm-mem-tables
```

**含义**

未 flush 的不可变 memtable 中的条目数量。



## 23. kNumDeletesActiveMemTable

**定义**

```cpp
static const std::string kNumDeletesActiveMemTable;
```

**属性**

```
rocksdb.num-deletes-active-mem-table
```

**含义**


当前活跃的 memtable 中删除的条目总数。



## 24. kNumDeletesImmMemTables

**定义**

```cpp
static const std::string kNumDeletesImmMemTables;
```

**属性**

```
rocksdb.num-deletes-imm-mem-tables
```

**含义**

未 flush 的不可变 memtable 中删除的条目总数。



## 25. kEstimateNumKeys

**定义**

```cpp
static const std::string kEstimateNumKeys;
```

**属性**

```
rocksdb.estimate-num-keys
```

**含义**

数据库中大概的 key 数量。

包含活跃的 memtable，未 flush 的不可变 memtable，以及存储空间（已 flush 的 sst 文件）中的。

是大致数量而不是精确数量，原因是有一些 key 的删除、覆盖操作难以及时统计。



## 26. kEstimateTableReadersMem

**定义**

```cpp
static const std::string kEstimateTableReadersMem;
```

**属性**

```
rocksdb.estimate-table-readers-mem
```

**含义**

用于读取 sst 而使用的大概内存大小。

不包括 block cache 使用的内存（例如 filter 和 index blocks）。



## 27. kIsFileDeletionsEnabled

**定义**

```cpp
static const std::string kIsFileDeletionsEnabled;
```

**属性**

```
rocksdb.is-file-deletions-enabled
```

**含义**

表示是否允许删除过期、废弃的文件。

0 表示允许，非 0 表示不允许。

这里的内部实现是有一个计数器 `disable_delete_obsolete_files_`，用来累计禁用次数，默认为 1。

方法 `DBImpl::DisableFileDeletions()` 给此计数器加 1。

方法 `DBImpl::EnableFileDeletions(bool force)`：  
参数 `force` 若为 true 则直接将计数器归零。  
参数 `force` 若为 false 则给计数器减 1，最多把计数器减到 0。

## 28. kNumSnapshots

**定义**

```cpp
static const std::string kNumSnapshots;
```

**属性**

```
rocksdb.num-snapshots
```

**含义**

数据库中未释放的快照数量。



## 29. kOldestSnapshotTime

**定义**

```cpp
static const std::string kOldestSnapshotTime;
```

**属性**

```
rocksdb.oldest-snapshot-time
```

**含义**

最旧的未释放的快照的 unix 时间戳。



## 30. kOldestSnapshotSequence

**定义**

```cpp
static const std::string kOldestSnapshotSequence;
```

**属性**

```
rocksdb.oldest-snapshot-sequence
```

**含义**

最旧的未释放的快照的序列号。



## 31. kNumLiveVersions

**定义**

```cpp
static const std::string kNumLiveVersions;
```

**属性**

```
rocksdb.num-live-versions
```

**含义**

实时版本(live version)数量。

`Version` 是一个内部数据结构，详见 `version_set.h`。

更多实时版本通常意味着更多的 SST 文件不会被迭代器或未完成的 compactions 删除。



## 32. kCurrentSuperVersionNumber

**定义**

```cpp
static const std::string kCurrentSuperVersionNumber;
```

**属性**

```
rocksdb.current-super-version-number
```

**含义**


返回当前 LSM 版本的编号。它是一个 `uint64_t` 整数，在 LSM 树发生任何更改后递增。

重新启动数据库后，不会保留该号码，将再次从 0 开始。


## 33. kEstimateLiveDataSize

**定义**

```cpp
static const std::string kEstimateLiveDataSize;
```

**属性**

```
rocksdb.estimate-live-data-size
```

**含义**


以字节为单位返回实时数据量的估计值。

对于 BlobDB ，它还包括版本的 blob 文件中实时的字节确切值。


## 34. kMinLogNumberToKeep

**定义**

```cpp
static const std::string kMinLogNumberToKeep;
```

**属性**

```
rocksdb.min-log-number-to-keep
```

**含义**

应当保留日志文件的最小数量。



## 35. kMinObsoleteSstNumberToKeep

**定义**

```cpp
static const std::string kMinObsoleteSstNumberToKeep;
```

**属性**

```
rocksdb.min-obsolete-sst-number-to-keep
```

**含义**

返回要保留的过时 SST 的最小文件号。

如果可以删除所有过时的文件，将返回 `uint64_t` 的最大值（即 18446744073709551615）。



## 36. kTotalSstFilesSize

**定义**

```cpp
static const std::string kTotalSstFilesSize;
```

**属性**

```
rocksdb.total-sst-files-size
```

**含义**

所有 sst 文件的总大小（字节）。

注意，如果 sst 文件太多，输出这个属性值可能会降低数据库的查询速度。



## 37. kLiveSstFilesSize

**定义**

```cpp
static const std::string kLiveSstFilesSize;
```

**属性**

```
rocksdb.live-sst-files-size
```

**含义**

返回属于最新 LSM 树的所有 SST 文件的总大小（字节）。

这里的“最新”，我的理解是 LSM 的最新状态。


## 38. kLiveSstFilesSizeAtTemperature

**定义**

```cpp
static const std::string kLiveSstFilesSizeAtTemperature;
```

**属性**

```
rocksdb.live_sst_files_size_at_temperature
```

**含义**

所有特定文件温度(temperature)下的 SST 文件的总大小（字节）

rocksdb 中定义了一个温度类 `Temperature`，用以描述文件的温度：
```cpp
enum class Temperature : uint8_t {
  kUnknown = 0,
  kHot = 0x04,
  kWarm = 0x08,
  kCold = 0x0C,
};
```
rocksdb 把文件的温度信息传递给文件系统，使得文件获得不同的放置或编码。

这几个温度的编号中间保留了一些数字，留给以后可能的扩展。

另外，虽然这个属性比较好解释，<font color="red">但我没在代码中发现任何的实际调用，该属性的输出始终为空。</font>

## 39. kBaseLevel

**定义**

```cpp
static const std::string kBaseLevel;
```

**属性**

```
rocksdb.base-level
```

**含义**

L0 数据将被 compact 到的 level 数。

这个值默认是 1。

如果当前不是 level-compaction，这个值是 -1，因为不适用。

这个值的含义是 L0 的数据会被直接 compact 到哪个层，一般来说就是 L1，也可以设的更高。

举个例子，如果该值为 3，那么 L0 的数据就会被直接 compact 到 L3，L1 和 L2 就是空的。


## 40. kEstimatePendingCompactionBytes

**定义**

```cpp
static const std::string kEstimatePendingCompactionBytes;
```

**属性**

```
rocksdb.estimate-pending-compaction-bytes
```

**含义**

返回 compaction 需要重写的估计总字节数，以使所有 level 降低到目标大小以下。

对 level-based 以外的其他压缩无效。



## 41. kAggregatedTableProperties

**定义**

```cpp
static const std::string kAggregatedTableProperties;
```

**属性**

```
rocksdb.aggregated-table-properties
```

**含义**


返回目标列族的聚合表属性的字符串或映射表示。

仅包含对聚合有意义的属性。
 
下面是一个使用 `GetProperty()` 输出的例子（字符串），基本都可以通过名字来理解。  
其实用 `GetMapProperty()` 更合适，可以输出更好理解的键值对形式（我懒了）。


```
# data blocks=770855; # entries=109566553; # deletions=0; # merge operands=0; # range deletions=0; raw key size=2557375015; raw average key size=23.340837; raw value size=1680842603; raw average value size=15.340837; data block size=3134714864; index block size (user-key? 62, delta-value? 62)=21101981; filter block size=0; # entries for filter=0; (estimated) table size=3155816845; filter policy name=N/A; prefix extractor name=N/A; column family ID=N/A; column family name=N/A; comparator name=N/A; merge operator name=N/A; property collectors names=N/A; SST file compression algo=N/A; SST file compression options=N/A; creation time=0; time stamp of earliest key=0; file creation time=0; slow compression estimated data size=0; fast compression estimated data size=0; DB identity=; DB session identity=; DB host id=; original file number=0; 
```

## 42. kAggregatedTablePropertiesAtLevel

**定义**

```cpp
static const std::string kAggregatedTablePropertiesAtLevel;
```

**属性**

```
rocksdb.aggregated-table-properties-at-level<N>
```

**含义**

与上一个类似，不过是按 Level 输出的。



## 43. kActualDelayedWriteRate

**定义**

```cpp
static const std::string kActualDelayedWriteRate;
```

**属性**

```
rocksdb.actual-delayed-write-rate
```

**含义**

当前实际的延迟写速率。

0 表示没有延迟。



## 44. kIsWriteStopped

**定义**

```cpp
static const std::string kIsWriteStopped;
```

**属性**

```
rocksdb.is-write-stopped
```

**含义**


表示写入是否已经停止。

1 表示已停止，0 反之。


## 45. kEstimateOldestKeyTime

**定义**

```cpp
static const std::string kEstimateOldestKeyTime;
```

**属性**

```
rocksdb.estimate-oldest-key-time
```

**含义**


数据库中最旧的 key 的时间戳估计值。

目前只对 `compaction_options_fifo.allow_compaction` 为 `false` 的 FIFO compaction 有效。


## 46. kBlockCacheCapacity

**定义**

```cpp
static const std::string kBlockCacheCapacity;
```

**属性**

```
rocksdb.block-cache-capacity
```

**含义**

block cache 的容量。



## 47. kBlockCacheUsage

**定义**

```cpp
static const std::string kBlockCacheUsage;
```

**属性**

```
rocksdb.block-cache-usage
```

**含义**

存在于 block cache 中的条目占用的内存大小。



## 48. kBlockCachePinnedUsage

**定义**

```cpp
static const std::string kBlockCachePinnedUsage;
```

**属性**

```
rocksdb.block-cache-pinned-usage
```

**含义**

被 pinned 的条目占用的内存大小。



## 49. kOptionsStatistics

**定义**

```cpp
static const std::string kOptionsStatistics;
```

**属性**

```
rocksdb.options-statistics
```

**含义**

一个多行 strings，即 `options.statistics` 的内容。



## 50. kNumBlobFiles

**定义**

```cpp
static const std::string kNumBlobFiles;
```

**属性**

```
rocksdb.num-blob-files
```

**含义**

当前版本中 blob 文件的数量。



## 51. kBlobStats

**定义**

```cpp
static const std::string kBlobStats;
```

**属性**

```
rocksdb.blob-stats
```

**含义**

当前版本中所有 blob 文件总的数量和总的大小，以及 blob 文件总的垃圾大小。

下面是一个示例值：
```
Number of blob files: 0
Total size of blob files: 0
Total size of garbage in blob files: 0
```



## 52. kTotalBlobFileSize

**定义**

```cpp
static const std::string kTotalBlobFileSize;
```

**属性**

```
rocksdb.total-blob-file-size
```

**含义**

所有的版本中的 blob 文件的总大小。




## 53. kLiveBlobFileSize

**定义**

```cpp
static const std::string kLiveBlobFileSize;
```

**属性**

```
rocksdb.live-blob-file-size
```

**含义**

当前版本的 blob 文件的总大小。

