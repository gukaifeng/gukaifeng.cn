
JCF: Java collections framework，Java集合框架.



容器：能够存放数据的空间结构。
* 数组/多维数组，只能线性存放。
* 列表/散列表/树/……



容器框架：为表示和操作容器而规定的一种标准体系结构。

* 对外的接口：容器中所能存放的抽象数据类型。
* 接口的实现：可复用的数据结构。
* 算法：对数据的查找和排序。



容器框架的优点：提高数据存取率，避免程序员重复劳动。

典型的容器框架：C++的STL，Java的JCF。

<!--more-->


## 1. JCF层级

![](https://gukaifeng.cn/posts/java-jcf/Java-JCF_1.png)



可以看到上层结点都是接口，叶子结点都是实现类。

算法都是在实现类里面实现的。

上面结构中主要由三个分支，绿色、黄色和天蓝色分支（深蓝色基本上废弃了）。

* 绿色：List列表。
* 黄色：Set散列集。
* 天蓝色：Map映射。

额外的还有两个工具类，分别是Arrays和Collections，帮助我们实现一些数据结构里面的查找、排序等算法。



## 2. JCF的集合接口Collection

JCF集合接口Collection中有一些必须由实现类实现的抽象函数。

* add() 增加一个元素；

* contains() 检查是否包含一个元素；

* remove() 删除一个元素；

* size() 返回集合中元素个数；

* iterator() 迭代器；



## 3. JCF的迭代器接口

JCD的迭代器接口中也有一些必须由实现类实现的抽象函数。

* hasNext() 检查是否有下一个元素。
* next() 获取下一个元素。
* remove() 删除某一个元素。



## 4. JCF主要的数据结构实现类

* 列表（List，ArrayList，LinkedList）
* 集合（Set，HashSet，TreeSet，LinkedHashSet）
* 映射（Map，HashMap，TreeMap，LinkedHashMap）







## 5. JCF主要的算法类

* Arrays：对数组进行查找和排序等操作。
* Collections：对Collection及其子类进行排序和查找操作。

