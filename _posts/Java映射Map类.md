
Map映射

* 数学定义：两个集合之间的元素对应关系。
* 一个输入对应到一个输出。
* {1, 张三}，{2, 李四}，{Key, Value}，即键值对，K-V对。

Java中的Map

* **HashMap：**不支持同步，快，数据量大。

* Hashtable：同步，慢，数据量小。

* LinkedHashMap：基于双向链表的维持插入顺序的HashMap。

* TreeMap：基于红黑树的Map，可以根据key的自然排序或者compareTo()方法进行排序输出。

* Properties：同步，文件形式，数据量小。

    Properties也比较老，但是还有些程序使用，因为这是**唯一可以把K-V对保存到文件中的类**。

<!--more-->

HashMap遍历是无序的；

LinkedHashMap遍历的顺序和它插入的顺序保持一致；

TreeMap遍历的顺序是按照大小或者compareTo()方法规定的。



## 1. HashMap

HashMap

* K-V对，K和V都允许为null。

* 不同步，线程不安全。

    可以通过`Map m = Collections.synchronizedMap(new HashMap(...));`获取一个线程安全的HashMap。

* HashMap是无序的。

* 主要方法

    clear() 清空HashMap。

    containsKey() 检查JHashMap中是否有某个Key。

    containsValue() 检查HashMap中是否有某个Value。

    get() 通过Key获取Value。

    put() 增加新的K-V对。

    remove() 删除某个K-V对。

    size() 返回HashMap中K-V对的个数。



下面通过观察代码示例与输出结果，了解HashMap的基本用法。

Hashtable有2种遍历办法，根据Entry迭代器遍历、**根据Key的Iterator遍历**和根据遍历，根据Key的Iterator遍历性能更好。

HashMap的Key和Value都可以为null。

```java
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;

public class HashMapTest {

    public static void main(String[] args) {
        HashMap<Integer, String> hm = new HashMap<Integer, String>();
        hm.put(1, null);
        hm.put(null, "abc");
        hm.put(1000, "aaa");
        hm.put(2, "bbb");
        hm.put(30000, "ccc");
        System.out.println(hm.containsValue("aaa"));
        System.out.println(hm.containsKey(30000));
        System.out.println(hm.get(30000));

        hm.put(30000, "ddd");  //更新覆盖ccc
        System.out.println(hm.get(30000));

        hm.remove(2);
        System.out.println("size: " + hm.size());

        hm.clear();
        System.out.println("size: " + hm.size());


        HashMap<Integer, String> hm2 = new HashMap<Integer, String>();
        for (int i = 0; i < 100000; i++) {
            hm2.put(i, "aaa");
        }
        traverseByEntry(hm2);
        traverseByKeySet(hm2);
    }

    public static void traverseByEntry(HashMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============Entry迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = ht.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeySet(HashMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeySet迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Integer> iter = ht.keySet().iterator();
        while (iter.hasNext()) {
            key = iter.next();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
true
true
ccc
ddd
size: 4
size: 0
============Entry迭代器遍历==============
11703124纳秒
============KeySet迭代器遍历==============
8647615纳秒
```





## 2. Hashtable

Hashtable

* K-V对，K和V都不允许为null。

* 同步，多线程安全。

* 无序的。

* 适合小数据量。

* 主要方法

    clear() 清空Map。

    contains() / containsValue() 检查是否包含某个值Value。

    containsKey() 检查是否包含某个键Key。

    get() 根据key获取相应的值。

    put() 增加新的K-V对。

    remove() 删除某个K-V对。

    size() 返回Map内键值对个数。



下面通过观察代码示例与输出结果，了解Hashtable的基本用法。

Hashtable有三种遍历办法，根据Entry迭代器遍历、根据Key的Iterator遍历和根据Key的Enumeration（很老的方法）遍历。

我做了多次实验，三种遍历方法的效率相差无几，但**Iterator**的方法更新，功能更多，建议使用。

```java
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;

public class HashtableTest {

    public static void main(String[] args) {
        Hashtable<Integer, String> ht = new Hashtable<Integer, String>();
        //ht.put(1, null); 编译不报错  运行报错
        //ht.put(null,1);  编译报错
        ht.put(1000, "aaa");
        ht.put(2, "bbb");
        ht.put(30000, "ccc");
        System.out.println(ht.contains("aaa"));
        System.out.println(ht.containsValue("aaa"));
        System.out.println(ht.containsKey(30000));
        System.out.println(ht.get(30000));

        ht.put(30000, "ddd");  //更新覆盖ccc
        System.out.println(ht.get(30000));

        ht.remove(2);
        System.out.println("size: " + ht.size());

        ht.clear();
        System.out.println("size: " + ht.size());


        Hashtable<Integer, String> ht2 = new Hashtable<Integer, String>();
        for (int i = 0; i < 100000; i++) {
            ht2.put(i, "aaa");
        }
        traverseByEntry(ht2);
        traverseByKeySet(ht2);
        traverseByKeyEnumeration(ht2);
    }

    public static void traverseByEntry(Hashtable<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============Entry迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = ht.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeySet(Hashtable<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeySet迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Integer> iter = ht.keySet().iterator();
        while (iter.hasNext()) {
            key = iter.next();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeyEnumeration(Hashtable<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeyEnumeration迭代器遍历==============");
        Integer key;
        String value;
        Enumeration<Integer> keys = ht.keys();
        while (keys.hasMoreElements()) {
            key = keys.nextElement();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
true
true
true
ccc
ddd
size: 2
size: 0
============Entry迭代器遍历==============
11808138纳秒
============KeySet迭代器遍历==============
14858399纳秒
============KeyEnumeration迭代器遍历==============
8758827纳秒
```





