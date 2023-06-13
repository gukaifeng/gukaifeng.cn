







## 1. 简介

RocksDB 是一个有着 key/value 接口的存储引擎，由 FaceBook 公司基于 LevelDB 开发，并向后兼容 LevelDB 的 API。作为一个存储引擎，RocksDB 工作在各种各样的存储介质上，其最初的目标是快速存储 (尤其是 Flash 存储)。  RocksDB 是一个存储 key/value 的 C++ 库，kv 是任意长度的字节流。支持点查找、范围扫描，并且提供了不同类型的 ACID 保证策略。

RocksDB 平衡了可自定义性与自适应性。RocksDB 支持高度灵活的配置设置，以保证其可以再各种生产环境下运行，包括 SSD、硬盘、ramfs 以及远程存储。RocksDB 支持多种压缩算法，支持一些在产品支持与调试上的优秀工具。另一方面，RocksDB 努力限制旋钮的数量，以提供足够好的开箱即用性能，并在适用的情况下使用一些自适应算法。

<!--more-->


## 2. 特性

1\) 专为希望在本地或远程存储系统上存储数 TB 的数据的应用服务器而设计；  
2\) 针对在快速存储设备 (flash 设备或内存) 中存储小到中等大小的 kv 优化；  
3\) RocksDB 可以在多核处理器中运行良好。

另外，由于 RocksDB 基于 LevelDB 开发，相对于 LevelDB 扩展了很多新特性。  
RocksDB 的 wiki 中列举了属于 RocksDB，但不属于 LevelDB 的特性：https://github.com/facebook/rocksdb/wiki/Features-Not-in-LevelDB。


## 3. 假设和目标

**性能**

RocksDB 的主要设计点是，其  
应该具有快速存储和服务器工作负载的性能；  
应该支持有效的点查找和范围扫描；  
应该可以配置为支持高随机读取工作负载、高更新工作负载或两者的组合；  
体系结构应该支持对不同的工作负载和硬件进行轻松的权衡。

**产品支持**


RocksDB 应该被设计成这样一种方式：内置支持工具和实用程序，以帮助在生产环境中部署和调试。如果存储引擎还不能自动适应某些应用程序和硬件，RocksDB 将提供一些参数以允许用户调优性能。


**兼容性**

RocksDB 的新版本应该向后兼容，以便在升级到 RocksDB 的新版本时，现有的应用程序不需要更改。除非使用新提供的特性，现有的应用程序也应该能够恢复到最近的旧版本。See [RocksDB Compatibility Between Different Releases](https://github.com/facebook/rocksdb/wiki/RocksDB-Compatibility-Between-Different-Releases).


## 4. 参考资料

https://github.com/facebook/rocksdb/wiki  
https://github.com/facebook/rocksdb/wiki/RocksDB-Overview
