---
title: Java大数类(BigInteger和BigDecimal)
mathjax: false
date: 2020-03-19 21:00:00
updated: 2020-03-19 21:00:00
tags: [Java]
categories: [编程语言基础]
toc: true
---



Java从5.0版本开始提供了两个大数类，BigInteger和BigDecimal。

* 大整数类BigInteger：支持无限大的整数运算。
* 大浮点数类BigDecimal：支持无限大的小数运算。注意精度和截断。

<!--more-->

## 1. 大整数类BigInteger

关于BigInteger大整数类的测试代码和输出结果如下。

```java
package BigNumber;

import java.math.BigInteger;

public class BigIntegerTest {

    public static void main(String[] args) {
        BigInteger b1 = new BigInteger("123456789"); // 声明BigInteger对象
        BigInteger b2 = new BigInteger("987654321"); // 声明BigInteger对象
        System.out.println("b1: " + b1 +  ", b2:" + b2);
        System.out.println("加法操作：" + b2.add(b1)); // 加法操作
        System.out.println("减法操作：" + b2.subtract(b1)); // 减法操作
        System.out.println("乘法操作：" + b2.multiply(b1)); // 乘法操作
        System.out.println("除法操作：" + b2.divide(b1)); // 除法操作，会截断
        System.out.println("最大数：" + b2.max(b1)); // 求出最大数
        System.out.println("最小数：" + b2.min(b1)); // 求出最小数
        BigInteger[] result = b2.divideAndRemainder(b1); // 求出余数的除法操作
        System.out.println("商是：" + result[0] + "；余数是：" + result[1]);
        System.out.println("等价性是：" + b1.equals(b2));
        int flag = b1.compareTo(b2);
        if (flag == -1)
            System.out.println("比较操作: b1<b2");
        else if (flag == 0)
            System.out.println("比较操作: b1==b2");
        else
            System.out.println("比较操作: b1>b2");
    }
}
```

```
b1: 123456789, b2:987654321
加法操作：1111111110
减法操作：864197532
乘法操作：121932631112635269
除法操作：8
最大数：987654321
最小数：123456789
商是：8；余数是：9
等价性是：false
比较操作: b1<b2
```





## 2. 大浮点数类BigDecimal

关于BigDecimal大浮点数类的测试代码和输出结果如下。

```java
import java.math.BigDecimal;
import java.math.BigInteger;

public class BigDecimalTest {
	public static void main(String[] args) {
		BigDecimal b1 = new BigDecimal("123456789.987654321"); // 声明BigDecimal对象
		BigDecimal b2 = new BigDecimal("987654321.123456789"); // 声明BigDecimal对象
		System.out.println("b1: " + b1 +  ", b2:" + b2);
		System.out.println("加法操作：" + b2.add(b1)); // 加法操作
		System.out.println("减法操作：" + b2.subtract(b1)); // 减法操作
		System.out.println("乘法操作：" + b2.multiply(b1)); // 乘法操作
		//需要指定位数，防止无限循环，或者包含在try-catch中
		System.out.println("除法操作：" + b2.divide(b1,10,BigDecimal.ROUND_HALF_UP)); // 除法操作，BigDecimal.ROUND_HALF_UP是四舍五入
		
		System.out.println("最大数：" + b2.max(b1)); // 求出最大数
		System.out.println("最小数：" + b2.min(b1)); // 求出最小数
		
		int flag = b1.compareTo(b2);
		if (flag == -1)
			System.out.println("比较操作: b1<b2");
		else if (flag == 0)
			System.out.println("比较操作: b1==b2");
		else
			System.out.println("比较操作: b1>b2");
		
		System.out.println("===================");
		
		//尽量采用字符串赋值，精度准确
		System.out.println(new BigDecimal("2.3"));
		System.out.println(new BigDecimal(2.3)); // 会出现精度问题
		
		System.out.println("===================");
		
		BigDecimal num1 = new BigDecimal("10");
		BigDecimal num2 = new BigDecimal("3");
		//需要指定位数，防止无限循环，或者包含在try-catch中
		BigDecimal num3 = num1.divide(num2, 3, BigDecimal.ROUND_HALF_UP);
		System.out.println(num3);
	}
}
```

```
b1: 123456789.987654321, b2:987654321.123456789
加法操作：1111111111.111111110
减法操作：864197531.135802468
乘法操作：121932632103337905.662094193112635269
除法操作：8.0000000099
最大数：987654321.123456789
最小数：123456789.987654321
比较操作: b1<b2
===================
2.3
2.29999999999999982236431605997495353221893310546875
===================
3.333
```



