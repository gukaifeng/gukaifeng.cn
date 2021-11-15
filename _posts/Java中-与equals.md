---
title: Java中==与equals()
mathjax: false
date: 2020-03-16 23:30:09
updated: 2020-03-16 23:30:09
tags: [Java,JavaSE]
categories: [编程语言概念]
toc: true
---

## 1. `==`操作符

1. 基本类型比较值：只要两个变量的值相等，即为 true；
2. 引用类型比较引用：只有指向同一个对象时，== 才返回 true。

## 2. equals()方法

所有类都继承了 Object，也就获得了  equals() 方法，也还可以重写。

1. 只能比较引用类型，其作用与 == 相同，比较是否指向同一个对象

    格式：`obj.equals(obj2)`

2. 特殊的，当用 equals() 方法进行比较时，对类 File、String、Date 及包装类（Wrapper Class）来说，是比较类型和内容而不考虑引用的是否是一个对象。原因是在这些类中重写了 Object 类的 equals() 方法。

