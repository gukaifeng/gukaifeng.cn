---
title: RocksDB -- MANIFEST 浅析
date: 2021-12-06 17:57:25
updated: 2021-12-07 19:22:25
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



## 1. 什么是 MANIFEST

RocksDB 是文件系统与存储介质无关的。

文件系统的操作不具有原子性，可能会由于意外情况，导致 RocksDB 出现不一致的问题。

即使打开了日志记录，文件系统也不能保证不洁重启的一致性。

POSIX 文件系统也不支持原子的操作批处理。

因此，在 RocksDB 重新启动时，仅仅依赖 RocksDB 数据存储文件中的元数据来重建最后的一致状态是不现实的。

RocksDB 拥有一个内置的机制来克服上述 POSIX 文件系统的限制，即 MANIFEST。

<!--more-->

RocksDB 把使 RocksDB 状态改变的事务日志存在 MANIFEST 文件中。

RocksDB 重启时，会通过 MANIFEST 中的内容，将 RocksDB 恢复到最后已知的一致性状态。

MANIFEST 中采用版本编辑记录(**Version Edit Records**)。





## 2. 相关术语

* `MANIFEST`: RocksDB 状态变化的事务日志。
* `Manifest log`: 单个包含 RocksDB 状态快照/编辑的日志文件。
* `CURRENT`: 最新的 manifest log。





## 3. MANIFEST 如何工作



MANIFEST 中包含一个 manifest log 和一个指向 CURRENT 的指针。

Manifest log 是名为 `MANIFEST-序号` 的滚动日志文件，例如 `MANIFEST-000094`，序列号始终是递增的。

CURRENT 即最新的 manifest log。

当系统启动或重启时，最新的 manifest log 中包含了 RocksDB 最新的一致状态。任何后续的 RocksDB 状态的修改都会被记录到此 manifest log 中。

当这个 manifest log 超过一个指定的大小时，将使用 RocksDB 状态的快照创建一个新的 manifest log。随后更新指向 CURRENT 的指针（修改为指向刚创建的新 manifest log，这个更新）并同步文件系统（指的是 RocksDB 在磁盘上的文件）。在文件系统同步到 CURRENT 的状态后（即最新的的一致状态），旧的 manifest log 文件将被删除。

-

下面最后再整理一下：

```
MANIFEST = { CURRENT, MANIFEST-<seq-no>* } 
CURRENT = 指向最新的 manifest log 文件的指针
MANIFEST-<seq no> = 包含一个 RocksDB 状态的快照，以及后续的修改记录
```

-

MANIFEST 也是 RocksDB 架构的重要一环，如果你还不了解 RocksDB 的架构，可以看看这个：[RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/)。



## 4. 查看 MANIFEST 中的内容

RocksDB 提供了工具 ldb，其代码位于 RocksDB 源码中的 tools 目录下。

我们可以使用 ldb 工具查看 manifest log 文件中的具体内容。

### 4.1. 编译 ldb

在 rocksdb 源码根目录下（不是 tools 目录），执行下面的命令：

```shell
DEBUG_LEVEL=0 make ldb
```

由于我们的 rocksdb 使用的是 release 版本，所以这里的 ldb 也要是 release 版本，不加 `DEBUG_LEVEL=0` 的话，编译的是 debug 版本的 ldb。

ldb 是编译为 debug 版本还是 release 版本要看你的 rocksdb 是编译的哪个版本，要与之一样。

编译完成后会在 rocksdb 源码根目录生成一个名为 `ldb` 的可执行文件。



### 4.2. 使用 ldb 查看 manifest log

使用 ldb 查看 manifest log 的方法如下：

```shell
/path/to/ldb manifest_dump --path="manifest_log_path"
```

例如：

```shell
./rocksdb-6.25.3/ldb manifest_dump --path="./testdb/MANIFEST-000094"
```

一个空的 manifest log 长这样：

