---
title: Java中的字符串类
mathjax: false
date: 2020-03-19 23:44:34
updated: 2020-03-19 23:44:34
tags: [Java,JavaSE]
categories: [编程语言基础]
toc: true
---

Java中有多个字符串类，其中最常用的是String类。

由于String类不可变，这在某些场合下可能会造成资源浪费，例如拼接字符串会产生很多无用的中间对象。所以有了两个可变的字符串类StringBuffer和StringBuilder，用于解决相关问题。



## 1. String类

String类是Java中使用频率最高的类，是一个字符串类。

String对象是**不可变**对象，加减操作性能较差。



String类的常用方法

String类有一些比较重要的常用方法，如charAt()，concat()，contains()，endsWith()，equals()，equalsIgnoreCase()，hashCode()，indexOf()，length()，matches()，replace()，replaceAll，split()，startsWith()，substring()，trim()，valueOf()。

<!--more-->

下面是这些常方法的示例代码和输出结果。

```java
package MyString;

public class StringTest {

    public static void main(String[] args) {
        String a = "123;456;789;123 ";
        System.out.println(a.charAt(0)); // 返回第0个元素
        System.out.println(a.indexOf(";")); // 返回第一个;的位置
        System.out.println(a.concat(";000")); // 连接一个新字符串并返回，a不变
        System.out.println(a.contains("000")); // 判断a是否包含000
        System.out.println(a.endsWith("000")); // 判断a是否以000结尾
        System.out.println(a.equals("000")); // 判断是否等于000
        System.out.println(a.equalsIgnoreCase("000"));// 判断在忽略大小写情况下是否等于000
        System.out.println(a.length()); // 返回a长度
        System.out.println(a.trim()); // 返回a去除前后空格后的字符串，a不变
        String[] b = a.split(";"); // 将a字符串按照;分割成数组
        for (int i = 0; i < b.length; i++) {
            System.out.println(b[i]);
        }

        System.out.println("===================");

        System.out.println(a.substring(2, 5)); // 截取a的第2个到第5个字符 a不变
        System.out.println(a.replace("1", "a")); // 注意这里也是全部替换，第一个参数是普通字符串
        System.out.println(a.replaceAll("1", "a")); // replaceAll第一个参数是正则表达式

        System.out.println("===================");

        String s1 = "12345?6789";
        String s2 = s1.replace("?", "a");
        String s3 = s1.replaceAll("[?]", "a");
        // 这里的[?] 才表示字符问号，这样才能正常替换。不然在正则中会有特殊的意义就会报异常
        System.out.println(s2);
        System.out.println(s3);
        System.out.println(s1.replaceAll("[\\d]", "a")); //将s1内所有数字替换为a并输出，s1的值未改变。

    }
}
```

```
1
3
123;456;789;123 ;000
false
false
false
false
16
123;456;789;123
123
456
789
123 
===================
3;4
a23;456;789;a23 
a23;456;789;a23 
===================
12345a6789
12345a6789
aaaaa?aaaa
```



## 2.  StringBuffer/StringBuilder

StringBuffer/StringBuilder都是**可变**字符串。

StringBuffer/StringBuilder中的方法一样，区别在于同步。

* StringBuffer：字符串加减，同步，性能好。
* StringBuilder：字符串加减，不同步，性能更好。

-

StringBuffer/StringBuilder的常用方法有

append()，insert()，delete()，replace()，substring()，

toString() 转换为String类型，

length() 字符串实际大小（长度），

capacity() 字符串占用空间大小，

trimToSize() 去除空隙，将字符串存储压缩到实际大小。

如果有大量 append()，事先预估大小，再调用相应的构造函数，性能会更好一些。

-

下面是关于StringBuffer/StringBuilder，字符串长度与占用空间的测试，以StringBuffer为例。

StringBuffer的的初始大小为（16+初始字符串长度）即capacity=16+初始字符串长度。

append()操作后，一旦length大于capacity时，capacity便在前一次的基础上加1后翻倍。

如果append()的对象很长，超过(加1再2倍数额)，将以最新的长度更换。

```java
package MyString;

public class StringBufferCapacityTest {

    public static void main(String[] args) {
        //StringBuffer的的初始大小为（16+初始字符串长度）即capacity=16+初始字符串长度
        //length 实际长度  capacity 存储空间大小
        StringBuffer sb1 = new StringBuffer();
        System.out.println("sb1 length: " + sb1.length());
        System.out.println("sb1 capacity: " + sb1.capacity());
        System.out.println("=====================");

        StringBuffer sb2 = new StringBuffer("123");
        sb2.append("456");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
        System.out.println("=====================");

        sb2.append("7890123456789");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
        System.out.println("=====================");

        sb2.append("0");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
        //一旦length大于capacity时，capacity便在前一次的基础上加1后翻倍；
        System.out.println("=====================");

        //当前sb2length 20   capacity 40， 再append 70个字符 超过(加1再2倍数额)
        sb2.append("1234567890123456789012345678901234567890123456789012345678901234567890");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
        //如果append的对象很长，超过(加1再2倍数额)，将以最新的长度更换

        System.out.println("=====================");
        sb2.append("0");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
        sb2.trimToSize();
        System.out.println("=====after trime================");
        System.out.println("sb2 length: " + sb2.length());
        System.out.println("sb2 capacity: " + sb2.capacity());
    }

}

```

```
sb1 length: 0
sb1 capacity: 16
=====================
sb2 length: 6
sb2 capacity: 19
=====================
sb2 length: 19
sb2 capacity: 19
=====================
sb2 length: 20
sb2 capacity: 40
=====================
sb2 length: 90
sb2 capacity: 90
=====================
sb2 length: 91
sb2 capacity: 182
=====after trime================
sb2 length: 91
sb2 capacity: 91
```





## 3. 三种字符串类拼接性能比较

在下面的代码中，分别使用对String类，StringBuffer类，StringBuilder类进行50000次拼接操作，

耗时分别为1194，6和3。很明显，在拼接字符串效率上，StringBuffer/StringBuilder要远远好于String，StringBuilder又略好于StringBuffer。

```java
package MyString;

import java.util.Calendar;

public class StringAppendTest {

    public static void main(String[] args) {
        int n = 50000;
        Calendar t1 = Calendar.getInstance();
        String a = new String();
        for(int i=0;i<n;i++)
        {
            a = a + i + ",";
        }
        System.out.println(Calendar.getInstance().getTimeInMillis() - t1.getTimeInMillis());

        Calendar t2 = Calendar.getInstance();
        StringBuffer b = new StringBuffer("");
        for(int i=0;i<n;i++)
        {
            b.append(i);
            b.append(",");
        }
        System.out.println(Calendar.getInstance().getTimeInMillis() - t2.getTimeInMillis());

        Calendar t3 = Calendar.getInstance();
        StringBuilder c = new StringBuilder("");
        for(int i=0;i<n;i++)
        {
            b.append(i);
            b.append(",");
        }
        System.out.println(Calendar.getInstance().getTimeInMillis() - t3.getTimeInMillis());
    }
}
```

```shell
1194
6
3
```

