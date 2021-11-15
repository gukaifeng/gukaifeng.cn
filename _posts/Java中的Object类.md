---
title: Java中的Object类
mathjax: false
date: 2020-03-16 23:34:19
updated: 2020-03-16 23:34:19
tags: [Java,JavaSE]
categories: [编程语言基础]
toc: true
---

Object 类是所有 Java 类的根父类；

如果在类的声明中未使用 extends 关键字指明其父类，则默认父类为 Object 类。

```java
// 下面两个类等价
public class Person { ... }
public class Person extends Object { ... }
```

``` java
public void method(Object obj) { ... } // 可以接收任何类作为参数
```

<!--more-->

* Object 类中的主要方法

| NO.  |             方法名称              | 类型 |                             描述                             |
| :--: | :-------------------------------: | :--: | :----------------------------------------------------------: |
|  1   |          public Object()          | 构造 |                           构造方法                           |
|  2   | public boolean equals(Object obj) | 普通 |                      判断是否为同一对象                      |
|  3   |       public int hashCode()       | 普通 |                         获取 Hash 码                         |
|  4   |     public String toString()      | 普通 | 打印"对象所属包@对象地址"。直接打印对象名，等价于打印其toString()返回结果，可以重写toString()函数打印期望的内容。 |

