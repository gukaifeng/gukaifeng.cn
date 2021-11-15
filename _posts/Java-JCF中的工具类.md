---
title: Java JCF中的工具类
mathjax: false
date: 2020-03-21 14:48:33
updated: 2020-03-21 14:48:33
tags: [Java,数据结构]
categories: [编程语言基础]
toc: true
---

JCF中的工具类

* 不存储数据，而是在数据容器上，实现高效操作。例如排序、搜索。
* Arrays类：数组的工具类，提供了对数组操作的工具方法。
* Collections类：集合对象的工具类，提供了操作集合的工具方法。

Arrays和Collections中所有的方法都为静态的，不需要创建对象，直接使用类名调用即可。

<!--more-->

## 1. Arrays

Arrays：处理对象是数组。

* 排序：对数组排序。sort()，parallelSort()。
* 查找：从数组中查找一个元素。binarySearch()。
* 批量拷贝：从源数组批量复制元素到目标数组。copyOf()。
* 批量赋值：对数组进行批量赋值。fill()。
* 等价性比较：判定两个数组内容是否相同。equals()。

Arrays中的方法有非常多，上面只是列出了几个最常用的，了解更多可以看官方API文档。

下面通过代码示例和其输出结果，来了解Arrays的基本使用。

```java
import java.util.Arrays;
import java.util.Random;

public class ArraysTest {
    public static void main(String[] args) {
        testSort();
        testSearch();
        testCopy();
        testFill();
        testEquality();
    }

    public static void testSort() {
        Random r = new Random();
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++) {
            a[i] = r.nextInt();
        }
        System.out.println("===============测试排序================");
        System.out.println("排序前");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i] + ",");
        }
        System.out.println();
        System.out.println("排序后");
        Arrays.sort(a);
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i] + ",");
        }
        System.out.println();
    }

    public static void testSearch() {
        Random r = new Random();
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++) {
            a[i] = r.nextInt();
        }
        System.out.println();
        a[a.length - 1] = 10000;
        System.out.println("===========测试查找============");

        // Array.binarySearch() 说明
        //  参数：
        //    a - 要搜索的数组
        //    key - 要搜索的值
        //  返回：
        //    如果它包含在数组中，则返回搜索键的索引；否则返回 (-(插入点) - 1)。
        //    插入点 被定义为将键插入数组的那一点：即第一个大于此键的元素索引，如果数组中的所有元素都小于指定的键，则为 a.length。
        //    注意，这保证了当且仅当此键被找到时，返回的值将 >= 0。

        Arrays.sort(a); // 注意 binarySearch是二分查找，要求数组有序
        System.out.println("10000 的位置是" + Arrays.binarySearch(a, 10000));
    }

    public static void testCopy() {
        Random r = new Random();
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++) {
            a[i] = r.nextInt();
        }
        int[] b = Arrays.copyOf(a, 5);
        System.out.println("===========测试拷贝前五个元素============");
        System.out.print("源数组：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i] + ",");
        }
        System.out.println();
        System.out.print("目标数组：");
        for (int i = 0; i < b.length; i++) {
            System.out.print(b[i] + ",");
        }
        System.out.println();
    }

    public static void testFill() {
        int[] a = new int[10];
        Arrays.fill(a, 100); // a数组全部赋值为100
        Arrays.fill(a, 2, 8, 200); // a数组索引从2到7，赋值为200
        System.out.println("===========测试批量赋值============");
        System.out.print("数组赋值后：");
        for (int i = 0; i < a.length; i++) {
            System.out.print(a[i] + ",");
        }
        System.out.println();
    }

    public static void testEquality() {
        int[] a = new int[10];
        Arrays.fill(a, 100);
        int[] b = new int[10];
        Arrays.fill(b, 100);
        System.out.println(Arrays.equals(a, b));
        b[9] = 200;
        System.out.println(Arrays.equals(a, b));
    }
}
```

```
===============测试排序================
排序前
1283461119,-143705604,-574266644,692863819,1643772367,990026094,2104945760,-2045200996,30775867,-1303441633,
排序后
-2045200996,-1303441633,-574266644,-143705604,30775867,692863819,990026094,1283461119,1643772367,2104945760,

===========测试查找============
10000 的位置是5
===========测试拷贝前五个元素============
源数组：-1775375895,1966660387,1093093070,-1979042230,253104600,1010284039,939092092,-195636571,2063264742,-1730020008,
目标数组：-1775375895,1966660387,1093093070,-1979042230,253104600,
===========测试批量赋值============
数组赋值后：100,100,200,200,200,200,200,200,100,100,
true
false
```





## 2. Collections

Collections：处理对象是Collection及其子类，主要聚焦在对List的处理上。

