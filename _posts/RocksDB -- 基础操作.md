---
title: RocksDB -- 基础操作
date: 2021-10-19 00:14:57
updated: 2021-10-19 00:14:57
categories: [技术杂谈]
tags: [RocksDB, 数据库]
toc: true
---



这篇文章介绍几个 RocksDB 中基础的、常见的操作。

**请注意，这篇文章中对于某些功能的高级用法并没有阐述，仅介绍基础用法。**要了解更多的内容需要去查看介绍特定功能的文章或官方 Wiki。



## 1. RocksDB 启动！

一个 RocksDB 数据库其实就是文件系统中的一个目录，RocksDB 数据库中的所有内容都存在这个目录中。

现在从下面的代码开始来介绍 RocksDB 数据库的几个基础操作：

<!--more-->

```cpp
#include <cassert>
#include "rocksdb/db.h"

int main(int argc, char* argv[]) {
  rocksdb::DB* db;
  rocksdb::Options options;
  options.create_if_missing = true;
  rocksdb::Status status =
      rocksdb::DB::Open(options, "./testdb", &db);
  assert(status.ok());
  delete db;

  return 0;
}
```

* `rocksdb::DB* db;`:  `db` 是一个 RocksDB 中的数据库指针，RocksDB 通过其来操作数据库；

* `rocksdb::Options options;`: 这里的 `options` 是打开 RocksDB 的选项，会和 `db` 一起传给打开数据库函数 `rocksdb::DB::Open()`；

* `options.create_if_missing = true;`: 这句话表示，如果数据库不存在，则创建一个新的；  
    与之类似的选项还有一个 `options.error_if_exists = true;` 表示如果数据库已经存在，则报错。`create_if_missing` 和 `error_if_exists ` 这两个选项可以一起设定，就表示要新建一个数据库，如果数据目录已经存在的话就报错。

* `rocksdb::Status` 是 RocksDB 库中绝大多数操作函数的返回值类型，用以说明操作是否顺利完成。





### 1.1. 打开数据库

函数原型如下：

```cpp
Status CompactedDBImpl::Open(const Options& options,
                             const std::string& dbname, DB** dbptr)
```

* `const Options& options`：打开 RocksDB 数据库的选项；
* `const std::string& dbname,`：待打开的 RocksDB 数据库目录；
* `DB** dbptr`：RocksDB 数据库指针，RocksDB 是通过此指针操作数据库的。







### 1.2. RocksDB 选项

我们上面的代码中已经演示过 RocksDB 选项。

```
rocksdb::Options options;
options.create_if_missing = true;
```

这两行代码，第一行定义了一个选项类的对象，第二行通过这个对象设置选项，最终这个选项对象会传递给打开 RocksDB 的函数。这里不深入介绍，先简单知道什么是 RocksDB 的选项就可以。

-

上面的方法，是在打开数据库之前设定的选项，还有一个方法可以在数据库运行时动态修改选项。

像下面这样，通过数据库指针  `db` 来修改：

```cpp
rocksdb::Status s;
s = db->SetOptions({{"write_buffer_size", "131072"}});
assert(s.ok());
s = db->SetDBOptions({{"max_background_flushes", "2"}});
assert(s.ok());
```





---

我们知道，RocksDB 是基于 LevelDB 开发的。如果你正在将代码从 LevelDB 移植到 RocksDB，你可以使用`rocksdb::LevelDBOptions` 将你的 `leveldb::Options` 对象转换为 `rocksdb::Options` 对象，该对象具有与`leveldb::Options` 相同的功能。下面是代码示例：

```cpp
#include "rocksdb/utilities/leveldb_options.h"

rocksdb::LevelDBOptions leveldb_options;
leveldb_options.option1 = value1;
leveldb_options.option2 = value2;
...
rocksdb::Options options = rocksdb::ConvertOptions(leveldb_options);
```





### 1.3. `Status` 状态

我们上面的代码中已经演示过这种类型。这种类型的值由可能遇到错误的 RocksDB 中的大多数函数返回。

你可以检查函数的结果是否正确，并打印一个相关的错误消息：

```cpp
rocksdb::Status s = ...;
if (!s.ok()) cerr << s.ToString() << endl;
```



### 1.4. 关闭数据库

RocksDB 中关闭数据库主要有两种方式：

1. 直接释放 `db` 指针指向的对象；
2. 先调用 `DB::Close()` 函数，然后再释放 `db` 指针指向的对象。

第一种方法，直接删除 `db` 指针（释放其指向的对象）会释放数据库打开时所有持有的资源。但是，如果在释放资源时遇到错误（例如在关闭 `info_log`  文件时出现错误），这些资源将丢失。

第二种方法先调用 `DB::Close()`，然后删除数据库对象。`DB::Close()` 的好处是其会返回 `Status` 可以用来检查关闭数据库过程中是否有错误。不过是否有错误，最终的结果都和第一种方法一样，会释放掉所有资源，出错的资源一样会丢失。注意，如果返回状态为 `Aborted()`，则关闭失败，因为系统中存在未释放的快照，在这种情况下，用户可以释放未释放的快照，并再次尝试，

简单的说，第一种方法直接关闭数据库，第二种方法可以检查下关闭数据的时候有没有出现什么错误。

下面简单通过代码说下：

