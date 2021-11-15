---
title: Java随机数类
mathjax: false
date: 2020-03-19 22:24:47
updated: 2020-03-19 22:24:47
tags: [Java]
categories: [编程语言基础]
toc: true
---



Java中提供了两种主要的获取随机数的方法，分别是Random随机数类和Math.random()函数，其中Random随机数类更为常用。

Java中生成随机数的方法代码范例和输出结果如下。

<!--more-->

```java
package Random;

import java.util.Random;

public class RandomTest {

    public static void main(String[] args)
    {
        //第一种办法，采用Random类 随机生成在一定范围内的随机数
        Random rd = new Random();
        System.out.println(rd.nextInt());
        System.out.println(rd.nextInt(100)); //0--100的随机数
        System.out.println(rd.nextLong());
        System.out.println(rd.nextDouble()); //0.0--1.0的随机数
        System.out.println("=========================");

        //第二种，生成一个范围内的随机数 例如0到时10之间的随机数
        //Math.random() 生成(0,1)的Double数
        System.out.println(Math.round(Math.random()*10)); // Math.round()四舍五入
        System.out.println("=========================");


        //JDK 8 新增方法
        rd.ints();  //返回无限个int类型范围内的数据
        int[] arr = rd.ints(10).toArray();  //生成10个int范围类的个数。
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
        System.out.println("=========================");

        arr = rd.ints(5, 10, 100).toArray();
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

        System.out.println("=========================");

        arr = rd.ints(10).limit(5).toArray(); // limit()用于限定个数
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }

}
```

```
1404320356
53
-8492628924417241038
0.06849198818168623
=========================
1
=========================
-1312517856
-509940637
2134173514
-1742855070
447151405
-1519546383
-512164412
-1127120168
-978804043
-1470191102
=========================
69
39
41
98
17
=========================
1744076186
498618148
-584035397
546922796
1704853629
```





