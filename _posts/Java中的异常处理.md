---
title: Java中的异常处理
mathjax: false
date: 2020-03-17 18:17:50
updated: 2020-03-17 18:17:50
tags: [Java]
categories: [编程语言基础]
toc: true
---

捕获错误最理想的是在编译期间，但有的错误只有运行时才会发生。

对于这些错误，一般有两种解决方法：

1. 遇到错误就终止程序运行；
2. 由程序员在编写程序时，就考虑到错误的检测、错误消息的提示以及错误的处理。


<!--more-->
## 1. 异常事件的分类

Java 程序运行过程中所发生的异常事件可分类两类：

1. Error：JVM系统内部错误、资源耗尽的严重情况；
2. Exception：其他因编程错误或偶然的外在因素导致的一般性问题，例如：
    * 空指针访问
    * 试图读取不存在的文件
    * 网络连接中断  






## 2. 异常类层次

![](https://gukaifeng.cn/posts/java-zhong-de-yi-chang-chu-li/Java%E4%B8%AD%E7%9A%84%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86_1.png)



## 3. 异常处理机制

* 在编写程序时，经常要在可能出现错误的地方加上检测代码。如进行 x/y 运算时，要检测分母为 0，数据为空，输入的不是数字而是字符等。过多的分支会导致程序的代码过长，可读性差，因此采用异常机制。
* Java 采用异常处理机制，将异常处理的程序代码集中在一起，与正常的程序代码分开，使得程序简洁，并易于维护。
* Java 提供的是异常处理的**抓抛模型**。
* Java 程序的执行过程中如出现异常，会自动生成一个异常类对象，该异常对象将被提交给 Java 运行时系统，这个过程称为抛出（throw）异常。
* 如果一个方法内抛出异常，该异常会被抛到调用方法中。如果异常没有在调用方法中处理，它会继续被抛给这个方法的调用者。这个过程将一直持续下去，直到异常被处理。这个过程称为捕获（catch）异常。
* 如果一个异常回到 main() 方法，并且 main() 方法也不处理，则程序运行终止。
* 程序员通常只能处理 Exception，而对 Error无能为力。

```JAVA
int i = 0;
try { // 用 try{} 来括住一段有可能出现异常的代码段
    Sysyem.out.println(3 / i);	// 由于异常程序会中断，可以通过异常处理机制防止程序中断
    不会被执行的其他语句 // try 内异常语句后的代码不会被执行
} catch(Exception e) { // 当不知道捕获的是什么类型的异常时，可以直接使用所有异常的父类 Exception
    // 异常处理代码
    e.printStackTrace();
    System.out.println(e.getMessage());
}
System.out.println("ok");
```



### 3.1. 处理异常

异常处理是通过 try-catch-finally 语句实现的

```java
try {
    // 可能出现异常的代码
} catch(ExceptionName1 e) {
    // 当产生 ExcentionName1 型异常时的处理措施
} catch(ExceptionName2 e) {
    // 当产生 ExcentionName2 型异常时的处理措施
} ... finally { // final 可选，一定会执行
    // 无条件执行的语句
}
```

### 3.2. 抛出异常

声明抛出异常时 Java 中处理异常的第二种方式。

#### 3.2.1. 声明抛出异常

如果一个方法中的语句执行时可能生成某种异常，但是并不能确定如何处理这种异常，则此方法应显式的声明抛出异常，表名该方法将不对这些异常进行处理，而由该方法的调用者负责处理（调用者使用try-catch）。

在方法声明中用 throws 子句可以声明抛出异常的列表，throws 后面的异常类型可以是方法中产生的异常类型，也可以是它的父类。

声明抛出异常举例

```java
public void readFile(String file) throws FileNotFoundException {
    // 读文件操作可能产生 FileNotFoundException 类型的异常
    FileInputStream fis = new FileInputStream(file);
}
```

重写方法声明抛出异常的原则

1. 子类重写父类的方法时，若父类方法有抛出异常，子类也要有抛出异常；
2. 重写方法不能抛出比被重写方法范围更大的异常类型。



#### 3.2.2. 人工抛出异常

Java 异常类对象除在程序执行过程中出现异常时由系统自动生成并抛出，也可根据需要人工创建并抛出。

首先要生异常类对象，然后通过 throw 语句实现抛出操作（提交给 Java 运行环境）。

```java
IOException e = new IOException();
throw e；
```

可以抛出的异常必须是 Throwable 或其子类的实例。下面为错误范例：

```java
throw new String("want to throw"); // error
```



人工抛出异常代码举例：

```java
// 会抛出异常的类
public class Error{
    public Error(int age) {
        this.age = age;
    }
    public void printage() throws MyException {
        if (age <= 0) throw new MyException("年龄小于0");
        else System.out.println(age);
    }
    int age;
}
// 除自原有的异常类，也可自己创建异常类
class MyException extends Exception {
    public MyException(String s) {
        super(s);
    }
}
```

```java
// 调用上面类方法的主程序
public static void main(String[] args) {
    Error e = new Error(-1);
    try {
        e.printage();
    } catch (Exception ee) {
        ee.printStackTrace();
    }
}
```

```java
// 调用程序的输出
java.lang.Exception: 年龄小于0
	at untitled.Error.printage(Error.java:8)
	at untitled.untitled.main(untitled.java:7)
```