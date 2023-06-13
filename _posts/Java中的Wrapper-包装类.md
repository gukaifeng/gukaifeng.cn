
Java中针对八种基本数据类型定义相应的引用类型——包装类（封装类）。

## 1. 基本数据类型与包装类的对应关系

| 基本数据类型 |  包装类   |
| :----------: | :-------: |
|   boolean    |  Boolean  |
|     btye     |   Byte    |
|    short     |   Short   |
|     int      |  Integer  |
|     long     |   Long    |
|     char     | Character |
|    float     |   Float   |
|    double    |  Double   |

<!--more-->

## 2. 装箱

有了类的特点，就可以调用类中的方法。基本数据类型包装成包装类的实例 —— 装箱

``` java
// 通过包装类的构造器实现
int a = 5;
Integer t = new Integer(a);
// 通过字符串参数构造包装类对象，要求字符串所含内容为对应类型的值
Float f = new Float("4.56");
```

## 3. 拆箱

获得包装类对象中包装的基本类型变量 —— 拆箱

```JAVA
// 调用包装类的 .xxxValue() 方法
boolean b = bObj.booleanValue();
```

JDK1.5 以后，支持自动装箱、拆箱，但类型必须匹配。

```java
int a2 = t;	// 自动拆箱
Integer t2 = 1; // 自动装箱
```

## 4. 字符串转换为基本数据类型

1. 通过包装类的构造器实现

    ```java
    int i = new Integer("12");
    ```

2. 通过包装类的 parseXxx(String s) 静态方法

    ```java
    Float f = Float.parseFloat("12.1");
    ```

## 5. 基本数据类型转换为字符串

1. 调用字符串重载的 valueOf() 方法

    ```java
    String fstr = String.valueOf(2.34f);
    ```

2. 更直接的方法

    ```java
    String intStr = 5 + "";
    ```