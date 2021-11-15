---
title: Java集合Set类
mathjax: false
date: 2020-03-20 19:17:05
updated: 2020-03-20 19:17:05
tags: [Java,JavaSE,JCF,Set,HashSet,TreeSet,LinkedHashSet,Collection,数据结构,容器]
categories: [编程语言基础]
toc: true
---

集合Set

* 确定性：对任意对象都能判定其是否属于某一个集合。
* 互异性：集合内每个元素都是互不相同的，注意是内容互异。
* 无序性：集合内的顺序无关。



Java中的集合接口Set

* HashSet：基于散列函数的集合，无序，不支持同步。
* TreeSet：基于树结构的集合，可排序的，不支持同步。
* LinkedHashSet：基于散列函数和双向链表的集合，可排序的，不支持同步。


<!--more-->
## 1. HashSet

HashSet

* 基于HashMap实现的，可以容纳null元素，不支持同步。

    可以通过`Set s = Collections.synchronizedSet(new HashSet(...))`获取一个支持同步的HashSet。

* add() 添加一个元素。

* clear() 清空整个HashSet。

* contains() 判定是否包含一个元素。

* remove() 删除一个元素。

* size() 集合大小，即元素个数。

* retainAll() 计算两个集合交集。 



下面通过示例代码和其输出结果，来演示HashSet的使用。

下面的代码中还测试了HashSet使用迭代器和使用for-each遍历的性能，for-each性能更好。

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;

public class HashSetTest {
    public static void main(String[] args) {
        HashSet<Integer> hs = new HashSet<Integer>();
        hs.add(null);
        hs.add(1000);
        hs.add(20);
        hs.add(3);
        hs.add(4);
        hs.add(5000000);
        hs.add(3);                      //3 重复
        hs.add(null);                   //null重复
        System.out.println(hs.size());  //6
        if (!hs.contains(6)) {
            hs.add(6);
        }
        System.out.println(hs.size());  //7
        hs.remove(4);
        System.out.println(hs.size());  //6
        //hs.clear();
        //System.out.println(hs.size());  //0

        System.out.println("============for循环遍历==============");
        for (Integer item : hs) {
            System.out.println(item);
        }

        System.out.println("============测试集合交集==============");

        HashSet<String> set1 = new HashSet<String>();
        HashSet<String> set2 = new HashSet<String>();

        set1.add("a");
        set1.add("b");
        set1.add("c");

        set2.add("c");
        set2.add("d");
        set2.add("e");

        //交集
        set1.retainAll(set2); // set1中不在set2中的元素将被删除
        System.out.println("交集是 " + set1);

        System.out.println("============测试多种遍历方法速度==============");

        HashSet<Integer> hs2 = new HashSet<Integer>();
        for (int i = 0; i < 100000; i++) {
            hs2.add(i);
        }
        traverseByIterator(hs2);
        traverseByFor(hs2);
    }