## 3. LinkedHashMap

LinkedHashMap：基于双向链表的维持插入顺序的HashMap。

LinkedHashMap与LinkedHashSet类似。下面通过示例代码，简单了解LinkedHashMap。

LinkedHashMap的Key和Value都可以为null。

```java
import java.util.LinkedHashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;

public class LinkedHashMapTest {

    public static void main(String[] args) {
        LinkedHashMap<Integer, String> hm = new LinkedHashMap<Integer, String>();
        hm.put(1, null);
        hm.put(null, "abc");
        hm.put(1000, "aaa");
        hm.put(2, "bbb");
        hm.put(30000, "ccc");
        System.out.println(hm.containsValue("aaa"));
        System.out.println(hm.containsKey(30000));
        System.out.println(hm.get(30000));

        hm.put(30000, "ddd");  //更新覆盖ccc
        System.out.println(hm.get(30000));

        hm.remove(2);
        System.out.println("size: " + hm.size());

        //hm.clear();
        //System.out.println("size: " + hm.size());

        System.out.println("遍历开始==================");

        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = hm.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            System.out.println("Key:" + key + ", Value:" + value);
        }
        System.out.println("遍历结束==================");

        LinkedHashMap<Integer, String> hm2 = new LinkedHashMap<Integer, String>();
        for (int i = 0; i < 100000; i++) {
            hm2.put(i, "aaa");
        }
        traverseByEntry(hm2);
        traverseByKeySet(hm2);
    }

    public static void traverseByEntry(LinkedHashMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============Entry迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = ht.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeySet(LinkedHashMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeySet迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Integer> iter = ht.keySet().iterator();
        while (iter.hasNext()) {
            key = iter.next();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
true
true
ccc
ddd
size: 4
遍历开始==================
Key:1, Value:null
Key:null, Value:abc
Key:1000, Value:aaa
Key:30000, Value:ddd
遍历结束==================
============Entry迭代器遍历==============
11921612纳秒
============KeySet迭代器遍历==============
9370364纳秒
```



## 4. TreeMap

TreeMap：基于红黑树的Map，可以根据key的自然排序或者compareTo()方法进行排序输出。

TreeMap与TreeSet类似，下面通过示例代码，简单了解TreeMap。

TreeMap的key不可以为null，但是值可以为null。

```java
package Map;

import java.util.TreeMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;

public class TreeMapTest {

    public static void main(String[] args) {
        TreeMap<Integer, String> hm = new TreeMap<Integer, String>();
        hm.put(1, null);
        //hm.put(null, "abc");  编译没错，运行报空指针异常
        hm.put(1000, "aaa");
        hm.put(2, "bbb");
        hm.put(30000, "ccc");
        System.out.println(hm.containsValue("aaa"));
        System.out.println(hm.containsKey(30000));
        System.out.println(hm.get(30000));

        hm.put(30000, "ddd");  //更新覆盖ccc
        System.out.println(hm.get(30000));

        //hm.remove(2);
        System.out.println("size: " + hm.size());

        //hm.clear();
        //System.out.println("size: " + hm.size());

        System.out.println("遍历开始==================");

        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = hm.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            System.out.println("Key:" + key + ", Value:" + value);
        }
        System.out.println("遍历结束==================");

        TreeMap<Integer, String> hm2 = new TreeMap<Integer, String>();
        for (int i = 0; i < 100000; i++) {
            hm2.put(i, "aaa");
        }
        traverseByEntry(hm2);
        traverseByKeySet(hm2);
    }

    public static void traverseByEntry(TreeMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============Entry迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = ht.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeySet(TreeMap<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeySet迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Integer> iter = ht.keySet().iterator();
        while (iter.hasNext()) {
            key = iter.next();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
true
true
ccc
ddd
size: 4
遍历开始==================
Key:1, Value:null
Key:2, Value:bbb
Key:1000, Value:aaa
Key:30000, Value:ddd
遍历结束==================
============Entry迭代器遍历==============
12945810纳秒
============KeySet迭代器遍历==============
18604239纳秒
```





## 5. Properties

Properties

* 继承于Hashtable。

