---
title: Java中的泛型
mathjax: false
date: 2020-03-17 18:49:03
updated: 2020-03-17 18:49:03
tags: [Java,JavaSE]
categories: [编程语言基础]
toc: true
---

## 1. 泛型类

```java
class Generic<T> {...}
```

## 2. 泛型接口

```java
interface Generic<T> {...}
```

如果类中不确定接口中的泛型，那么类也要定义为泛型类。

## 3. 泛型方法

```JAVA
class Generic {
    
    // 无返回值的泛型方法
    public <T> void test1(T s) {
        T t = s;
    }
    // 有返回值的泛型方法
    public <T> T test2(T s) {
        return s;
    }
  
  	// 无返回值的可变参数泛型方法
    public <T> void test3(T... strs) {
        for(T s: strs) {
          System.out.println(s);
        }
    }
}
```

在静态方法中，不能使用类定义的泛型，若需要泛型，只能自己定义一个泛型。

<!--more-->

## 4. 泛型通配符

```JAVA
class Generic {
    // test 方法需要一个 List 参数，但不确定 List 内的数据类型
    public void test(List<?> list);
}
```

泛型通配符的用法举例：

```java
<? extends Person> // 只允许 Person 类及其子类使用
<? super Person> // 只允许 Person 类及其父类使用
<? extends Comparable> // 只允许实现了 Comparable 接口的类使用
```