```cpp
// 第 1 种关闭 RocksDB 数据库的方法
... open the db as described above ...
... do something with db ...
delete db;
```



```cpp
// 第 2 种关闭 RocksDB 数据库的方法
... open the db as described above ...
... do something with db ...
Status s = db->Close();
... log status ...
delete db;
```





## 2. RocksDB 基本操作

RocksDB 提供了四个数据库基本操作，分别是  `Put`、`Delete`、`Get` 和 `MultiGet`。

* `Put` 往数据库中写入一条数据；
* `Delete` 从数据库中删除一条数据；
* `Get` 从数据库中读取一条数据；
* `MultiGet` 从数据库中读取多个数据。



当然，**RocksDB 中提供的操作远远不止这四个，只是这四个是最基本的。**

下面先逐一介绍这四个基本操作，**注意这四个操作函数均有重载更多的用法，这里只写了最基础的。**

### 2.1. `Put`

```cpp
virtual Status Put(const WriteOptions& options, const Slice& key,
                   const Slice& value);
```

* `option`：写选项；
* `key`：待写入的键；
* `value`：待写入键的值。



### 2.2. `Delete`

```cpp
virtual Status Delete(const WriteOptions& options, const Slice& key);
```

* `option`：写选项；
* `key`：要删除的键。



### 2.3. `Get`

```cpp
virtual Status Get(const ReadOptions& options, const Slice& key,
                   std::string* value);
```

* `option`：读选项；
* `key`：要读的键；
* `value`：把要读键的值存在 `value` 中。



### 2.4. `MultiGet`

`MultiGet` 有两种基础的变体（当然实际还有更多种其他用法）：

1\. 从单个列族中以更高效的方式读取多个键，即比循环调用 `Get `更快；

2\. 读跨多个列族一致的键。

这两种变体官方提供了两个示例，分别如下；

```cpp
std::vector<Slice> keys;
std::vector<PinnableSlice> values;
std::vector<Status> statuses;

for ... {
  keys.emplace_back(key);
}
values.resize(keys.size());
statuses.resize(keys.size());

db->MultiGet(ReadOptions(), cf, keys.size(), keys.data(), values.data(), statuses.data());
```

```cpp
std::vector<ColumnFamilyHandle*> column_families;
std::vector<Slice> keys;
std::vector<std::string> values;

for ... {
  keys.emplace_back(key);
  column_families.emplace_back(column_family);
}
values.resize(keys.size());

std::vector<Status> statuses = db->MultiGet(ReadOptions(), column_families, keys, values);
```

为了避免内存分配的开销，上面的 `keys`、`values` 和 `statuses` 可以是 `std::array` 类型，也可以是任何提供连续存储的类型。

这里就不对 `MultiGet` 做更进一步的解释了。

有关使用 `MultiGet` 的性能优势的更深入讨论看 [MultiGet Performance](https://github.com/facebook/rocksdb/wiki/MultiGet-Performance) 这篇文章。





### 2.5. 示例

```cpp
#include <iostream>
#include "rocksdb/db.h"

int main(int argc, char* argv[]) {
  rocksdb::DB* db;
  rocksdb::Options options;
  options.create_if_missing = true;
  rocksdb::Status s =
      rocksdb::DB::Open(options, "./testdb", &db);

  std::string k1 = "name", k2 = "age";
  std::string v1 = "gukaifeng", v2 = "24";

  if (s.ok()) s = db->Put(rocksdb::WriteOptions(), k1, v1);  // 写入 "name": "gukaifeng"
  if (s.ok()) s = db->Put(rocksdb::WriteOptions(), k2, v2);  // 写入 "age": "24"
  
  std::string value;
  if (s.ok()) s = db->Get(rocksdb::ReadOptions(), k1, &value);  // 把键 name 的值 "gukaifeng" 写入 value
  std::cout << "value = " << value << std::endl;  // 输出 value，此时应为 "gukaifeng"
  
  if (s.ok()) s = db->Delete(rocksdb::WriteOptions(), k2);  // 删除键 "age"
  if (s.ok()) s = db->Get(rocksdb::ReadOptions(), k2, &value);  // 尝试把键 "age" 的值 "24" 写入 value，这里 "age" 已经被删除了，这里的 s.ok() 应当为 false
  if (s.ok()) std::cout << "value is " << value << std::endl;  // 这里 s.ok() 为 false，这句话不会执行
  else std::cout << "s.ToString() is "<< s.ToString() << std::endl;  // 这里会执行，输出异常信息

  s = db->Close();
  delete db;

  return 0;
}
```

输出

```
value = gukaifeng
s.ToString() is NotFound:
```



### 2.6. 扩展 `PinnableSlice`

这里简单介绍一下 `PinnableSlice` 。

每个 `Get` 至少会有一个值从源 memcpy 到字符串。

如果源在块缓存中，你可以使用 `PinnableSlice` 来避免额外的复制。

在这里 `PinnableSlice`  的用法和 `Slice` 或 `std::string` 没什么不同。

```cpp
PinnableSlice pinnable_val;
rocksdb::Status s = db->Get(rocksdb::ReadOptions(), key1, &pinnable_val);
```

但是要注意，一旦 `pinnable_val` 被析构或对其调用 `::Reset`，源将被释放。点击[这里](http://rocksdb.org/blog/2017/08/24/pinnableslice.html)了解更多内容。



