---
title: Java列表List类
mathjax: false
date: 2020-03-20 19:04:38
updated: 2020-03-20 19:04:38
tags: [Java,数据结构]
categories: [编程语言基础]
toc: true
---

List 列表

* 有序的Collection
* 允许重复元素
* {1, 2, 4, {5, 2}, 1, 3}

List 的主要实现

* ArrayList（非同步的）
* LinkedList（非同步的）
* Vector（同步的），Vector 几乎和ArrayList一样，除了Vector本身是同步的。

<!--more-->

## 1. ArrayList

**ArrayList是Java中应用最广泛的List实现类。**

ArrayList

* 以**数组**实现的列表，不支持同步。
* 利用索引位置可以快速定位访问。
* 不适合指定位置的插入、删除操作。
* 适合变动不大，主要用于查询的数据。
* 和Java数组相比，其容量是可动态调整的。
* ArrayList在元素填满容器时会自动扩充容器大小的50%。

下面是ArrayList操作的示例代码与输出结果，一些解释在注释中。

在下面的代码中，也对ArrayList的三种遍历方法进行了效率测试，结果可以看到，使用迭代器遍历最慢，其次是索引位置遍历，最快的是for-each遍历。

```java
package List;

import java.util.ArrayList;
import java.util.Iterator;
//Vector 几乎和ArrayList一样，除了Vector本身是同步的

public class ArrayListTest {
    public static void main(String[] a) {
        ArrayList<Integer> al = new ArrayList<Integer>();
        al.add(3); // ArrayList中只能装对象，因此这里会自动把3装箱
        al.add(2);
        al.add(1);
        al.add(4);
        al.add(5);
        al.add(6);
        al.add(new Integer(6));

        System.out.print("The third element is  ");
        System.out.println(al.get(3));
        al.remove(3);  //删除第四个元素，后面元素往前挪动
        al.add(3, 9);  //将9插入到第4个元素，后面元素往后挪动

        System.out.println("======遍历方法=============");

        ArrayList<Integer> as = new ArrayList<Integer>(100000);
        for (int i = 0; i < 100000; i++) {
            as.add(i);
        }
        traverseByIterator(as);
        traverseByIndex(as);
        traverseByFor(as);
    }

    public static void traverseByIterator(ArrayList<Integer> al) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = al.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByIndex(ArrayList<Integer> al) {
        long startTime = System.nanoTime();
        System.out.println("============随机索引值遍历==============");
        for (int i = 0; i < al.size(); i++) {
            al.get(i);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(ArrayList<Integer> al) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : al) {
            ;
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
The third element is  4
======遍历方法=============
============迭代器遍历==============
5785409纳秒
============随机索引值遍历==============
5558767纳秒
============for循环遍历==============
4904241纳秒
```



## 2. LinkedList

LinkedList

* 以**双向链表**实现的列表，不支持同步。
* 可被当做堆栈、队列和双端队列进行操作。
* 顺序访问高效，随机访问较差，中间插入和删除高效。
* 适用于经常变化的数据。



下面是LinkedList操作的示例代码与输出结果，一些解释在注释中。

在下面的代码中，也对LinkedList的三种遍历方法进行了效率测试，结果可以看到，使用索引位置遍历最慢，其次是迭代器遍历，最快的是for-each遍历。

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;

public class LinkedListTest {

    public static void main(String[] args) {
        LinkedList<Integer> ll = new LinkedList<Integer>();
        ll.add(3);
        ll.add(2);
        ll.add(5);
        ll.add(6);
        ll.add(6);
        System.out.println(ll.size());
        ll.addFirst(9);  //在头部增加9
        ll.add(3, 10);   //将10插入到第四个元素，四以及后续的元素往后挪动
        ll.remove(3);    //将第四个元素删除

        LinkedList<Integer> list = new LinkedList<Integer>();
        for (int i = 0; i < 100000; i++) {
            list.add(i);
        }
        traverseByIterator(list);
        traverseByIndex(list);
        traverseByFor(list);

    }

