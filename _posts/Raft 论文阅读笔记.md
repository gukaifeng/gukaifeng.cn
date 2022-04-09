---
title: Raft 论文阅读笔记
date: 2022-04-08 00:47:00
updated: 2022-04-08 00:47:00
categories: [论文阅读笔记]
tags: [Raft,论文,分布式]
---



Raft 论文原文传送门：  
[In Search of an Understandable Consensus Algorithm (Extended Version)](https://raft.github.io/raft.pdf)





## 0. 摘要



Raft 是一种用于管理复制日志的一致性算法。

Raft 产生的结果和 (multi-)Paxos 一样，并且和 Paxos 一样高效，但是 Raft 的结构和 Paxos 不同。这使 Raft 比 Paxos 更容易理解，也为在构建实际系统提供了更好的基础。

为了更容易理解，Raft 分离了一致性的关键要素，例如领导选举(leader election)，日志复制(log replication)和安全性(safety)。

Raft 加强了一致性，以减少必须考虑的情况的数目。

一个来自用户学习的结果说明，对于学生来说，Raft 比 Paxos 更容易学。

Raft 还包括一种用于更改集群成员的新机制，该机制使用<font color=red>重叠多数</font>来保证安全。



## 1. 导论



一致性算法允许一组机器作为一个一致的组来工作，这个组能够在其部分成员故障时幸存。因此，在构建可靠的大规模软件系统中，一致性算法扮演着关键的角色。在过去十年间，关于一致性算法的讨论被 Paxos 统治：大部分一致性实现都是基于 Paxos 或受 Paxos 影响的，并且 Paxos 成为了给学生讲关于一致性内容的主要工具。

不幸的是，尽管有数次让 Paxos 更平易近人的尝试，其还是相当难于理解。此外，Paxos 的架构需要复杂的修改以支持实际的系统。结果就是，系统构建者和学生都在为 Paxos 苦苦挣扎。
