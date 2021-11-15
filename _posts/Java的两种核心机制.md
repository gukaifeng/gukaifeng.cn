---
title: Java的两种核心机制
mathjax: false
date: 2020-03-16 22:42:27
updated: 2020-03-16 22:42:27
tags: [Java,JavaSE]
categories: [编程语言概念]
toc: true
---

## 1. JVM(Java虚拟机)

* JVM（Java Virtual Machine，Java虚拟机）

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/Java%E7%9A%84%E4%B8%A4%E7%A7%8D%E6%A0%B8%E5%BF%83%E6%9C%BA%E5%88%B6_1.png)

JVM 内存模型

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/Java%E7%9A%84%E4%B8%A4%E7%A7%8D%E6%A0%B8%E5%BF%83%E6%9C%BA%E5%88%B6_2.png)

## 2. CC(垃圾收集)

* CC（Garbage Collection，垃圾收集机制）



使用一个系统级线程，检测和回收内存。自动回收，但回收不及时。

