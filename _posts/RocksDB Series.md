---
title: RocksDB Series
date: 2021-06-29
updated: 2021-06-29
categories: [数据库]
tags: [数据库, RocksDB]
toc: true
---



>曾经我想过，我以后，也许会做前端、也许会做服务器、也许会去做游戏，
>但从没想过自己有一天会踏入数据库领域，可能是冥冥天意吧。
>现在，我接触到了数据库领域，摆在我面前的第一个坎，便是这业内使用极为广泛的 RocksDB kv 存储引擎。
>寻觅很久，但 RocksDB 的相关学习资料甚少。
>于是便有了此系列博客，我将在学习 RocksDB 数据库的同时在此系列博客中记录其过程，希望可以帮到后来人。

此系列博客的所有文章，均基于 **RocksDB v6.25.3** 撰写。  
使用版本 v6.25.3 是因为，本弱鸡开始学习 RocksDB 并且撰写此博客时，这是 RocksDB 的最新版本。  
固定版本仅仅是为了便于学习，同时将忽略此版本以前的 bug，专注于 RocksDB 在此版本中已经提供的内容。  
固定版本并不代表不再关注 RocksDB 新版本中的的新特性，以及后续新版本中对 v6.25.3 版本中现存 bug 的修复，这些内容将在学习完 v6.25.3 后继续探索。

<!--more-->

此笔记分为概念篇、使用篇与源码篇：  
概念篇讲述了 RocksDB 的一些概念、术语、结构等内容；  
使用篇帮助大家来使用原生 RocksDB；  
源码篇则是通过分析源码更深入地理解 RocksDB。

学习 RocksDB 最好的参考资料，一定是其官方文档，但官方文档部分内容阅读起来晦涩难懂。因此，为了更便于读者学习理解，此系列博客有较多篇幅翻译自 RocksDB 相关文档，同时调整了部分文档结构，添加了更多相关说明。读者也可在阅读此博客时，搭配官方相关文档。  
RocksDB Home: https://rocksdb.org/  
RocksDB GitHub: https://github.com/facebook/rocksdb  
RocksDB Wiki: https://github.com/facebook/rocksdb/wiki

#### 1. 概念篇

#### 2. 使用篇
#### 3. 源码篇