    public static void traverseByIterator(HashSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = hs.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(HashSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : hs) {
            ;
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
6
7
7
============for循环遍历==============
null
40000
3
20
6
1000
5000000
============测试集合交集==============
交集是 [c]
============测试多种遍历方法速度==============
============迭代器遍历==============
7311791纳秒
============for循环遍历==============
6130979纳秒
```



## 2. LinkedHashSet

LinkedHashSet

* 继承HashSet，也是基于HashMap实现的，可以容纳null元素。

* 不支持同步

    同样可以通过`Set s = Collections.synchronizedSet(new LinkedHashSet(...))`获取一个支持同步的LinkedHashSet。

* 方法和HashSet基本一致。

    add()，clear()，contains()，remove()，size()等。

* 通过一个**双向链表**维护插入顺序。



下面通过示例代码和其输出结果，来演示LinkedHashSet的使用。

LinkedHashSet用法与HashSet几乎完全一致，仅仅只是保留了插入的顺序。

下面的代码中还测试了LinkedHashSet使用迭代器和使用for-each遍历的性能，for-each性能更好。

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashSet;

public class LinkedHashSetTest {
    public static void main(String[] args) {
        LinkedHashSet<Integer> lhs = new LinkedHashSet<Integer>();
        lhs.add(null);
        lhs.add(1000);
        lhs.add(20);
        lhs.add(3);
        lhs.add(4);
        lhs.add(5000000);
        lhs.add(3);                      //3 重复
        lhs.add(null);                   //null 重复
        System.out.println(lhs.size());  //6
        if (!lhs.contains(6)) {
            lhs.add(6);
        }
        System.out.println(lhs.size());  //7
        lhs.remove(4);
        System.out.println(lhs.size());  //6
        //lhs.clear();
        //System.out.println(lhs.size());  //0

        System.out.println("============for循环遍历==============");
        for (Integer item : lhs) {
            System.out.println(item);
        }

        LinkedHashSet<Integer> lhs2 = new LinkedHashSet<Integer>();
        for (int i = 0; i < 100000; i++) {
            lhs2.add(i);
        }
        traverseByIterator(lhs2);
        traverseByFor(lhs2);

    }

    public static void traverseByIterator(LinkedHashSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = hs.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(LinkedHashSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : hs) {
            ;
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
6
7
6
============for循环遍历==============
null
1000
20
3
5000000
6
============迭代器遍历==============
7966941纳秒
============for循环遍历==============
4119412纳秒
```

## 3. TreeSet

TreeSet

* 基于TreeMap实现的，**不可以容纳null元素**，不支持同步。

    可以通过`SortedSet s = Collectons.synchronizedSortedSet(new TreeSet(...))` 得到一个支持同步的TreeSet。

* add() 添加一个元素。

* clear() 清空整个TreeSet。

* contains() 判定是否包含一个元素。

* remove() 删除一个元素。

* size() 返回集合中元素的个数。

* **根据compareTo方法或指定Comparator排序。**



下面通过示例代码和其输出结果，来演示TreeSet的使用。

下面的代码中还测试了TreeSet使用迭代器和使用for-each遍历的性能，for-each性能更好。

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashSet;
import java.util.TreeSet;

public class TreeSetTest {
    public static void main(String[] args) {
        TreeSet<Integer> ts = new TreeSet<Integer>();
        // ts.add(null);  错误，不支持null
        ts.add(1000);
        ts.add(20);
        ts.add(3);
        ts.add(4);
        ts.add(5000000);
        ts.add(3);                      //3 重复
        System.out.println(ts.size());  //5
        if (!ts.contains(6)) {
            ts.add(6);
        }
        System.out.println(ts.size());  //6
        ts.remove(4);
        System.out.println(ts.size());  //5
        //lhs.clear();
        //System.out.println(lhs.size());  //0

        System.out.println("============for循环遍历==============");
        for (Integer item : ts) // 这里顺序是从小到大
        {
            System.out.println(item);
        }

        TreeSet<Integer> ts2 = new TreeSet<Integer>();
        for (int i = 0; i < 100000; i++) {
            ts2.add(i);
        }
        traverseByIterator(ts2);
        traverseByFor(ts2);

    }

    public static void traverseByIterator(TreeSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============迭代器遍历==============");
        Iterator<Integer> iter1 = hs.iterator();
        while (iter1.hasNext()) {
            iter1.next();
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }

    public static void traverseByFor(TreeSet<Integer> hs) {
        long startTime = System.nanoTime();
        System.out.println("============for循环遍历==============");
        for (Integer item : hs) {
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
6
5
============for循环遍历==============
3
6
20
1000
40000
5000000
============迭代器遍历==============
8215442纳秒
============for循环遍历==============
6403116纳秒
```





## 4. HashSet/LinkedHashSet/TreeSet的异同

可以存放的内容

* HashSet，LinkedHashSet和TreeSet中的元素都只能是对象。

* HashSet和LinkedHashSet可以存放null，TreeSet则不可以。

判定元素重复的原则

* HashSet和LinkedHashSet判定元素重复的原则

    * 判定两个元素的hashCode是否相同，若不同，返回false。

    * 若两者hashCode相同，判定equals()方法，若不同，返回false；否则返回true。

        可以通过重写hashCode()和equals()方法让期望的两个对象相等，通常同时修改hashCode()，equals()和toStirng()，因为hasdCode()和equals()相同，toString()也应该相同。

    * hashCode和equals()方法是所有类都有的，因为Object类有。

* TreeSet判定元素重复的原则

    * **需要元素实现Comparable接口。**
    * 比较两个元素的compareTo()方法。





下面通过代码示例，来理解三种集合的重复判定。

我们先定义3个简单的类，Cat、Dog和Tiger，用于做HashSet和LinkedHashSet，还有TreeSet的判重实验。

```java
class Cat
{
	  private int size;
	
	  public Cat(int size)
	  {
	    	this.size = size;
  	}
}
```

```java
class Dog {
    private int size;

    public Dog(int s) {
        size = s;
    }

    public int getSize() {
        return size;
    }

    public boolean equals(Object obj2) {
        System.out.println("Dog equals()~~~~~~~~~~~");
        if (0 == size - ((Dog) obj2).getSize()) {
            return true;
        } else {
            return false;
        }
    }

    public int hashCode() {
        System.out.println("Dog hashCode()~~~~~~~~~~~");
        return size;
    }

    public String toString() {
        System.out.print("Dog toString()~~~~~~~~~~~");
        return size + "";
    }
}
```

```java
public class Tiger implements Comparable{
    private int size;

    public Tiger(int s) {
        size = s;
    }

    public int getSize() {
        return size;
    }

    public int compareTo(Object o) {
        System.out.println("Tiger compareTo()~~~~~~~~~~~");
        return size - ((Tiger) o).getSize();
    }
}
```



下面通过在集合中存放Cat类或Dog类或Tiger类的对象，来观察判重的结果。

```java
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashSet;
import java.util.TreeSet;


public class ObjectHashSetTest {

    public static void main(String[] args) {
        System.out.println("==========Cat HashSet ==============");
        HashSet<Cat> hs = new HashSet<Cat>();
        hs.add(new Cat(2));
        hs.add(new Cat(1));
        hs.add(new Cat(3));
        hs.add(new Cat(5));
        hs.add(new Cat(4));
        hs.add(new Cat(4));
        System.out.println(hs.size());  //6

        System.out.println("========================");
        LinkedHashSet<Cat> lhs = new LinkedHashSet<Cat>();
        lhs.add(new Cat(2));
        lhs.add(new Cat(1));
        lhs.add(new Cat(3));
        lhs.add(new Cat(5));
        lhs.add(new Cat(4));
        lhs.add(new Cat(4));
        System.out.println(lhs.size());  //6


        System.out.println("==========Dog HashSet ==============");
        HashSet<Dog> hs2 = new HashSet<Dog>();
        hs2.add(new Dog(2));
        hs2.add(new Dog(1));
        hs2.add(new Dog(3));
        hs2.add(new Dog(5));
        hs2.add(new Dog(4));
        hs2.add(new Dog(4));
        System.out.println(hs2.size());  //5

        System.out.println("========================");
        LinkedHashSet<Dog> lhs2 = new LinkedHashSet<Dog>();
        lhs2.add(new Dog(2));
        lhs2.add(new Dog(1));
        lhs2.add(new Dog(3));
        lhs2.add(new Dog(5));
        lhs2.add(new Dog(4));
        lhs2.add(new Dog(4));
        System.out.println(lhs2.size());  //5


        System.out.println("==========Tiger HashSet ==============");
        HashSet<Tiger> hs3 = new HashSet<Tiger>();
        hs3.add(new Tiger(2));
        hs3.add(new Tiger(1));
        hs3.add(new Tiger(3));
        hs3.add(new Tiger(5));
        hs3.add(new Tiger(4));
        hs3.add(new Tiger(4));
        System.out.println(hs3.size());  //6

        System.out.println("========================");
        LinkedHashSet<Tiger> lhs3 = new LinkedHashSet<Tiger>();
        lhs3.add(new Tiger(2));
        lhs3.add(new Tiger(1));
        lhs3.add(new Tiger(3));
        lhs3.add(new Tiger(5));
        lhs3.add(new Tiger(4));
        lhs3.add(new Tiger(4));
        System.out.println(lhs3.size());  //6


        /*
        // Cat和Dog类没有实现接口Comparable，没有compareTo()，不能存在TreeSet中。
		System.out.println("==========Cat TreeSet ==============");
		TreeSet<Cat> ts = new TreeSet<Cat>();
		ts.add(new Cat(2));
		ts.add(new Cat(1));
		ts.add(new Cat(3));
		ts.add(new Cat(5));
		ts.add(new Cat(4));
		ts.add(new Cat(4));
		System.out.println(ts.size());  //5

		System.out.println("==========Dog TreeSet ==============");


		TreeSet<Dog> ts2 = new TreeSet<Dog>();
		ts2.add(new Dog(2));
		ts2.add(new Dog(1));
		ts2.add(new Dog(3));
		ts2.add(new Dog(5));
		ts2.add(new Dog(4));
		ts2.add(new Dog(4));
		System.out.println(ts2.size());  //5
		*/

        //添加到TreeSet的，需要实现Comparable接口，即实现compareTo方法

        System.out.println("==========Tiger TreeSet ==============");


        TreeSet<Tiger> ts3 = new TreeSet<Tiger>();
        ts3.add(new Tiger(2));
        ts3.add(new Tiger(1));
        ts3.add(new Tiger(3));
        ts3.add(new Tiger(5));
        ts3.add(new Tiger(4));
        ts3.add(new Tiger(4));
        System.out.println(ts3.size());  //5
    }
}
```

```
==========Cat HashSet ==============
6
========================
6
==========Dog HashSet ==============
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog equals()~~~~~~~~~~~
5
========================
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog hashCode()~~~~~~~~~~~
Dog equals()~~~~~~~~~~~
5
==========Tiger HashSet ==============
6
========================
6
==========Tiger TreeSet ==============
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
Tiger compareTo()~~~~~~~~~~~
5
```

观察上面代码及输出结果可以发现。

* 在HashSet/LinkedHashSet中，Cat类虽然size相同，但却被认为没有重复，因为两个Cat对象的hashCode是不同的。

* 在HashSet/LinkedHashSet中，因为Dog类重写了hashCode()函数，两个size相同的Dog对象的hashCode相等，所以再判断equals()，因为equals也重写了，判定相等。所以最终这两个相同size的Dog对象被判重。通常来说我们重写了hashCode()和equals()，也应当重写toString()。hashCode()、equals()和toString()应该都是一样的。

* Tiger实现Comparable接口，所以必须实现compareTo()方法来比较大小。

    CompareTo()方法具体规则如下：

    `int a = obj1.compareTo(obj2);`

    如果 a > 0，则 obj1 > obj2;

    如果 a == 0，则 obj1 == obj2；

    如果 a < 0，则 obj1 < obj2。

* HashSet/LinkedHashSet只关注hashCode和equals()，不关注compareTo()；

    TreeSet只关注compareTo()，不关注hashCode和equals()。