```
--------------- Column family "default"  (ID 0) --------------
log number: 0
comparator: leveldb.BytewiseComparator
--- level 0 --- version# 1 ---
--- level 1 --- version# 1 ---
--- level 2 --- version# 1 ---
--- level 3 --- version# 1 ---
--- level 4 --- version# 1 ---
--- level 5 --- version# 1 ---
--- level 6 --- version# 1 ---
--- level 7 --- version# 1 ---
--- level 8 --- version# 1 ---
--- level 9 --- version# 1 ---
--- level 10 --- version# 1 ---
--- level 11 --- version# 1 ---
--- level 12 --- version# 1 ---
--- level 13 --- version# 1 ---
--- level 14 --- version# 1 ---
--- level 15 --- version# 1 ---
--- level 16 --- version# 1 ---
--- level 17 --- version# 1 ---
--- level 18 --- version# 1 ---
--- level 19 --- version# 1 ---
--- level 20 --- version# 1 ---
--- level 21 --- version# 1 ---
--- level 22 --- version# 1 ---
--- level 23 --- version# 1 ---
--- level 24 --- version# 1 ---
--- level 25 --- version# 1 ---
--- level 26 --- version# 1 ---
--- level 27 --- version# 1 ---
--- level 28 --- version# 1 ---
--- level 29 --- version# 1 ---
--- level 30 --- version# 1 ---
--- level 31 --- version# 1 ---
--- level 32 --- version# 1 ---
--- level 33 --- version# 1 ---
--- level 34 --- version# 1 ---
--- level 35 --- version# 1 ---
--- level 36 --- version# 1 ---
--- level 37 --- version# 1 ---
--- level 38 --- version# 1 ---
--- level 39 --- version# 1 ---
--- level 40 --- version# 1 ---
--- level 41 --- version# 1 ---
--- level 42 --- version# 1 ---
--- level 43 --- version# 1 ---
--- level 44 --- version# 1 ---
--- level 45 --- version# 1 ---
--- level 46 --- version# 1 ---
--- level 47 --- version# 1 ---
--- level 48 --- version# 1 ---
--- level 49 --- version# 1 ---
--- level 50 --- version# 1 ---
--- level 51 --- version# 1 ---
--- level 52 --- version# 1 ---
--- level 53 --- version# 1 ---
--- level 54 --- version# 1 ---
--- level 55 --- version# 1 ---
--- level 56 --- version# 1 ---
--- level 57 --- version# 1 ---
--- level 58 --- version# 1 ---
--- level 59 --- version# 1 ---
--- level 60 --- version# 1 ---
--- level 61 --- version# 1 ---
--- level 62 --- version# 1 ---
--- level 63 --- version# 1 ---
next_file_number 6 last_sequence 0  prev_log_number 0 max_column_family 0 min_log_number_to_keep 0
```





### 4.3. 示例

