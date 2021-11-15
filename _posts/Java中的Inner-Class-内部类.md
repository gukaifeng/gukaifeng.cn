---
title: Java中的Inner Class(内部类)
mathjax: false
date: 2020-03-16 23:46:54
updated: 2020-03-16 23:46:54
tags: [Java]
categories: [编程语言基础]
toc: true
---

## 1. 什么是内部类

* 在 Java 中，允许一个类的定义位于另一个类内部，前者称为内部类，后者称为外部类。
* 内部类的作用：主要解决 Java 不能多重继承的问题。
* Inner Class 一般用在定义它的类或语句块之内，在外部引用它时必须给出完整的名称。
* Inner Class 的名字不能与包含它的类相同。
* Inner Class 可以使用外部类的私有数据，因为他是外部类的成员，同一个类的成员之间可以相互访问。而外部类要访问内部类的成员需要 `内部类.成员` 或 `内部类对象.成员`。

<!--more-->

## 2. 内部类的分类

1. 成员内部类（static 成员内部类和非 static 成员内部类）
2. 局部内部类（不谈修饰符）
3. 匿名内部类

## 3. Inner Class 作为类的成员

1. 可以声明为 final；
2. 和外部类不同，可以声明为 private 和 protected；
3. Inner Class 可以声明为 static 的，但此时就不能再使用外层类的非 static 成员变量。

## 4. Inner Class 作为类

可以声明为 abstract 类，因此可以被其他内部类继承。

非 static 的内部类中的成员不能声明为 static 的，只有在外部类或 static 内部类中才能声明 static 成员。



## 5. 内部类实现多重继承举例

```java
// class A 同时继承 class B 的 testB() 方法 和 class C 的 testC() 方法

class A {
    public void testB() {
        new InnerB().testB();
    }
    public void testC() {
        new InnerC().testC();
    }
  
    private class InnerB extends B {
        public void testB() {
            System.out.println("this is override testB()");
        }
    }
    private class InnerC extends C {
        public void testC() {
            System.out.println("this is override testC()");
        }
    }
}

class B { public void testB() {} }
class C { public void testC() {} }

// main 函数
// 输出结果
// this is override testB()
// this is override testC()
public class Inner {
    public static void main(String[] args) {
        A a = new A();
        a.testB();
        a.testC();
    }
}
```

