---
title: Java中的Interface(接口)
mathjax: false
date: 2020-03-16 22:13:01
updated: 2020-03-16 22:13:01
tags: [Java,JavaSE]
categories: [编程语言概念]
toc: true
---

## 1. 什么是Interface(接口)

有时必须从几个类中派生出一个子类，继承他们所有的属性和方法。

但是，Java 不支持多重继承，使用接口实现多重继承的效果。

接口是抽象方法和常量值的定义的集合。

从本质上讲，接口是一种特殊的抽象类，这种抽象类中只包含常量和方法的定义，而没有变量和方法的实现。

一个类可以实现多个接口，接口也可以继承其他接口。

```java
public class ClassName implements Interface1, Interface2 { ... }
public interface Interface3 extends Interface1 { ... }
```

<!--more-->

## 2. 接口的特点

1. 用 `interface` 来定义；
2. 接口中所有的成员变量都默认是由 public static final 修饰的；
3. 接口中所有的方法都默认是由 public abstract 修饰的；
4. 接口没有构造器；
5. 接口采用多层继承机制。

## 3. 接口定义举例

```java
public interface Runner {
    int ID = 1;
    void start();
    public void run();
    void stop();
}
```

```java
public interface Runner {
    public static final int ID = 1;
    public abstract void start();
    public abstract void run();
    public abstract void stop();
}
```

上面两段代码完全等价。

## 4. 接口的应用举例

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/Java%E4%B8%AD%E7%9A%84Interface(%E6%8E%A5%E5%8F%A3)_1.png)

```java
public abstract class Person {
    public Person() {}
    public abstract void showInfo();
    String name;
    int sex;
    int age;
}

public interface Cooking {
    public abstract void fry();
}

public interface Singing {
    public abstract void singing();
}

public class SCTeacher extends Person implements Cooking, Sing {
    public void setInfo(String name, int sex, int age, String course) {
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.course = course;
    }
    public void showInfo() { ... }
    public void fry() { ... }
    public void singing() { ... }
    String course;
}
```

## 5. Interface的一些要点

* 实现接口的类中必须提供接口中所有方法的具体实现，否则仍未抽象类。

* 接口的主用用途就是被实现类实现（面向接口编程）。

* 与继承关系类似，接口与实现类之间存在多态性。

* 定义 Java 类的语法格式，先写 extends 后写 implements。

    ```java
    <modifier> class <name> [extends <superclass>] [implements <interface1> [, <interface2>, ...]]  { ... }
    ```

* 接口类引用变量可以接收实现类的对象。

    与父子类关系类似，接口类与实现类之间存在多态性。

    ```java
    Cooking c = new SCTeacher(); // 与父子类多态类似，c 只可访问 fry() 方法
    ```

* 接口类也可以使用 extends 继承其他接口。

* 同一个 `.class` 文件中，只可以有一个 public 类或接口。

### 