> 下面演示的示例是非常非常简单的，修改 RocksDB 的操作多种多样，不同操作的 MANIFEST 记录格式也不一样。关于 MANIFEST 记录格式的分析，请查看 [RocksDB -- MANIFEST 记录格式](https://gukaifeng.cn/posts/rocksdb-manifest-ji-lu-ge-shi/) 这篇文章。

下面我们执行一段简易的代码，然后查看其生成的 manifest log。

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

    std::cout << "Enter your keys/values, format: \"[key] [value]\"." << std::endl;
    std::cout << "Enter \"0 0\" to quit." << std::endl;

    std::string k, v;
    while (std::cin >> k >> v && !(k == "0" && v == "0")) {
        status = db->Put(rocksdb::WriteOptions(), k, v);
        if (!status.ok()) {
            std::cout << status.ToString() << std::endl;
            break;
        } else {
            std::cout << "Put key=\"" << k << "\" val=\"" << v << "\" successfully." << std::endl;
        }

    }
    std::cout << "Quit" << std::endl;

    db->Close();
    delete db;

    return 0;
}
```

这段代码比较简陋，大家先忽略=。=

这段代码使得我们循环插入一些 kv 对，我们假定 `./testdb` 目前不存在，执行上述代码。

```
Enter your keys/values, format: "[key] [value]".
Enter "0 0" to quit.
name gukaifeng
Put key="name" val="gukaifeng" successfully.
age 24
Put key="age" val="24" successfully.
blog gukaifeng.cn
Put key="blog" val="gukaifeng.cn" successfully.
0 0
Quit
```

我们输入了三个字段，`name:gukaifeng`、`age:24` 和 `blog:gukaifeng.cn`，然后退出了程序，关闭了数据库。

再次运行上面的代码，我们什么都不用做，运行就好：

```
Enter your keys/values, format: "[key] [value]".
Enter "0 0" to quit.
```

这时，我们用之前的方法查看最新的 manifest log，我这里是 `MANIFEST-000010`，其内容如下：

```
--------------- Column family "default"  (ID 0) --------------
log number: 6
comparator: leveldb.BytewiseComparator
--- level 0 --- version# 1 ---
 9:1026[1 .. 3]['age' seq:2, type:1 .. 'name' seq:1, type:1]
--- level 1 --- version# 1 ---
--- level 2 --- version# 1 ---
--- level 3 --- version# 1 ---
--- level 4 --- version# 1 ---
--- level 5 --- version# 1 ---
--- level 6 --- version# 1 ---
--- level 7 --- version# 1 ---
--- level 8 --- version# 1 ---
--- level 9 --- version# 1 ---
--- level 10 --- version# 1 ---
--- level 11 --- version# 1 ---
--- level 12 --- version# 1 ---
--- level 13 --- version# 1 ---
--- level 14 --- version# 1 ---
--- level 15 --- version# 1 ---
--- level 16 --- version# 1 ---
--- level 17 --- version# 1 ---
--- level 18 --- version# 1 ---
--- level 19 --- version# 1 ---
--- level 20 --- version# 1 ---
--- level 21 --- version# 1 ---
--- level 22 --- version# 1 ---
--- level 23 --- version# 1 ---
--- level 24 --- version# 1 ---
--- level 25 --- version# 1 ---
--- level 26 --- version# 1 ---
--- level 27 --- version# 1 ---
--- level 28 --- version# 1 ---
--- level 29 --- version# 1 ---
--- level 30 --- version# 1 ---
--- level 31 --- version# 1 ---
--- level 32 --- version# 1 ---
--- level 33 --- version# 1 ---
--- level 34 --- version# 1 ---
--- level 35 --- version# 1 ---
--- level 36 --- version# 1 ---
--- level 37 --- version# 1 ---
--- level 38 --- version# 1 ---
--- level 39 --- version# 1 ---
--- level 40 --- version# 1 ---
--- level 41 --- version# 1 ---
--- level 42 --- version# 1 ---
--- level 43 --- version# 1 ---
--- level 44 --- version# 1 ---
--- level 45 --- version# 1 ---
--- level 46 --- version# 1 ---
--- level 47 --- version# 1 ---
--- level 48 --- version# 1 ---
--- level 49 --- version# 1 ---
--- level 50 --- version# 1 ---
--- level 51 --- version# 1 ---
--- level 52 --- version# 1 ---
--- level 53 --- version# 1 ---
--- level 54 --- version# 1 ---
--- level 55 --- version# 1 ---
--- level 56 --- version# 1 ---
--- level 57 --- version# 1 ---
--- level 58 --- version# 1 ---
--- level 59 --- version# 1 ---
--- level 60 --- version# 1 ---
--- level 61 --- version# 1 ---
--- level 62 --- version# 1 ---
--- level 63 --- version# 1 ---
next_file_number 12 last_sequence 3  prev_log_number 0 max_column_family 0 min_log_number_to_keep 0
```

这里已经可以看到我们之前写入的大致信息了。

