---
title: RocksDB - BlockBasedTable 分析
mathjax: false
date: 2021-05-19 15:13:47
updated: 2021-05-19 15:13:47
tags:
categories: [数据库]
toc: true
---



> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>







参考链接：https://github.com/facebook/rocksdb/wiki/Rocksdb-BlockBasedTable-Format

\- BlockBasedTable 是 RocksDB 默认的表类型。
\- 在 BlockBasedTable 中，数据被存储到固定大小的块，每个块依次存储数据条目。
\- 当我们把数据存储到块中时，数据可以被高效的压缩或编码，也就是说，最终存储到块中的数据大小，往往是远小于原始数据大小的。
\- 当读取一条记录时，我们首先定位这个记录所在的块，然后把这个块读到内存中，最后在这个块中查找要读取的记录。
\- 为了避免频繁读取一个相同的块，RocksDB 引入了块缓存将加载的块保存在内存中。

<!--more-->


### 文件格式

```
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1: filter block]                  (see section: "filter" Meta Block)
[meta block 2: index block]
[meta block 3: compression dictionary block]  (see section: "compression dictionary" Meta Block)
[meta block 4: range deletion block]          (see section: "range deletion" Meta Block)
[meta block 5: stats block]                   (see section: "properties" Meta Block)
...
[meta block K: future extended block]  (we may add more meta blocks in the future)
[metaindex block]
[Footer]                               (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```



这个文件中包含了内部指针 `BlockHandles`，`BlockHandles` 中有两个信息：

* offset:    varint64
* size:        varint64

更多关于 varint64 的内容参见  [this document](https://developers.google.com/protocol-buffers/docs/encoding#varints)。



1. 文件中的键值对序列会被按序并划分存储在一系列数据库中。这些数据块依次存放在文件的开头。每个数据块都按照 `block_builder.cc` 中的代码进行格式化，然后可选压缩。
2. 在这些数据块后面，BlockBasedTable 存放了一些元块（meta block），上面已经描述过了有哪些元块，未来 RocksDB 将会开发更多的元块。每个元块也会再用 `block_build.cc` 格式化，并可选压缩。
3. 在所有的 meta block 后面，有一个元索引块（metaindex block），每有一个元块，元索引块中就有一个对应的条目，这个条目的 key 是该元块的名字，value 是 `BlockHandle` 中一个指向该元块位置的指针。
4. 在文件的最后，是一个固定长度的 footer，其中包含了元索引块和 index block 的 `BlockHandle`，以及一个神奇的数字（magic number）。

```
metaindex_handle: char[p];  		// Block handle for metaindex
index_handle:     char[q];      // Block handle for index
padding:          char[40-p-q]; // zeroed bytes to make fixed length
                                // (40==2*BlockHandle::kMaxEncodedLength)
magic:            fixed64;      // 0x88e241b785f4cff7 (little-endian)
```

\* magic number: 暂时没有理解。



### `Index` Block

前面说过，查找一条记录时，先找到存储这条记录的 data block，然后把这个 data block 读到内存，再在块中查找记录。Index Block 就是用来查找这个 data block 的，其是一个二分查找的数据结构。一个文件可能包含一个 index block，或者一系列划分后的 index blocks （参考 [Partitioned Index Filters](https://github.com/facebook/rocksdb/wiki/Partitioned-Index-Filters)）。index block 格式在 [Index Block Format](https://github.com/facebook/rocksdb/wiki/Index-Block-Format) 中。





### `Filter` Meta Block

> Note: `format_version`=5 (Since RocksDB 6.6) uses a faster and more accurate [Bloom filter implementation](https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter) for full and partitioned filters.

#### Full filter

这个过滤器用于整个 SST 文件

#### Partitional Filter

这是一个划分成了多个块的 Full filter。顶级 index block 中会添加一个映射键用于与这些块通信。

#### Block-based filter

已弃用。继续了解见 https://github.com/facebook/rocksdb/wiki/Rocksdb-BlockBasedTable-Format#block-based-filter。



### `Properties` Meta Block

这个元块包含了一堆属性 kv，key 是属性名，value 是属性值。

块的内容格式如下

```
[属性1] 		(每个属性都是一个键值对)
[属性2]
...
[属性n]
```

这些属性会被排序且不重复。

默认情况下，每个表都会提供下面这几项属性

```
 data size               // the total size of all data blocks. 
 index size              // the size of the index block.
 filter size             // the size of the filter block.
 raw key size            // the size of all keys before any processing.
 raw value size          // the size of all value before any processing.
 number of entries
 number of data blocks
```

RocksDB 还给用户提供了一个 `callback`，用户可以用 ` callback` 获取想要的表的其他属性。参考 `UserDefinedPropertiesCollector`。



### `Compression Dictionary` Meta Block

这个元块包含了一个字典，用于在压缩/解压缩每个块前准备好压缩库。目的是解决小数据块在使用动态压缩算法时的一个基本问题：字典是在单次读取完块的时候建立的，所以小数据块总是有小且无效的字典。

对于这个问题，RocksDB 中的解放方案是，通过之前读取过的块中数据构建一个字典，再用这个字典初始化压缩库。随后，这个字典会被存储在一个文件级元块中，在解压缩时使用。字典大小的上限可以通过 `CompressionOPtions::max_dict_bytes` 配置，默认是 0，即不会产生或者存储 Compression Dictionary Meta Block。目前支持四种方案： `kZlibCompression`、 `kLZ4Compression`、 `kLZ4HCCompression`  以及 `kZSTDNotFinalCompression`。

这个 Compression Dictionary 仅用在最低级的压缩中，数据最大最稳定。为了避免遍历输入数据太多次，这个字典仅包含了子压缩的第一个输出文件中的样本。这个字典会被应用、存储在所有后续输出文件的元块中。要注意下，字典不会被应用或者存储在第一个输出文件中，因为第一个文件完成处理之前，字典的内容还不确定。

目前采样是均匀的、随机的，每个样本是 64 字节。因为在选择样本偏移时，我们预先不知道输出文件的大小，所以我们假定输出文件大小会达到最大值，这通常是正确的，因为这是子压缩的第一个文件。如果这个文件比较小，一些样本区间的边界可能会超过 EOF，这只意味着，最后字典的大小，只比  `CompressionOptions::max_dict_bytes` 小一点点。



### `Range Deletion` Meta Block





