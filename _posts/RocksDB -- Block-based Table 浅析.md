---
title: RocksDB -- Block-based Table 浅析
date: 2021-12-08 18:01:43
updated: 2021-12-09 17:27:35
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---



## 1. 什么是 Block-based Table



Block-basedTable 是 RocksDB 默认的 SST 表格式。

SST 是 RocksDB 架构的重要部分，关于 RocksDB 的架构，查看 [RocksDB -- 高级架构](https://gukaifeng.cn/posts/rocksdb-gao-ji-jia-gou/)。

SST 文件就是数据库目录里那些以后缀 `.sst` 结尾的文件，例如 `000009.sst`，其中存储着 RocksDB 持久化的数据。

<!--more-->

## 2. 文件格式

Block-based Table 的格式如下：

```
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]
[meta block 2: index block]
[meta block 3: compression dictionary block]
[meta block 4: range deletion block]
[meta block 5: stats block]
...
[meta block K: future extended block]
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```

我们可以看到，在文件的开头，有 N 个 data block，随后是 K 个 meta block，然后有 1 个 metaindex block 和 1 个 Footer。（N 和 K 不是固定的，这里只是表示多个，且 N 和 K 的值是相互独立的。）

下面，我们把这些内容分开来说。



## 3. BlockHandle



这个文件中，包含了一些内部指针，称为 `BlockHandles`，其中包含以下信息:

```
offset:         varint64
size:           varint64
```

* `offset`: 在文件中的偏移，类型是 varint64；
* `size`: 大小，类型是 varint64。

我们这里不介绍什么是 varint，有关 varint64 格式的说明，可以看看这里 [什么是 Varint 编码](#)。

`BlockHandles` 用来描述一个 block 的位置和大小，通过其中的 `offset` 可以找到一个 block 的开始位置，再结合 `size`，就确定了一个 block 的开始和结束位置。

我们在下面的文件格式解析中，会说明这些 `BlockHandles` 具体存在哪里。



## 4. 文件格式解析





### 4.1. data block

data block 即数据块，实际存储数据的块。

RocksDB 将文件中的 k/v 对序列按顺序存储，并划分为一系列 data block，每个 data block 的大小都是一样的。

这些 data block 连续地存在文件的开头。

每个数据块都根据 `block_builder.cc` 中的代码进行格式化，然后可选压缩。





### 4.2. meta block

meta block 即元块，元块是用于某种功能的块，

在 datablock 之后，存放着一些 meta block。

上面的格式信息中，描述了 Block-based Table 目前支持的 meta block 类型，即 meta block 1-5，分别是 filter block、index block、compression dictionary block、range deletion block 和 stats block，只有这五个。

以后可能会加入更多类型的 meta block 的支持。

每个 meta block 再次使用 `block_builder.cc` 的代码进行格式化，然后可选压缩。



#### 4.2.1. filter block

过滤器块。

过滤器大类上主要分为全过滤器和分区过滤器，也就是下面的 1 和 2，具体的过滤器实现可以不同。

我们下面简单介绍下：

1\. `Full filter`: 应用于整个 SST 文件的过滤器。

2\. `Partitioned Filter`: `Full filter` 被划分为多个块。添加一个顶级索引块来将键映射到相应的过滤分区。查看 [RocksDB -- Partitioned Index Filters 浅析](#) 这篇文章了解更多内容。

3\. `Block-based filter`: 已弃用。这里不说了，有兴趣的可以自己去看看官方 wiki 或者看看源码。



#### 4.2.2. index block

在我们查找一个 key 时，就会用到 index block，index block 用于查找一个 data block，这个 data block 包含一个 key 范围，这个范围里面可能有我们要查找的 key。

index block 是一种二进制搜索数据结构。

一个 SST 文件可能包含一个 index block，或者一列分区索引块(partitioned index blocks)。关于分区索引快，查看 [RocksDB -- Partitioned Index Filters 浅析](#) 这篇文章。

index block 的格式，查看 [RocksDB -- Index Block 格式](#) 这篇文章。



#### 4.2.3. compression dictionary block



这个 meta block 包含在压缩/解压缩每个块之前用来启动压缩库的字典。

它的目的是解决动态字典压缩算法在小数据块上的一个基本问题：字典是在对块的一次传递中构建的，所以小数据块总是有小而无效的字典。

RocksDB 的解决方案是用一个字典来初始化压缩库，这个字典是从前面看到的块中采样的数据构建的。然后，该字典被存储在文件级元块中，以便在解压过程中使用。字典大小的上限可以通过`CompressionOptions::max_dict_bytes` 配置。默认情况下，它是 0，也就是说，这个 block 不会生成或存储。目前，`kZlibCompression`、`kLZ4Compression`、`kLZ4HCCompression` 和 `kZSTDNotFinalCompression` 都支持该特性。

更具体地说，压缩字典只在压缩到最底层时构建，在底层数据最大且最稳定。为了避免多次迭代输入数据，字典只包含来自 subcompaction 的第一个输出文件的示例。然后，字典被应用到所有后续输出文件的 meta block 中并存储在其中。

注意，字典不会应用于第一个文件，也不会存储在第一个文件中，因为字典的内容直到该文件被完全处理后才最终确定。

目前的采样是均匀随机的，每个采样是 64 字节。在选择样本偏移量时，RocksDB 不预先知道输出文件的大小，因此 RocksDB 假设它将达到最大大小，这通常是正确的，因为它是 subcompaction 中的第一个文件。在文件较小的情况下，一些样本间隔将引用超出 EOF 的偏移量，这只是意味着字典将比`CompressionOptions::max_dict_bytes` 小一点。

#### 4.2.4. range deletion block

这个 meta block 包含范围删除操作的 key 范围和序号范围。

范围删除不能与点数据一起内联在数据块中，因为这样的话，这个范围将会无法进行二分查找。

这个 block 的格式是标准的键值格式。一个范围删除被编码如下:



```
User key: 范围开始的 key
Sequence number: 将范围删除插入到 DB 中的序列号(sequence number)
Value type: kTypeRangeDeletion
Value: 范围结束的 key
```



当使用与非范围数据类型相同的机制（Put、Delete 等等）插入时，才会给范围删除分配序列号。

范围删除还使用与点数据相同的 flush/compaction 机制遍历 LSM，且只有在压缩到最底层的过程中，才会被废弃（即被删除）。

#### 4.2.5. stats block

这个元块包含很多属性，每个属性都是一个键值对。

key 是属性的名称，value 是属性值。



stats 块的格式如下:

```
[prop1]    (每个属性都是一个键值对)
[prop2]
...
[propN]
```

属性保证在没有重复的情况下进行排序。

默认情况下，每个表都提供了以下属性：

```
 data size               // 所有 data block 的总大小
 index size              // index block 的大小
 filter size             // filter block 的大小
 raw key size            // 在进行任何处理前，所有 key 的总大小
 raw value size          // 在进行任何处理前，所有 value 的总大小
 number of entries       // SST 中条目的数量
 number of data blocks   // data block 的个数
```

RocksDB 还为用户提供了 "callback" 功能来收集他们对该表感兴趣的属性。想了解更多可以看看 `TablePropertiesCollector` 相关的内容，这是一个抽象类，定义在 `table_properties.h` 中，[源码在这里](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/table_properties.h#L80-L128)。

### 4.3. metaindex block

metaindex block，顾名思义，元索引块，用于索引 meta block。

每有一个 meta block，metaindex block 中就有一个条目。

每个条目都是一个 key/value 对，key 是 meta block 的名字，value 是一个 `BlockHandle`，其中包含对应 meta block 在文件中的位置和大小。



### 4.4. Footer

在文件的最后是一个固定长度的 Footer。

其包含 metablock block 和 index block 的 `BlockHandle` 以及一个 magic number。

下面是官方 wiki 给出的 Footer 内容示意图：

```
metaindex_handle: char[p];      // Block handle for metaindex
index_handle:     char[q];      // Block handle for index
padding:          char[40-p-q]; // zeroed bytes to make fixed length
// (40==2*BlockHandle::kMaxEncodedLength)
magic:            fixed64;      // 0x88e241b785f4cff7 (little-endian)
```

metaindex block 的 `BlockHandle` 占 p 个字节，index block 的 `BlockHandle` 占 q 个字节。

接下来是 40-p-q 个 0 字节，用来填充。也就是说 magic number 之前的内容固定大小为 40 字节，如果 metaindex block 的 `BlockHandle` 和 index block 的 `BlockHandle`  加起来不到 40 字节，就要补 0 补到 40 字节。

然后就是一个 magic number 了，这个数是固定的 64 位，即 8字节，采用小端序存储。







