
数组只能线性存储同一种类型的数据。

## 1. 一维数组

### 1.1. 声明方式 

注意Java中声明时是没有分配内存的，所以不需要指定大小，在new的时候才指定。

```java
int arr1[]; // 方法一
int[] arr2; // 方法二（常用）
```
<!--more-->
### 1.2. 初始化

数组一旦初始化，其长度是不可变的。

```java
// 动态初始化：在声明的时候指定数组长度，分配内存。
int[] arr1 = new int[3]; // 此时会有默认值，int 类型默认值为 0，对象的默认值是 null
arr1[0] = 1;
arr1[1] = 2;
arr1[2] = 3;
// 静态初始化：初始化时由程序猿显示指定每个数组元素的初始值，由系统决定数组的长度。
int[] arr2 = new int[]{1, 2, 3};
int[] arr3 = {1, 2, 3};
```

1. 数组元素的引用：下标索引如 `arr[2]`，与 C/C++ 相同。
2. 每个数组有一个属性 length 表示其长度。



### 1.3. 一维数组遍历

```java
// 有一维数组 d
//需要自己控制索引位置
for(int i=0;i<d.length;i++)	{
    System.out.println(d[i]);
}
		
//无需控制索引位置
for(int e : d) {
    System.out.println(e);
}
```





## 2. 多维数组

### 2.1. 声明方式

```java
int[][] arr1; // 常用此方法
int arr2[][];
int[] arr3[];
```

### 2.2. 初始化

```java
// 动态初始化：规则数组，类比一维，第二维长度可省略（Java中最后一维的长度可以不同）
int[][] arr1 = new int[2][3];
// 静态初始化，不规则数组
int[][] arr2 = new int[][] {{1,2,3}, {2,7}, {4,5,6}};

// 动态初始化，不规则数组
int b[][];
b = new int[3][];
b[0]=new int[3];
b[1]=new int[4];
b[2]=new int[5];
```

### 2.3. 多维数组遍历

```JAVA
for(int i=0;i<a.length;i++)
{
    for(int j=0;j<a[i].length;j++)
    {
        a[i][j] = ++k; 
    }
}

for(int[] items : a)
{
    for(int item : items)
    {
        System.out.print(item + ", ");
    }
    System.out.println();
}
```