    public static void traverseByIterator(LinkedList<Integer> list) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = list.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByIndex(LinkedList<Integer> list) {
        long startTime = System.nanoTime();
        System.out.println("============随机索引值遍历==============");
        for (int i = 0; i < list.size(); i++) {
            list.get(i);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(LinkedList<Integer> list) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : list) {
            ;
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
5
============迭代器遍历==============
7851446纳秒
============随机索引值遍历==============
3922928500纳秒
============for循环遍历==============
4288072纳秒
```



## 3. ArrayList和LinkedList性能比较

下面的代码中，对ArrayList和LinkedList分别进行了头部插如，读取和头部删除操作，每种操作ArrayList和LinkedList各10000次，比较性能。

在头部插入和头部删除时，ArrayList要进行大量元素的移动，因此效率较低。

在读取元素时，ArrayList的随机存取特性有极大的优势，性能远高于LinkedList。

 ```java
package List;

import java.util.ArrayList;
import java.util.LinkedList;

public class ListCompareTest {

    public static void main(String[] args) {
        int times = 10 * 1000;
        // times = 100 * 1000;
        // times = 1000 * 1000;

        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        LinkedList<Integer> linkedList = new LinkedList<Integer>();

        System.out.println("Test times = " + times);
        System.out.println("-------------------------");

        // ArrayList add
        long startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            arrayList.add(0, i);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + " <--ArrayList addFirst");

        // LinkedList add
        startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            linkedList.add(0, i);
        }
        endTime = System.nanoTime();
        duration = endTime - startTime;
        System.out.println(duration + " <--LinkedList addFirst");
        System.out.println("-------------------------");

        // ArrayList get
        startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            arrayList.get(i);
        }
        endTime = System.nanoTime();
        duration = endTime - startTime;
        System.out.println(duration + " <--ArrayList get");

        // LinkedList get
        startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            linkedList.get(i);
        }
        endTime = System.nanoTime();
        duration = endTime - startTime;
        System.out.println(duration + " <--LinkedList get");
        System.out.println("-------------------------");

        // ArrayList remove
        startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            arrayList.remove(0);
        }
        endTime = System.nanoTime();
        duration = endTime - startTime;
        System.out.println(duration + " <--ArrayList removeFirst");

        // LinkedList remove
        startTime = System.nanoTime();

        for (int i = 0; i < times; i++) {
            linkedList.remove(0);
        }
        endTime = System.nanoTime();
        duration = endTime - startTime;
        System.out.println(duration + " <--LinkedList removeFirst");
    }
}
 ```

```
Test times = 10000
-------------------------
6786110 <--ArrayList addFirst
4929023 <--LinkedList addFirst
-------------------------
1366341 <--ArrayList get
46363504 <--LinkedList get
-------------------------
3854896 <--ArrayList removeFirst
856570 <--LinkedList removeFirst
```





## 4. Vector

Vector

* 和ArrayList类似，可变数组实现的列表。
* Vector是**同步**的，适合在多线程下使用，也因为同步所以性能较ArrayList略差。
* 原先不属于JCF框架，属于Java最早的数据结构，性能较差。
* 从JDK1.2开始，Vector被重写，并纳入到JCF。
* 官方w文档建议在非同步情况下，优先采用ArrayList。



下面是Vector操作的示例代码与输出结果，一些解释在注释中。

在下面的代码中，也对Vector的四种遍历方法进行了效率测试，最快的依然是for-each遍历。

```java
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.Iterator;
import java.util.Vector;

public class VectorTest {

    public static void main(String[] args) {
        Vector<Integer> v = new Vector<Integer>();
        v.add(1);
        v.add(2);
        v.add(3);
        v.remove(2);
        v.add(1, 5);
        System.out.println(v.size());

        System.out.println("======遍历方法=============");

        Vector<Integer> v2 = new Vector<Integer>(100000);
        for (int i = 0; i < 100000; i++) {
            v2.add(i);
        }
        traverseByIterator(v2);
        traverseByIndex(v2);
        traverseByFor(v2);
        traverseByEnumeration(v2);
    }

    public static void traverseByIterator(Vector<Integer> v) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = v.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByIndex(Vector<Integer> v) {
        long startTime = System.nanoTime();
        System.out.println("============随机索引值遍历==============");
        for (int i = 0; i < v.size(); i++) {
            v.get(i);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(Vector<Integer> v) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : v) {
            ;
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByEnumeration(Vector<Integer> v) {
        long startTime = System.nanoTime();
        System.out.println("============Enumeration遍历==============");
        for (Enumeration<Integer> enu = v.elements(); enu.hasMoreElements(); ) {
            enu.nextElement();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
3
======遍历方法=============
============迭代器遍历==============
5655941纳秒
============随机索引值遍历==============
4243295纳秒
============for循环遍历==============
4140848纳秒
============Enumeration遍历==============
4458364纳秒
```

