---
title: Java中的工厂方法(FactoryMethod)
mathjax: false
date: 2020-03-16 23:56:15
updated: 2020-03-16 23:56:15
tags: [Java,设计模式]
categories: [设计模式]
toc: true
---

FactoryMethod 模式是设计模式中应用最为广泛的模式。

在面向对象编程中，对象的创建工作非常简单，但对象创建的时机却很重要。

FactoryMethod 解决的就是这个问题。

FactoryMethod 通过面向对象的手法，将所要创建具体对象的创建工作延迟到了子类，从而提供了一种扩展的策略，较好的解决了这种紧耦合的关系。

<!--more-->

* 工厂方法举例

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/Java%E4%B8%AD%E7%9A%84%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95-FactoryMethod_1.png)

    开发人员1：橙色框；

    开发人员2：紫色框。

    理解：开发人1 提供一个固定的制造汽车的接口，此接口永远不变。需要更改内容的的时候，只修改内容、与内容和接口之间的衔接，不对开发人员2的代码造成任何影响。

