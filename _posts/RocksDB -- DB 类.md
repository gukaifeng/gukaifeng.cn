---
title: RocksDB -- DB 类
date: 2021-11-26 17:16:14
updated: 2021-11-26 17:16:14
categories: [数据库]
tags: [RocksDB,数据库]
toc: true
---



## 1. 什么是 DB 类

DB 类是 RocksDB 中一个非常基础且常用的类，我们对 RocksDB 数据库的大部分操作，都是通过 DB 对象来执行的。



DB 是一个持久的、版本化的、从键到值的有序映射。

DB 对于多个线程的并发访问是安全的，而不需要任何外部同步。

**DB 是一个抽象基类**，它有一个**主实现(DBImpl)**和许多包装器实现。

在 DB 类的定义中，有一部分方法是 `static` 的，其余都是 `virtual` 的，`virtual` 的方法有一些在 DB 类中有实现，有一些只有声明，需要派生类去实现。DB 类的定义中，还有一些常量定义，常量定义都是 `static` 的。

DB 类的定义中，所有的内容，都是 public 的。

DB 类定义在头文件 `db.h` 中，[点此查看](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L147-L1754)。

我们先在此文章中介绍 DB 抽象类的相关内容，了解完 DB 类，我们就对 RocksDB 的基本操作有一定了解了。

<!--more-->

这篇文章不会讲大部分方法的具体实现，少部分比较简单的会顺便说一下，这篇文章主要还是说一说 DB 类里都有哪些东西。



## 2. 构造与析构

```cpp
// Abstract class ctor
DB() {}
// No copying allowed
DB(const DB&) = delete;
void operator=(const DB&) = delete;

virtual ~DB();
```

上面就是 DB 类中构造与析构相关的全部代码。

DB 类定义了一个空的默认构造函数，禁用了拷贝和赋值构造，析构函数是 virtual 的，需要派生类来实现。

因为这篇文章只讲 DB 类，所以派生类的相关实现暂且不谈。

## 3. 打开数据库

### 3.1. 以读写模式打开

```cpp
static Status Open(const Options& options, const std::string& name,
                   DB** dbptr);
```

* `option`: 打开数据库时的选项；
* `name`: 待打开数据库的路径；
* `dbptr`: 用来操作打开数据库的指针。

这个方法，以**读写**模式打开数据库，其把一个在**堆**中分配的数据库的指针存储在 `*dbptr` 中，后面所有的打开方法同理。

如果成功打开数据库，返回的 Status 为 OK。

如果失败，则 Status 中不是 OK，并且 `dbptr` 的值为 nullptr。

失败的情况可能有多种，例如 `name` 数据库已经被另一个 DB 对象以读写模式打开，那么我们肯定就不能再打开了，这个已经被另一个 DB 对象以读写模式打开的判定依赖于 `options.env->LockFile()`，也就是说，如果你自定义了 `env`，这个判定可能不会起作用。

当我们不再需要操作数据库的时候，切记要释放 `*dbptr`（不是 `dbptr`），后面所有的打开方法同理。





### 3.2. 以只读模式打开

```cpp
static Status OpenForReadOnly(const Options& options, const std::string& name,
                              DB** dbptr,
                              bool error_if_wal_file_exists = false);
```

* `option`: 打开数据库时的选项；
* `name`: 待打开数据库的路径；
* `dbptr`: 用来操作打开数据库的指针；
* `error_if_wal_file_exists`: 顾名思义，当 [WAL](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log) 文件存在时是否报错。

以**只读**模式打开数据库。所有修改数据的 DB 接口，都会返回错误。

以只读模式打开数据库时，不会发生压缩(compactions)。

一个 RocksDB 数据库，可以同时被任意多个用户以只读模式打开。

如果一个 RocksDB 数据库同时被两个（或多个）用户以只读和读写模式打开，那么只读实例会存在未定义的行为（虽然经常可以成功，如果快速关闭），读写实例不受影响。

此方法在 `ROCKSDB_LITE` 中不支持，此时 `OpenForReadOnly()` 函数将返回`Status::NotSupported`。

> RocksDB Lite是一个专注于移动用例的项目，它不需要我们为服务器工作负载构建很多华丽的东西，而且它们对二进制大小非常敏感。因此，RocksDB 添加了一个编译标志ROCKSDB_LITE，它可以注释掉很多不必要的代码，并保持二进制精简。

### 3.3. 以只读模式打开（指定列族）

```cpp
static Status OpenForReadOnly(
  const DBOptions& db_options, const std::string& name,
  const std::vector<ColumnFamilyDescriptor>& column_families,
  std::vector<ColumnFamilyHandle*>* handles, DB** dbptr,
  bool error_if_wal_file_exists = false);
```

* `db_options`: 打开数据库时的选项；
* `name`: 待打开数据库的路径；
* `column_families`: 指定列族的描述符数组；
* `handles`: 指定列族的句柄数组；
* `dbptr`: 用来操作打开数据库的指针；
* `error_if_wal_file_exists`: 顾名思义，当 [WAL](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log) 文件存在时是否报错。

此方法大部分内容特性与 3.2 中的一致：

>以**只读**模式打开数据库。所有修改数据的 DB 接口，都会返回错误。
>
>以只读模式打开数据库时，不会发生压缩(compactions)。
>
>一个 RocksDB 数据库，可以同时被任意多个用户以只读模式打开。
>
>如果一个 RocksDB 数据库同时被两个（或多个）用户以只读和读写模式打开，那么只读实例会存在未定义的行为（虽然经常可以成功，如果快速关闭），读写实例不受影响。
>
>此方法在 `ROCKSDB_LITE` 中不支持，此时 `OpenForReadOnly()` 函数将返回`Status::NotSupported`。