* 排序：对List进行排序。sort()。
* 搜索：从List中搜索元素。binarySearch()。
* 批量赋值：对List批量赋值。fill()。
* 最大/最小：查找集合中最大/最小值。max()，min()。
* 逆置：将List逆置。reverse()。

Collections中的方法有非常多，上面只是列出了几个最常用的，了解更多可以看官方API文档。

下面通过代码示例和其输出结果，来了解Collections的基本使用。

```java
import java.util.ArrayList;
import java.util.Collections;

public class CollectionsTest {

    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(1);
        list.add(12);
        list.add(2);
        list.add(19);

        // 排序
        Collections.sort(list);
        // 检索
        System.out.println("元素所在的索引值是：" + Collections.binarySearch(list, 12));
        //最大最小
        System.out.println("最大值：" + Collections.max(list));
        System.out.println("最小值：" + Collections.min(list));
        Collections.reverse(list); //翻转不需要用到排序

        Collections.fill(list, 100); //全部赋值为100
    }
}
```

```
元素所在的索引值是：2
最大值：19
最小值：1
```





## 3. 对象比较

前面Arrays或Collections都会对数组或集合里面的元素进行排序。

如果数组或集合里面的元素都是数字或Integer对象，都可以从小到大正常排序。

如果数组或集合里面的对象都是普通对象、自定义对象，则对其排序需要实现Comparable接口。



普通对象比较的两种方法

* 在对象类中**实现Comparable接口，需要修改对象类**。

    Comparable接口中只有一个方法需要实现类实现，compareTo()方法，Arrays和Collections在进行对象sort时，会自动调用该方法。

    **compareTo()：>返回1，==返回0，<返回-1。**

* 若**对象类不可修改，可以自动以一个比较器，比较器实现Comparator<T, T>接口。**。

    在比较器类中定义compare()方法。

    **compare()：同样是 >返回1，==返回0，<返回-1。**



Java工具类中的排序，个人理解为都是按从小到大排序。

假设有两个对象比较`obj1.compareTo(obj2)`，如果我们想让obj1在obj2前面，就要让这个函数返回-1，即认定obj1小于obj2。同样的，如果想要对普通的数值类对象从大到小排序（默认为从小到大），只要反着定义compareTo()函数就可以了。自定义比较器中的compare()同理。

下面通过具体代码示例，来理解两种对象比较方法。

### 3.1. 实现Comparable接口的方法

```java
import java.util.Arrays;

public class Person implements Comparable<Person> {
    String name;
    int age;

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public int compareTo(Person another) {
        int i = 0;
        i = name.compareTo(another.name); // 使用字符串的比较
        if (i == 0) {
            // 如果名字一样,比较年龄, 返回比较年龄结果
            return age - another.age;
        } else {
            return i; // 名字不一样, 返回比较名字的结果.
        }
    }

    public static void main(String... a) {
        Person[] ps = new Person[3];
        ps[0] = new Person("Tom", 20);
        ps[1] = new Person("Mike", 18);
        ps[2] = new Person("Mike", 20);

        Arrays.sort(ps);
        for (Person p : ps) {
            System.out.println(p.getName() + "," + p.getAge());
        }
    }
}
```

```
Mike,18
Mike,20
Tom,20
```

上面的Person类中实现了Comparable接口，定义了compareTo()方法。

compareTo()方法中，设定按人名大小（String大小）排序，同名则按年龄从小到大排序。



### 3.2. 自定义比较器的方法

在下面的Person2类中，没有实现了Comparable接口，因此若要比较，需要自定义一个比较器类。

可以看到输出结果与3.1.中的方法相同，具体写法查看代码即可。

```java
public class Person2 {
    private String name;
    private int age;
    public String getName() {
        return name;
    }
    public int getAge() {
        return age;
    }

    public Person2(String name, int age)
    {
        this.name = name;
        this.age = age;
    }
}
```

```java
import java.util.Arrays;
import java.util.Comparator;

public class Person2Comparator implements Comparator<Person2> {
    public int compare(Person2 one, Person2 another) {
        int i = 0;
        i = one.getName().compareTo(another.getName());
        if (i == 0) {
            // 如果名字一样,比较年龄,返回比较年龄结果
            return one.getAge() - another.getAge();
        } else {
            return i; // 名字不一样, 返回比较名字的结果.
        }
    }

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Person2[] ps = new Person2[3];
        ps[0] = new Person2("Tom", 20);
        ps[1] = new Person2("Mike", 18);
        ps[2] = new Person2("Mike", 20);

        Arrays.sort(ps, new Person2Comparator()); // 给sort()传入一个比较器对象
        for (Person2 p : ps) {
            System.out.println(p.getName() + "," + p.getAge());
        }
    }
}
```

```
Mike,18
Mike,20
Tom,20
```

