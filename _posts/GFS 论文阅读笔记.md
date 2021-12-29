---
title: GFS 论文阅读笔记
date: 2021-12-28 23:31:39
updated: 2021-12-28 23:31:39
categories: [论文阅读笔记]
tags: [GFS,论文,分布式]
toc: true
---



传送门: [GFS 论文原文](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/035fc972c796d33122033a0614bc94cff1527999.pdf)



## 1. 什么是 GFS



GFS，全称 Google File System，谷歌文件系统。

这篇论文是 2003 年发表的，在这之前，GFS 已经大规模应用在了 Google 内部。

GFS 是 Google 提出的一个文件系统，其是分布式的，主要用于处理越来越庞大的数据。因为当数据量大到一定程度时，传统的数据存储与处理方式就显得很笨重了，不适用了（比如你很难很快地读取数百 TB 的数据）。



<!--more-->





## 2. 设计概述



opportunity n. 机会，机遇

allude v. 略加提及

lay out 展示

promptly adv. 迅速地

contiguous adj. 连续的

arbitrary adj. 任意的

conscious adj. 意识到的

Performance-conscious 注重表现的，追求性能的

steadily adv. 逐渐地

forth adv. 向前

go back and forth 来回走动

seldom adv. 很少

well-defined adj. 定义明确的

semantics n. 语义学

essential adj. 基本的，必不可少的

simultaneously adv. 同时地

sustain v. 维持、保持

sustained adj. 持续的，持久的

latency n. 延迟

place a premiun on ... 重视 ...

stringent adj. 严格的，严厉的

hierarchically adv. 分层次地

invaluable adj. 无价的，极为宝贵的

### 2.1. 假想

GFS 在设计的时候有一些假想，即预期要实现的目标。

1. 这个系统由很多廉价的、经常会故障的商用组件构建，所以在日常使用中，这个系统必须持续地监控自身，以检测、容忍组件故障，并迅速从组件故障中恢复。
2. 这个系统存储数量适中的大文件。Google 期望是几百万个文件，每个一般是 100MB 或者更大。数 GB 大小的文件在这个系统中也是很常见的，需要高效管理。而小文件肯定也要支持，但是不需要为了这些小文件专门优化。
3. 工作负载主要包括两类读：大文件流的读（流只能顺序读）和小文件的随机读。
    * 大文件流的读：单个读操作一般读几百 KB，更常见的是读 1MB 或者更多。来自同一个客户端连续的读操作经常是从一个文件连续的位置读。
    * 小文件的随机读：一般是在文件的任意位置读几 KB 大小。注重性能的应用程序通常对它们的小读取进行批处理和排序，以逐渐地浏览文件，而不是来回的读（文件指针来回移动）。
4. 这个系统也会有很多大的、连续的写操作，将数据追加到文件末尾。一般这种操作的大小和读差不多。一旦写入操作完成，这个文件很少会再次修改。小的随机写也支持，但是不太高效。
5. 这个系统必须高效地实现定义明确的语义，以支持多客户端并发写入（追加写入）同一个文件。GFS 中的文件通常用作生产者消费者队列或多路合并。系统中有数百个生产者，每个机器上运行一个，这些生产者并发地追加修改一个文件，因此以最小的同步开销来实现原子性是必不可少的。这些文件可能随后被读取，也可能有一个消费者在写的同时读。
6. 高的持续的带宽比低的延迟更重要。GFS 的大多数目标应用程序都重视以高速率批量处理数据，而很少有应用程序对单个读或写有严格的响应时间要求。





### 2.2. 接口

GFS 提供了一个常见的文件系统接口，尽管 GFS 没有实现像 POSIX 这样的标准 API。

GFS 中文件在目录中以层次结构组织，通过路径名区分。

GFS 支持常用操作以创建(create)、删除(delete)、打开(open)、关闭(close)、读(read)和写(write)文件。

此外，GFS 中还有 *snapshot* 和 *recore append* 操作。Snapshot 以一个很低的开销创建一个文件的或者一个目录树的拷贝。Record append 允许多个客户端并发地追加写入同一个文件，且确保每个客户端的写入操作都是原子的。Record append 对实现多路合并结果、生产者消费者队列很有用，因为很多客户端可以同时追加写入，而不需要额外的锁。Google 发现在构建大型分布式应用时，这些类型的文件是非常有用的。

Snapshot 和 record append 会在后面进一步讨论。





### 2.3. 架构

![Figure 1: GFS Architecture](https://gukaifeng.cn/posts/gfs-lun-wen-yue-du-bi-ji/GFS_Figure_1.png)

