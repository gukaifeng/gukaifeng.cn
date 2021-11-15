---
title: Java中的final关键字
mathjax: false
date: 2020-03-16 23:40:17
updated: 2020-03-16 23:40:17
tags: [Java,JavaSE]
categories: [编程语言基础]
toc: true
---

在 Java 中声明类、属性和方法时，可使用关键字 final 来修饰，表示“最终”。

1. final 标记的类不能被继承

    如String类、Syetem类、StringBuffer类。

    ```java
    public final class TestPerson() { ... }
    ```
<!--more-->
2. final 标记的方法不能被重写

    如Object类中的 getClass()。

    ```java
    public class TestPerson() {
        public final void test() { ... }
    }
    ```

3. final 标记的变量（成员/局部变量）即为常量。名称大写，且只能被赋值一次。

    final 标记的成员变量必须在声明的同时，或在每个构造方法，或代码块中显式的赋值，然后才能使用。`final double PI= 3.14;`

    ```java
    public class TestPerson() {
        final String NAME_1 = "acb"; // final 修饰的常量必须显式赋值
        final static String NAME_2 = "qqq"; // final static 全局常量
    }
    ```