**细心的同学会发现，数据库选项的类型这里是 DBOptions 而不是之前的 Options。这里先简单说下，不作具体展开。RocksDB 中的选项主要有数据库选项 DBOptions 和列族选项 ColumFamilyOptions 两种，而 Options 则是 DBOptions 和 ColumFamilyOptions 的集合，其继承自 DBOptions 和 ColumFamilyOptions ，再额外定义了几个方法（与选项无关的方法，即没有扩展 DBOptions 和 ColumFamilyOptions）。**[点这里查看 Options 类实现源码](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/options.h#L1341-L1373)。

当以只读模式打开数据库时，你只能指定被打开数据库列族中的一个子集，同时还必须指定默认列族。  
默认列族的名字为 "default"，其存储在常量 `ROCKSDB_NAMESPACE::kDefaultColumnFamilyName` 中。  
例如，抽象类 DB 的主实现类 DBImpl 的实现 `dbimpl.cc`中就有 `const std::string kDefaultColumnFamilyName("default");`。

ColumnFamilyDescriptor 和 ColumnFamilyHandle（抽象类） 的定义如下（这不是此处重点，暂时先不讲解，简单看看就可以了）：

ColumnFamilyDescriptor 中包含了列族的名字和这个列族的选项。

```cpp
struct ColumnFamilyDescriptor {
  std::string name;
  ColumnFamilyOptions options;
  ColumnFamilyDescriptor()
      : name(kDefaultColumnFamilyName), options(ColumnFamilyOptions()) {}
  ColumnFamilyDescriptor(const std::string& _name,
                         const ColumnFamilyOptions& _options)
      : name(_name), options(_options) {}
};
```

```cpp
class ColumnFamilyHandle {
 public:
  virtual ~ColumnFamilyHandle() {}
  // Returns the name of the column family associated with the current handle.
  virtual const std::string& GetName() const = 0;
  // Returns the ID of the column family associated with the current handle.
  virtual uint32_t GetID() const = 0;
  // Fills "*desc" with the up-to-date descriptor of the column family
  // associated with this handle. Since it fills "*desc" with the up-to-date
  // information, this call might internally lock and release DB mutex to
  // access the up-to-date CF options.  In addition, all the pointer-typed
  // options cannot be referenced any longer than the original options exist.
  //
  // Note that this function is not supported in RocksDBLite.
  virtual Status GetDescriptor(ColumnFamilyDescriptor* desc) = 0;
  // Returns the comparator of the column family associated with the
  // current handle.
  virtual const Comparator* GetComparator() const = 0;
};
```



另外，在删除 `dbptr` 中存储的指针前，用户也要删除 `handle` vector 中的指针。



### 3.4. 以从库模式打开



```cpp
static Status OpenAsSecondary(const Options& options, const std::string& name,
                              const std::string& secondary_path, DB** dbptr);
```

* `option`: 打开从库时的选项；
* `name`: 待打开从库的主库的路径；
* `secondary_path`: 从库存储信息日志的路径；
* `dbptr`: 用来操作打开数据库的指针。

以从库模式打开数据库前，主库必须已经存在。

从库模式是只读的，也就是说你不能通过操作从库修改数据库中的内容。

当通过此方法打开一个从库后，从库会动态地跟踪主库的 MANIFEST 文件。如果用户觉得有必要（比如用户认为跟踪不及时等等），也可以通过调用 `TryCatchUpWithPrimary()` 来手动跟踪。

如果在从库已经启动后，主库创建了一个新的列族，那么从库目前会忽略这个列族（官方说法 "currently ignored"，也就是说理想的状态应该是列族会跟踪新的列族，但是目前还没实现）。

如果一个列族已经被一个从库打开，此时主库若删除了这个列族，那么从库也将删除这个列族。但是在这个列族的句柄被删除之前，从库依然可以访问这个列族中的内容。

目前的从库追踪主库，只支持追踪 MANIFEST，还不支持追踪 WAL，官方说法后面会加入对追踪 WAL 的支持。

从上面也可以看出来，目前 RocksDB 中的从库功能还是不够完善的。



### 3.5. 以从库模式打开（指定列族）

```cpp
static Status OpenAsSecondary(
  const DBOptions& db_options, const std::string& name,
  const std::string& secondary_path,
  const std::vector<ColumnFamilyDescriptor>& column_families,
  std::vector<ColumnFamilyHandle*>* handles, DB** dbptr);
```

* `db_options`: 打开从库时的选项；
* `name`: 待打开从库的主库路径；
* `secondary_path`: 从库存储信息日志的路径；
* `column_families`: 指定列族的描述符数组。如果指定的某个列族不存在，这个方法会返回一个 non-OK status；
* `handles`: 指定列族的句柄数组；
* `dbptr`: 用来操作打开数据库的指针。

此方法大部分内容特性与 3.4 中的一致：

>以从库模式打开数据库前，主库必须已经存在。
>
>从库模式是只读的，也就是说你不能通过操作从库修改数据库中的内容。
>
>当通过此方法打开一个从库后，从库会动态地跟踪主库的 MANIFEST 文件。如果用户觉得有必要（比如用户认为跟踪不及时等等），也可以通过调用 `TryCatchUpWithPrimary()` 来手动跟踪。
>
>如果在从库已经启动后，主库创建了一个新的列族，那么从库目前会忽略这个列族（官方说法 "currently ignored"，也就是说理想的状态应该是列族会跟踪新的列族，但是目前还没实现）。
>
>如果一个列族已经被一个从库打开，此时主库若删除了这个列族，那么从库也将删除这个列族。但是在这个列族的句柄被删除之前，从库依然可以访问这个列族中的内容。
>
>目前的从库追踪主库，只支持追踪 MANIFEST，还不支持追踪 WAL，官方说法后面会加入对追踪 WAL 的支持。
>
>从上面也可以看出来，目前 RocksDB 中的从库功能还是不够完善的。

另外，在删除 `dbptr` 中存储的指针前，用户也要删除 `handle` vector 中的指针。



### 3.6. 打开数据库（指定列族）

```cpp
static Status Open(const DBOptions& db_options, const std::string& name,
                   const std::vector<ColumnFamilyDescriptor>& column_families,
                   std::vector<ColumnFamilyHandle*>* handles, DB** dbptr);
```

这个方法似乎是 3.1 和 3.3 的混合版本。

* `db_options`: 打开数据库时的选项；
* `name`: 待打开数据库的路径；
* `column_families`: 指定列族的描述符数组；
* `handles`: 指定列族的句柄数组；
* `dbptr`: 用来操作打开数据库的指针；

如果你想以读些模式打开数据库，那么必须打开全部的列族，你可以通过 `ListColumnFamilies()` 方法来获取全部的列族，这个方法获取的列族顺序是未指定的。其定义如下：

```cpp
static Status ListColumnFamilies(const DBOptions& db_options,
                                 const std::string& name,
                                 std::vector<std::string>* column_families);
```

如果你不想荡开全部的列族，那就和 3.2 一样，以只读模式打开部分列族。

如果一切正常，返回时 handles 将与 column_families 大小相同 —— handles[i] 将是一个句柄，用于对列族 column_family[i] 进行操作。

在删除 DB 之前，你必须通过调用 `DestroyColumnFamilyHandle()` 来关闭所有列族。



### 3.7. 打开并压缩

```cpp
static Status OpenAndCompact(
  const std::string& name, const std::string& output_directory,
  const std::string& input, std::string* output,
  const CompactionServiceOptionsOverride& override_options);
```

以只读模式打开数据库并执行压缩程序。

压缩结果不会存在 DB 里，而是存在 `output_directory`。

这个 API 只应该与 `options.CompactionService` 一起使用，来执行由 `CompactionService` 触发的 compaction。

这个方法的参数我也还没弄清楚，文档与源码中并没有写，等我理解了以后更新这里。



## 4. 恢复数据库 `Resume()`

```cpp
virtual Status Resume() { return Status::NotSupported(); }
```



抽象类 DB 中没有支持恢复数据库操作，所以返回了一个 `NotSupported()` 状态。

这个恢复数据库操作，DB 的实现类中可能有实现（例如主实现类 DBImpl），本文重点只是 DB 类，就不在这里说了。



## 5. 关闭数据库 `Close()`

```cpp
virtual Status Close() { return Status::NotSupported(); }
```

通过释放资源、关闭文件等方式关闭数据库。

这个方法应该在调用析构函数之前调用，以便调用者可以检查返回状态，以防出现任何错误。

这将不会 fsync WAL 文件。如果需要同步，调用者必须首先调用 `SyncWAL()` 或 `Write()` 使用 `WriteOptions.sync=true` 的空写批处理。

无论返回的状态是什么，DB 都必须被释放。

如果返回状态为 `Aborted()`，则关闭失败，因为系统中存在未释放的快照。在这种情况下，用户可以释放未释放的快照，然后再试一次。对于除了 `Aborted()` 的其他状态，再次调用 `Close()` 什么也不会发生。

如果返回状态是 `NotSupported()`，那么 DB 的实现类会执行析构函数中的清理操作。

抽象类 DB 中没有支持关闭数据库操作，所以返回了一个 `NotSupported()` 状态。

这个关闭数据库操作，DB 的实现类中可能有实现（例如主实现类 DBImpl），本文重点只是 DB 类，就不在这里说了。



## 6. 创建列族

### 6.1. 创建一个列族

```cpp
virtual Status CreateColumnFamily(const ColumnFamilyOptions& options,
                                  const std::string& column_family_name,
                                  ColumnFamilyHandle** handle);
```

* `options`: 要创建列族的选项；
* `column_family_name`: 要创建列族的名字；
* `handle`: 列族的句柄。

这个方法通过 `option` 和 `column_family_name` 创建一个列族，并将其句柄存在 `handle` 中，你可以通过 `handle` 来操作这个列族。

> 此方法在 DB 类中没有实现。在其主实现类 DBImpl 的 `dbimpl.cc` 源文件中，将其默认实现为仅返回 `NotSupported()` 状态，也就是没有实现，挂了个空壳。此方法具体的实现得看具体的实现类，例如 DBImpl，这里先只知道怎么用就可以了。

### 6.2. 创建多个列族（通过选项和名字）

```cpp
virtual Status CreateColumnFamilies(
  const ColumnFamilyOptions& options,
  const std::vector<std::string>& column_family_names,
  std::vector<ColumnFamilyHandle*>* handles);
```

* `options`: 要创建列族的选项；
* `column_family_names`: 要创建的多个列族的名字数组；
* `handles`: 存储创建的多个列族的句柄的句柄数组。

这个方法将创建多个列族，并且把这些列族的句柄都存在 `handles` 中。

所有的列族使用的都是 `option` 中的选项，都是一样的。

如果有列族创建失败了，创建操作可能也会继续，会把创建成功的列族句柄存在 `handles` 中，即 `handles` vector 的长度是创建成功的列族个数。

> 此方法在 DB 类中没有实现。在其主实现类 DBImpl 的 `dbimpl.cc` 源文件中，将其默认实现为仅返回 `NotSupported()` 状态，也就是没有实现，挂了个空壳。此方法具体的实现得看具体的实现类，例如 DBImpl，这里先只知道怎么用就可以了。

### 6.3. 创建多个列族（通过 ColumnFamilyDescriptor）

```cpp
virtual Status CreateColumnFamilies(
  const std::vector<ColumnFamilyDescriptor>& column_families,
  std::vector<ColumnFamilyHandle*>* handles);
```

* `column_families`: 要创建的多个列族的描述符数组；
* `handles`: 存储创建的多个列族的句柄的句柄数组。



这个方法将创建多个列族，并且把这些列族的句柄都存在 `handles` 中。

这个和上一个 6.2 中方法的区别是，这个为每个列族指定一个选项，而不是所有列族使用一样的选项，因为我们之前说过列族描述符 ColumnFamilyDescriptor 中存有列族选项与列族名字。

如果有列族创建失败了，创建操作可能也会继续，会把创建成功的列族句柄存在 `handles` 中，即 `handles` vector 的长度是创建成功的列族个数。

> 此方法在 DB 类中没有实现。在其主实现类 DBImpl 的 `dbimpl.cc` 源文件中，将其默认实现为仅返回 `NotSupported()` 状态，也就是没有实现，挂了个空壳。此方法具体的实现得看具体的实现类，例如 DBImpl，这里先只知道怎么用就可以了。





## 7. 删除列族



### 7.1. 删除一个列族

```cpp
virtual Status DropColumnFamily(ColumnFamilyHandle* column_family);
```

通过指定的句柄删除一个列族。

这个操作不会立刻删除列族，只是在 MANIFEST 中添加一个删除记录，并阻止这个列族 flush 或 compact。

> 此方法在 DB 类中没有实现。在其主实现类 DBImpl 的 `dbimpl.cc` 源文件中，将其默认实现为仅返回 `NotSupported()` 状态，也就是没有实现，挂了个空壳。此方法具体的实现得看具体的实现类，例如 DBImpl，这里先只知道怎么用就可以了。

### 7.2. 删除多个列族

```cpp
virtual Status DropColumnFamilies(
  const std::vector<ColumnFamilyHandle*>& column_families);
```

通过指定的句柄数组删除多个列族。

这个操作不会立刻删除列族，只是在 MANIFEST 中添加一个删除记录，并阻止这个列族 flush 或 compact。

如果删除某个列族时失败了，删除操作还会继续（也就是会成功删除部分列族），用户可以调用 `ListColumnFamilies()` 方法来查看列族列表，以此来了解列族的删除情况。

> 此方法在 DB 类中没有实现。在其主实现类 DBImpl 的 `dbimpl.cc` 源文件中，将其默认实现为仅返回 `NotSupported()` 状态，也就是没有实现，挂了个空壳。此方法具体的实现得看具体的实现类，例如 DBImpl，这里先只知道怎么用就可以了。

### 7.3. 关闭列族（删除句柄）

```cpp
virtual Status DestroyColumnFamilyHandle(ColumnFamilyHandle* column_family);
```

其实现在 `dbimpl.cc` 中，不过是直接实现的，实现类 DBImpl 没有重写此方法。如下：

```cpp
Status DB::DestroyColumnFamilyHandle(ColumnFamilyHandle* column_family) {
  delete column_family;
  return Status::OK();
}
```

这个实现其实就是删除了句柄。

这个方法的实际意义是，关闭列族，删除句柄，以防止句柄被重复 delete。

其实意义就是没了句柄，你就操作不了列族了，相当于关闭了。

官方建议使用此方法删除句柄，而不是你自己直接 delete 句柄（虽然我看着默认实现好像没啥区别，不过这个方法的实现是可能被具体的实现类重写的，那就大不一样！）。



## 8. 插入操作

下面的操作，官方建议将写选项 `option.sync` 设为 true（默认是 false），这里你暂时只需要知道，若这个选项为 true，写入、删除等操作会更不容易丢失，但性能会下降。

### 8.1. 指定列族插入

```cpp
virtual Status Put(const WriteOptions& options,
                   ColumnFamilyHandle* column_family, const Slice& key,
                   const Slice& value) = 0;
```

* `options`: 写选项；
* `column_family`: 指定列族
* `key`: 待插入的 key；
* `value`: 待插入的 value。

在指定列族插入一对 kv，如果 key 已经存在，那么 value 会被覆盖。

插入成功返回 OK 状态，否则返回一个非 OK 的状态。

此方法在 DB 类中没有实现，必须由实现类来实现。

### 8.2. 在默认列族插入

```cpp
virtual Status Put(const WriteOptions& options, const Slice& key,
                   const Slice& value) {
  return Put(options, DefaultColumnFamily(), key, value); 
}
```

这个方法就是调用了上一个方法，将列族指定为默认列族，别的都一样。

## 9. 删除操作

下面的操作，官方建议将写选项 `option.sync` 设为 true（默认是 false），这里你暂时只需要知道，若这个选项为 true，写入、删除等操作会更不容易丢失，但性能会下降。

### 9.1. 从指定列族中删除

```cpp
virtual Status Delete(const WriteOptions& options,
                      ColumnFamilyHandle* column_family,
                      const Slice& key) = 0;
```

* `options`: 写选项；
* `column_family`: 指定列族
* `key`: 待删除的 key。

从指定列族中删除一个 key。

删除成功返回 OK 状态，否则返回一个非 OK 的状态。

注意如果要删除的 key 本来就不存在，返回的也是 OK，也就是说 RocksDB 不认为这是异常状态。



### 9.2. 从默认列族中删除

```cpp
virtual Status Delete(const WriteOptions& options, const Slice& key) {
  return Delete(options, DefaultColumnFamily(), key);
}
```

这个方法就是调用了上一个方法，将列族指定为默认列族，别的都一样。

### 9.3. 从指定列族中删除一个没有被覆盖过的 key

```cpp
virtual Status SingleDelete(const WriteOptions& options,
                            ColumnFamilyHandle* column_family,
                            const Slice& key) = 0;
```

这个方法目前还是处于实验阶段的，只用于非常特定的工作场景下的性能优化。

这个方法会删除一个 key，但前提是这个 key 不能被覆盖过，比如多次调用 `Put()` 为一个 key 写入（覆盖）多次。

现在这个方法需要程序员来保证被删除的 key 没有被覆盖过。

如果 key 被覆盖过，或者将 SingleDelete() 和 Delete()、Merge() 等操作混合使用，其行为是未定义的。

### 9.4. 从默认列族中删除一个没有被覆盖过的 key

```cpp
virtual Status SingleDelete(const WriteOptions& options, const Slice& key) {
  return SingleDelete(options, DefaultColumnFamily(), key);
}
```

这个方法就是调用了上一个方法，将列族指定为默认列族，别的都一样。

### 9.5. 删除一个范围内的 key

```cpp
virtual Status DeleteRange(const WriteOptions& options,
                           ColumnFamilyHandle* column_family,
                           const Slice& begin_key, const Slice& end_key);
```

* `options`: 写选项；
* `column_family`: 指定列族
* `begin_key`: 待删除的 key 范围起点；
* `end_key`: 待删除的 key 范围终点。

此方法删除 [`begin_key`, `end_key`) 之间的 key，注意是**左闭右开**的。

和其他删除方法一样，如果 key 不存在，不认为是错误。

我们知道 RocksDB 中的 key 是有序的，可能是默认排序，也可能是使用用户指定的比较器来排序的，如果 `end_key` 在 `begin_key` 前面，会返回一个 `Status::InvalidArgument` 错误。

这个方法在生产环境中是很有用的，但是 RocksDB 也给出了两个注意事项：

1. 在 memtable 中如果有大量的墓碑(tombstones，即被标记删除但是还没真正删除的 key)，会导致读性能下降，这可以通过偶尔手动 flush 避免；
2. 在存在一个范围的墓碑的情况下，如果最大打开文件数被限制了，也会导致读性能下降。为了避免这个问题，在可能的情况下把 `max_open_files` 设为 `-1`（设为 `-1` 表示不限制最大打开文件数）。



## 10. 合并操作

下面的两个操作，官方建议将写选项 `option.sync` 设为 true（默认是 false），这里你暂时只需要知道，若这个选项为 true，写入、删除等操作会更不容易丢失，但性能会下降。

合并(Mergo)操作是 RocksDB 中原子 Read-Modify-Write（读-修改-写）操作，这里先简单了解，知道 DB 类里大概有啥就行，我会在其他文章详细说。



### 10.1. 合并一个条目到指定列族

```cpp
virtual Status Merge(const WriteOptions& options,
                     ColumnFamilyHandle* column_family, const Slice& key,
                     const Slice& value) = 0;
```

这个方法会将一个条目合并到指定列族，这个条目的 key 就是参数 `key`，value 就是参数 `value`。

这个方法具体的语义是由用户决定的，用户需要在打开数据库时传递一个 merge_operator，才可以使用此方法。

### 10.2. 合并一个条目到默认列族

```cpp
virtual Status Merge(const WriteOptions& options, const Slice& key,
                     const Slice& value) {
  return Merge(options, DefaultColumnFamily(), key, value);
}
```

这个方法就是调用了上一个方法，将列族指定为默认列族，别的都一样。

## 11. 应用更改 `Write()`

```cpp
virtual Status Write(const WriteOptions& options, WriteBatch* updates) = 0;
```

立即将 `updates` 中的内容应用到数据库。

如果参数 `updates` 里没有更新，且 `option.sync` 为 true，WAL仍然会被同步。

该方法由 DB 类具体的实现类实现。

官方建议将写选项 `option.sync` 设为 true（默认是 false）。



## 12. `Get()` 读操作

下面的读方法，具体实现要看具体的实现类，下面只说怎么用。

另外，时间戳(timestamp)这个东西，RocksDB 还在实验阶段，下面中涉及时间戳的方法，还不一定有效。

### 12.1. 从指定列族中读取 value

```cpp
virtual inline Status Get(const ReadOptions& options,
                          ColumnFamilyHandle* column_family, const Slice& key,
                          std::string* value) {
  assert(value != nullptr);
  PinnableSlice pinnable_val(value);
  assert(!pinnable_val.IsPinned());
  auto s = Get(options, column_family, key, &pinnable_val);
  if (s.ok() && pinnable_val.IsPinned()) {
    value->assign(pinnable_val.data(), pinnable_val.size());
  }  // else value is already assigned
  return s;
}
```

* `options`: 读选项；
* `column_family`: 待读取的列族；
* `key`: 待读取的 key；
* `value`: 将读到的 value 存在这里。

### 12.2. 从指定列族中读取 value（使用 PinnableSlice）

```cpp
virtual Status Get(const ReadOptions& options,
                   ColumnFamilyHandle* column_family, const Slice& key,
                   PinnableSlice* value) = 0;
```

* `options`: 读选项；
* `column_family`: 待读取的列族；
* `key`: 待读取的 key；
* `value`: 将读到的 value 存在这里。



### 12.3. 从默认列族中读取 value

```cpp
virtual Status Get(const ReadOptions& options, const Slice& key,
                   std::string* value) {
  return Get(options, DefaultColumnFamily(), key, value);
}
```

此方法和 12.1 的一样，只是从默认列族读。

### 12.4. 从指定列族中读取 value 和时间戳

```cpp
virtual inline Status Get(const ReadOptions& options,
                          ColumnFamilyHandle* column_family, const Slice& key,
                          std::string* value, std::string* timestamp) {
  assert(value != nullptr);
  PinnableSlice pinnable_val(value);
  assert(!pinnable_val.IsPinned());
  auto s = Get(options, column_family, key, &pinnable_val, timestamp);
  if (s.ok() && pinnable_val.IsPinned()) {
    value->assign(pinnable_val.data(), pinnable_val.size());
  }  // else value is already assigned
  return s;
}
```

这个方法的实现第 7 行调用了 12.5 中的方法，那个方法还没实现呢，所以这里的时间戳是无效的。

### 12.5. 从指定列族中读取 value 和时间戳（使用 PinnableSlice）

```cpp
virtual Status Get(const ReadOptions& /*options*/,
                   ColumnFamilyHandle* /*column_family*/,
                   const Slice& /*key*/, PinnableSlice* /*value*/,
                   std::string* /*timestamp*/) {
  return Status::NotSupported(
    "Get() that returns timestamp is not implemented.");
}
```

这个方法还没实现呢，源码里就是上面这样的，空的。

### 12.6. 从默认列族中读取 value 和时间戳

```cpp
virtual Status Get(const ReadOptions& options, const Slice& key,
                   std::string* value, std::string* timestamp) {
  return Get(options, DefaultColumnFamily(), key, value, timestamp);
}
```

此方法和 12.5 的一样，只是从默认列族读。

但是由于 12.5 中的方法还没实现呢，所以这个方法也无效。

## 13. 获取合并操作数 `GetMergeOperands()`

```cpp
virtual Status GetMergeOperands(
  const ReadOptions& options, ColumnFamilyHandle* column_family,
  const Slice& key, PinnableSlice* merge_operands,
  GetMergeOperandsOptions* get_merge_operands_options,
  int* number_of_operands) = 0;
```

（这个我暂时也没看懂，等我后面再更，如果你发现我忘了，邮件或者评论区踢我下！）

## 14. `MultiGet()` 读操作

和 `Get()` 类似，下面的读方法，具体实现要看具体的实现类，下面只说怎么用。

同样的，下面中涉及时间戳的方法，还不一定有效。

14.1 - 14.4 中的四个 `MultiGet()` 方法，用 std::string 类型存储读到的 value，将 Status 数组作为返回值返回；而 14.5 - 14.8 中则是用 PinnableSlice 类型存储读到的 value，将 Status 数组作为出参。后者的性能会更好，这个具体会在下面来说。

### 14.1. 跨列族读取多个 key

```cpp
virtual std::vector<Status> MultiGet(
  const ReadOptions& options,
  const std::vector<ColumnFamilyHandle*>& column_family,
  const std::vector<Slice>& keys, std::vector<std::string>* values) = 0;
```

* `options`: 读选项；
* `column_family`: 列族句柄数组；
* `keys`: 待读取的 key 数组；
* `values`: 读到的 value 数组，读到的 value 将会被存在这里。



显然，`column_family`，`keys` 和 `values` 三个 vector 的长度应该是一致的，且内容也是一一对应的。

例如，应当在列族 `column_family[i]` 读取 `keys[i]`，将结果存储到 `(*values)[i]`，并将操作状态 Status 存到返回值 vector 的索引为 `i` 的位置。

此方法不会对 key 去重，如果有重复 key，那么也会有重复的 value、Status 等，顺序和重复的 key 一致。





### 14.2. 从默认列族读取多个 key

```cpp
virtual std::vector<Status> MultiGet(const ReadOptions& options,
                                     const std::vector<Slice>& keys,
                                     std::vector<std::string>* values) {
  return MultiGet(
    options,
    std::vector<ColumnFamilyHandle*>(keys.size(), DefaultColumnFamily()),
    keys, values);
}
```

这个方法和上面 14.1 中的是没什么区别的，只是所有 key 都从默认列族读。



### 14.3. 跨列族读取多个 key 和时间戳

```cpp
virtual std::vector<Status> MultiGet(
  const ReadOptions& /*options*/,
  const std::vector<ColumnFamilyHandle*>& /*column_family*/,
  const std::vector<Slice>& keys, std::vector<std::string>* /*values*/,
  std::vector<std::string>* /*timestamps*/) {
  return std::vector<Status>(
    keys.size(), Status::NotSupported(
      "MultiGet() returning timestamps not implemented."));
}
```

此方法还没有实现，源代码就是上面这样的，空的。

### 14.4. 从默认列族读取多个 key 和时间戳

```cpp
virtual std::vector<Status> MultiGet(const ReadOptions& options,
                                     const std::vector<Slice>& keys,
                                     std::vector<std::string>* values,
                                     std::vector<std::string>* timestamps) {
  return MultiGet(
    options,
    std::vector<ColumnFamilyHandle*>(keys.size(), DefaultColumnFamily()),
    keys, values, timestamps);
}
```

这个方法就是调用了上面 14.3 中的方法，因为那个没有实现，所以这个方法现在也不能用。

### 14.5. 从一个列族中读取多个 key（使用 PinnableSlice）

```cpp
virtual void MultiGet(const ReadOptions& options,
                      ColumnFamilyHandle* column_family,
                      const size_t num_keys, const Slice* keys,
                      PinnableSlice* values, Status* statuses,
                      const bool /*sorted_input*/ = false) {
  std::vector<ColumnFamilyHandle*> cf;
  std::vector<Slice> user_keys;
  std::vector<Status> status;
  std::vector<std::string> vals;

  for (size_t i = 0; i < num_keys; ++i) {
    cf.emplace_back(column_family);
    user_keys.emplace_back(keys[i]);
  }
  status = MultiGet(options, cf, user_keys, &vals);
  std::copy(status.begin(), status.end(), statuses);
  for (auto& value : vals) {
    values->PinSelf(value);
    values++;
  }
}
```

重载的 `MultiGet()`，通过在读取路径中的批处理操作来提高性能。

目前，**只支持配备完整过滤器的 block based table 格式**，其他表格式，如 plain table, block based table with block based filters 和分区索引仍然可以工作，但不会获得任何性能优势。

* `options`: 读选项；
* `column_family`: 要读 key 的列族句柄，这里就一个列族，所有 key 都从这个列族读；
* `num_keys`: 要读的 key 的数量；
* `keys`: 待读取的 key 数组，这是一个 C 语言风格的数组，长度为 `num_keys`；
* `values`: 读到的 value 数组，读到的 value 将会被存在这里，这是一个 C 语言风格的数组，函数执行完后这个数组长度为 `num_keys`；
* `statuses`: 这是一个 C 语言风格的数组，其内容与每个 key 一一对应，存储读取 Status，函数执行完后这个数组长度为 `num_keys`；
* `sorted_input`: 输入的 key 数组是否已经排序好了，如果为 true，`MultiGet()·` 方法就不会再重复排序了，从而提高效率。**显然，这个参数对应的功能还没有实现，现在还是不起作用的。**







### 14.6. 从一个列族中读取多个 key 和时间戳（使用 PinnableSlice）

```cpp
virtual void MultiGet(const ReadOptions& options,
                      ColumnFamilyHandle* column_family,
                      const size_t num_keys, const Slice* keys,
                      PinnableSlice* values, std::string* timestamps,
                      Status* statuses, const bool /*sorted_input*/ = false) {
  std::vector<ColumnFamilyHandle*> cf;
  std::vector<Slice> user_keys;
  std::vector<Status> status;
  std::vector<std::string> vals;
  std::vector<std::string> tss;

  for (size_t i = 0; i < num_keys; ++i) {
    cf.emplace_back(column_family);
    user_keys.emplace_back(keys[i]);
  }
  status = MultiGet(options, cf, user_keys, &vals, &tss);
  std::copy(status.begin(), status.end(), statuses);
  std::copy(tss.begin(), tss.end(), timestamps);
  for (auto& value : vals) {
    values->PinSelf(value);
    values++;
  }
}
```

这个方法和 14.5 中的一样，只是加了个时间戳，而且是调用了 14.3 中的方法。

由于 14.3 中的方法还没有实现，所以这个方法目前也是无效的。

### 14.7. 跨列族中读取多个 key（使用 PinnableSlice）

```cpp
virtual void MultiGet(const ReadOptions& options, const size_t num_keys,
                      ColumnFamilyHandle** column_families, const Slice* keys,
                      PinnableSlice* values, Status* statuses,
                      const bool /*sorted_input*/ = false) {
  std::vector<ColumnFamilyHandle*> cf;
  std::vector<Slice> user_keys;
  std::vector<Status> status;
  std::vector<std::string> vals;

  for (size_t i = 0; i < num_keys; ++i) {
    cf.emplace_back(column_families[i]);
    user_keys.emplace_back(keys[i]);
  }
  status = MultiGet(options, cf, user_keys, &vals);
  std::copy(status.begin(), status.end(), statuses);
  for (auto& value : vals) {
    values->PinSelf(value);
    values++;
  }
}
```

这个方法和 14.5 中的区别，只是从一个列族换成了跨列族。

第 3 个参数由 `ColumnFamilyHandle* column_family` 变成了 `ColumnFamilyHandle** column_families`，这是 C 语言风格的柄数组，其他和 14.5 中干一样的。

### 14.8. 跨列族中读取多个 key 和时间戳（使用 PinnableSlice）

```cpp
virtual void MultiGet(const ReadOptions& options, const size_t num_keys,
                      ColumnFamilyHandle** column_families, const Slice* keys,
                      PinnableSlice* values, std::string* timestamps,
                      Status* statuses, const bool /*sorted_input*/ = false) {
  std::vector<ColumnFamilyHandle*> cf;
  std::vector<Slice> user_keys;
  std::vector<Status> status;
  std::vector<std::string> vals;
  std::vector<std::string> tss;

  for (size_t i = 0; i < num_keys; ++i) {
    cf.emplace_back(column_families[i]);
    user_keys.emplace_back(keys[i]);
  }
  status = MultiGet(options, cf, user_keys, &vals, &tss);
  std::copy(status.begin(), status.end(), statuses);
  std::copy(tss.begin(), tss.end(), timestamps);
  for (auto& value : vals) {
    values->PinSelf(value);
    values++;
  }
}
```

这个方法和 14.7 中的一样，只是加了个时间戳，而且是调用了 14.3 中的方法。

由于 14.3 中的方法还没有实现，所以这个方法目前也是无效的。



## 15. 检查 key 是否可能存在 `KeyMayExist()`



如果 key 在数据库中**肯定**不存在，则该方法返回 false，如果存在或者不确定，返回true。

如果调用者希望在内存中找到该 key 时获得 value，则必须传递 `value_found` 的 bool 值。

如果 value 被正确设置，`value_found` 将在返回时为 true。

这种检查可能比调用 `DB::Get()` 更轻量级。一种方法是避免使用任何 IOs 操作系统。

**目前，此方法的默认实现，都是直接返回 true，并把 `value_found` 设为 false。也就是说，这个方法的默认实现是无效的，具体实现依赖于具体的实现类，例如 `DBImpl`。**

**所以下面的方法（有 4 个重载），简单看看声明，知道其大概作用就好，具体的实现类不是本文要说的。**

#### 

```cpp
virtual bool KeyMayExist(const ReadOptions& /*options*/,
                         ColumnFamilyHandle* /*column_family*/,
                         const Slice& /*key*/, std::string* /*value*/,
                         std::string* /*timestamp*/,
                         bool* value_found = nullptr) {
  if (value_found != nullptr) {
    *value_found = false;
  }
  return true;
}

virtual bool KeyMayExist(const ReadOptions& options,
                         ColumnFamilyHandle* column_family, const Slice& key,
                         std::string* value, bool* value_found = nullptr) {
  return KeyMayExist(options, column_family, key, value,
                     /*timestamp=*/nullptr, value_found);
}

virtual bool KeyMayExist(const ReadOptions& options, const Slice& key,
                         std::string* value, bool* value_found = nullptr) {
  return KeyMayExist(options, DefaultColumnFamily(), key, value, value_found);
}

virtual bool KeyMayExist(const ReadOptions& options, const Slice& key,
                         std::string* value, std::string* timestamp,
                         bool* value_found = nullptr) {
  return KeyMayExist(options, DefaultColumnFamily(), key, value, timestamp,
                     value_found);
}
```



## 16. 新建迭代器(Iterator)



### 16.1. 新建一个空的迭代器（指定列族）

```cpp
virtual Iterator* NewIterator(const ReadOptions& options,
                              ColumnFamilyHandle* column_family) = 0;
```

这个是纯虚函数，具体实现要看具体实现类。

参数比较简单，就不说了。

其作用是创建一个读取指定列族的空的迭代器。

这个迭代器会创建在堆上，此方法返回指向此迭代器的指针。

新建的迭代器不能直接用，要先调用其中的 Seek 方法（有好几个，详见 [RocksDB -- 迭代器 Iterator 类](https://gukaifeng.cn/posts/rocksdb-die-dai-qi-iterator-lei/)），毕竟迭代器需要一个开始位置。

当不再需要的此迭代器的时候，要记得 delete，当 DB 被 delete 的时候，迭代器也应该被 delete。



### 16.2. 新建一个空的迭代器（默认列族）

```cpp
virtual Iterator* NewIterator(const ReadOptions& options) {
  return NewIterator(options, DefaultColumnFamily());
}
```

这个方法和 16.1 的没什么区别，只是指定列族为默认列族。其他的看 16.1 就好，都一样的。



### 16.3. 新建多个空的迭代器（跨列族）

```cpp
virtual Status NewIterators(
  const ReadOptions& options,
  const std::vector<ColumnFamilyHandle*>& column_families,
  std::vector<Iterator*>* iterators) = 0;
```

这个也是纯虚函数，具体实现要看具体实现类。

其作用是创建多个读取指定多个列族的空的迭代器。

这些迭代器会创建在堆上，此方法将所有创建的迭代器的指针存在一个 vector 中，返回指向此 vector 的指针。

新建的迭代器不能直接用，要先调用其中的 Seek 方法（有好几个，详见 [RocksDB -- 迭代器 Iterator 类](https://gukaifeng.cn/posts/rocksdb-die-dai-qi-iterator-lei/)），毕竟迭代器需要一个开始位置。

当不再需要的某个迭代器的时候，要记得 delete，当 DB 被 delete 的时候，迭代器也应该被 delete。

## 17. 快照(Snapshot)



快照就是某个时刻数据库状态的一个副本。

当不再需要快照时，调用者必须调用 `ReleasSnapshot()` 释放快照。

快照相关的具体实现依赖于具体的 DB 实现类，这里就先看看声明。



### 17.1. 获取快照



返回当前 DB 状态的句柄。

用这个句柄创建的迭代器都可以观察当前 DB 状态的稳定快照。

如果 DB 无法获取快照或不支持快照，则返回 nullptr。

```cpp
virtual const Snapshot* GetSnapshot() = 0;
```



### 17.2. 释放快照

释放之前创建的快照，此操作后之前的快照将不再可用。

```cpp
virtual void ReleaseSnapshot(const Snapshot* snapshot) = 0;
```





## 18. DB 的属性常量与相关方法





### 18.1. 属性常量

DB 类内定义了一些描述数据库属性的常量，均定义在 DB 类内的 struct `Properties` 中，且全部都是 `static` 的。

**要注意的是，DB 类中只有关于 `Properties` 的 struct 声明，是没有相关成员的。这里常量的具体使用，还是要依赖于具体的实现类。**

按照 RocksDB 的惯例，常量均以 k 开头。

这些属性常量，是不支持 `ROCKSDB_LITE` 的。

另外，属性名字不能以数字结尾，会被解析为参数（当然这个应该是 RocksDB 开发人员需要注意的，我们就看看）。



要看都定义了哪些常量的话，[这是 `Properties` 的定义代码](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L747-L988)。



### 18.2. 获取属性值的方法



下面获取属性值的方法，依赖于具体的实现类，这里只说函数声明。

获取属性值时，使用的名称是形如 "rocksdb.stats" 这样的那个，不是变量名啥的，也不是带 k 的常量名。在定义代码的注释中有一一对应的关系。

#### 18.2.1. 从指定列族中获取一个属性值 

```cpp
virtual bool GetProperty(ColumnFamilyHandle* column_family,
                         const Slice& property, std::string* value) = 0;
```

由于属性相关实现依赖于具体的实现类，所以某个属性，只有具体的实现类支持，你才能获取到。

如果实现类支持要查询的属性，则会把属性值存在 `value` 中，并返回 true，否则不会修改 `value`，且返回 false。

#### 18.2.2. 从默认列族中获取一个属性值

```cpp
virtual bool GetProperty(const Slice& property, std::string* value) {
  return GetProperty(DefaultColumnFamily(), property, value);
}
```

这个方法和上一个一样，只是指定了从默认列族获取。

#### 18.2.3. 从指定列族中获取一个属性值（使用 map）

```cpp
virtual bool GetMapProperty(ColumnFamilyHandle* column_family,
                            const Slice& property,
                            std::map<std::string, std::string>* value) = 0;
```

这个方法和 18.2.1 中的类似，区别只是把结果存在一个 map 里，map 中的键就是属性名字，值是属性值。

#### 18.2.4. 从默认列族中获取一个属性值（使用 map）

```cpp
virtual bool GetMapProperty(const Slice& property,
                            std::map<std::string, std::string>* value) {
  return GetMapProperty(DefaultColumnFamily(), property, value);
}
```

这个方法和上一个一样，只是指定了从默认列族获取。

#### 18.2.5. 从指定列族中获取一个整数类型的属性值

```cpp
virtual bool GetIntProperty(ColumnFamilyHandle* column_family,
                            const Slice& property, uint64_t* value) = 0;
```

这个方法和 18.2.1 中的没什么区别，只是把属性值存到一个 uint64_t 类型里。

不过不是所有的属性都支持把值存到整数类型中，[这里列出了所有支持的属性](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/db.h#L1011-L1051)。

#### 18.2.6. 从默认列族中获取一个整数类型的属性值

```cpp
virtual bool GetIntProperty(const Slice& property, uint64_t* value) {
  return GetIntProperty(DefaultColumnFamily(), property, value);
}
```

这个方法和上一个一样，只是指定了从默认列族获取。



#### 18.2.7. 从全部列族中获取某个整数类型属性值的和

```cpp
virtual bool GetAggregatedIntProperty(const Slice& property,
                                      uint64_t* value) = 0;
```

与 18.2.5 或 18.2.6 的获取一个列族的某个整数类型的属性值不同，此方法会查看数据库中所有的列族，然后计算所有列族中获取属性值的和，存在 `value 中`。

例如，你可以可以通过 `GetAggregatedIntProperty(“rocksdb.estimate-num-keys", &num_keys)` 获取整个 RocksDB 中大概的 key 的总数。之所以只能获取一个大概数值是因为 RocksDB 的磁盘文件有重复 key，而且 compact 的时候会进行 key 的淘汰，所以无法精确获取。

## 19. 重置内部统计数据 `ResetStats()`

```cpp
virtual Status ResetStats() {
  return Status::NotSupported("Not implemented");
}
```

这个方法默认没有实现，所以要看具体的实现类。

这个方法只能重置 DB 和所有列族中的内部统计数据，但是重置不了 `options.statistics`，因为这个不属于 DB。



## 20. 