* 可以将K-V对保存在文件中。

* 适用于数据量少的配置文件。

* 主要方法

    继承自Hashtable的：clear()，contains()/containsValue()，containsKey()，get()，put()，remove()，size()。

    load() 从文件中加载所以K-V对。

    store() 写入所有K-V对到文件中。

    getProperty() 获取一个Key对应的Value。

    setProperty() 写入一个K-V对。

下面通过观察代码示例与输出结果，了解Properties的基本用法。

```java
import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Enumeration;
import java.util.Properties;

//关于Properties类常用的操作
public class PropertiesTest {
    //根据Key读取Value
    public static String GetValueByKey(String filePath, String key) {
        Properties pps = new Properties();
        try {
            InputStream in = new BufferedInputStream(new FileInputStream(filePath));
            pps.load(in); //所有的K-V对都加载了
            String value = pps.getProperty(key);
            //System.out.println(key + " = " + value);
            return value;

        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    //读取Properties的全部信息
    public static void GetAllProperties(String filePath) throws IOException {
        Properties pps = new Properties();
        InputStream in = new BufferedInputStream(new FileInputStream(filePath));
        pps.load(in); //所有的K-V对都加载了
        Enumeration en = pps.propertyNames(); //得到配置文件的名字

        while (en.hasMoreElements()) {
            String strKey = (String) en.nextElement();
            String strValue = pps.getProperty(strKey);
            //System.out.println(strKey + "=" + strValue);
        }

    }

    //写入Properties信息
    public static void WriteProperties(String filePath, String pKey, String pValue) throws IOException {
        File file = new File(filePath);
        if (!file.exists()) {
            file.createNewFile();
        }
        Properties pps = new Properties();

        InputStream in = new FileInputStream(filePath);
        //从输入流中读取属性列表（键和元素对）
        pps.load(in);
        //调用 Hashtable 的方法 put。使用 getProperty 方法提供并行性。
        //强制要求为属性的键和值使用字符串。返回值是 Hashtable 调用 put 的结果。
        OutputStream out = new FileOutputStream(filePath);
        pps.setProperty(pKey, pValue);
        //以适合使用 load 方法加载到 Properties 表中的格式，
        //将此 Properties 表中的属性列表（键和元素对）写入输出流
        pps.store(out, "Update " + pKey + " name");
        out.close();
    }

    public static void main(String[] args) throws IOException {
        System.out.println("写入Test.properties================");
        WriteProperties("Test.properties", "name", "12345");

        System.out.println("加载Test.properties================");
        GetAllProperties("Test.properties");

        System.out.println("从Test.properties加载================");
        String value = GetValueByKey("Test.properties", "name");
        System.out.println("name is " + value);
    }
}
```

```
写入Test.properties================
加载Test.properties================
从Test.properties加载================
name is 12345
```

上面代码中在本地创建的文件 `Test.properties` 内容如下。

```properties
#Update name name
#Sat Mar 21 13:22:39 CST 2020
name=12345
```





## 5. HashMap和Hashtable性能对比

从下面 代码可以看到，无论是Entry迭代器遍历，还是KeySet迭代器遍历，HashMap的性能都要高很多。

```java
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;

public class MapCompareTest {

    public static void main(String[] args) {
        int n = 1000000;
        System.out.println("============HashMap=============");
        HashMap<Integer, String> hm = new HashMap<Integer, String>();
        for (int i = 0; i < n; i++) {
            hm.put(i, "aaa");
        }
        traverseByEntry(hm);
        traverseByKeySet(hm);

        System.out.println("============Hashtable=============");
        Hashtable<Integer, String> ht2 = new Hashtable<Integer, String>();
        for (int i = 0; i < n; i++) {
            ht2.put(i, "aaa");
        }
        traverseByEntry(ht2);
        traverseByKeySet(ht2);
    }

    public static void traverseByEntry(Map<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============Entry迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Entry<Integer, String>> iter = ht.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Integer, String> entry = iter.next();
            // 获取key
            key = entry.getKey();
            // 获取value
            value = entry.getValue();
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }


    public static void traverseByKeySet(Map<Integer, String> ht) {
        long startTime = System.nanoTime();
        System.out.println("============KeySet迭代器遍历==============");
        Integer key;
        String value;
        Iterator<Integer> iter = ht.keySet().iterator();
        while (iter.hasNext()) {
            key = iter.next();
            // 获取value
            value = ht.get(key);
            //System.out.println("Key:" + key + ", Value:" + value);
        }
        long endTime = System.nanoTime();
        long duration = endTime - startTime;
        System.out.println(duration + "纳秒");
    }
}
```

```
============HashMap=============
============Entry迭代器遍历==============
26454369纳秒
============KeySet迭代器遍历==============
29925026纳秒
============Hashtable=============
============Entry迭代器遍历==============
36413866纳秒
============KeySet迭代器遍历==============
37679046纳秒
```

