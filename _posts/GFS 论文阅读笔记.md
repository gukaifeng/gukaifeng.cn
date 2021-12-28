---
title: GFS 论文阅读笔记
date: 2021-12-28 23:31:39
updated: 2021-12-28 23:31:39
categories: [论文阅读笔记]
tags: [GFS,论文,分布式]
toc: true
---



传送门: [GFS 论文原文](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/035fc972c796d33122033a0614bc94cff1527999.pdf)



## 1. 概述



GFS，全称 Google File System，谷歌文件系统。

这篇论文是 2003 年发表的，在这之前，GFS 已经大规模应用在了 Google 内部。

GFS 是 Google 提出的一个文件系统，其是分布式的，主要用于处理越来越庞大的数据。因为当数据量大到一定程度时，传统的数据存储与处理方式就显得很笨重了，不适用了（比如你很难很快地读取数百 TB 的数据）。



<!--more-->





