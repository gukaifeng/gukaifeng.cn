---
title: Java数字工具类
mathjax: false
date: 2020-03-19 22:55:03
updated: 2020-03-19 22:55:03
tags: [Java,JavaSE]
categories: [编程语言概念]
toc: true
---

Java数字工具类java.lang.Math的几个常用函数。

```java
package MathTest;

public class MathTest {

    public static void main(String[] args) {

        System.out.println(Math.abs(-5));    //绝对值
        System.out.println(Math.max(-5,-8)); //最大值
        System.out.println(Math.min(-5,-8)); //最小值
        System.out.println(Math.pow(-5,2));  //求幂
        System.out.println(Math.round(3.5)); //四舍五入
        System.out.println(Math.ceil(3.5));  //向上取整
        System.out.println(Math.floor(3.5)); //向下取整
    }

}
```

```
5
-5
-8
25.0
4
4.0
3.0
```

