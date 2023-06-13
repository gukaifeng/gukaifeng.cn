
在Java的for循环中，有人写`for(int i = 0; i < 10; ++i)`，也有人写`for(int i = 0; i < 10; ++i)`。

关于`++i`与`i++`的原理，网络中有很多，也不是这篇文章的重点。

**我们这里只测试下`++i`和`i++`在for循环中的性能差异**，在其他地方（如表达式中）暂不考虑。

以前我在写C/C++的时候，无意间看到过有人说`i++`效率不如`++i`，这也是网络中很普遍的一种说法。至于效率差别的原因，在网络中的解释也很多，这里不多介绍。从那以后，我就一直坚持在写`++i`。

不得不说，虽然写`++i`的有相当一部分人，但主流还是`i++`。

今天看leetcode官方题解的时候，发现都是`i++`，于是又勾起了我一波好奇，就有了下面的测试。

两个测试分别使用`System.nanoTime()`和`System.currentTimeMillis()`计时。

先说结论，**`i++`的时间性能好于`++i`**，与网络中主流结论不一致。

**下面的代码分别在IDEA(2019.3.2)、Eclipse(2019-12 4.14.0)和命令行中做了多次测试，最终结论相同。**

**我的系统是MacOS Mojave(10.14.6)。**

<!--more-->

## 1. System.nanoTime() 计时

测试代码与输出结果如下。

`System.nanoTime()`的单位是纳秒。

```java
public class Main {
    public static void main(String[] args) {
      
        int times = 1000000000;
      
        System.out.println("========== ++i ==========");
        long t1_start = System.nanoTime();
        for (int i = 0; i < times; ++i);
        long t1_end = System.nanoTime();
        System.out.println(t1_end - t1_start);
      
        System.out.println("========== i++ ==========");
        long t2_start = System.nanoTime();
        for (int i = 0; i < times; i++) ;
        long t2_end = System.nanoTime();
        System.out.println(t2_end - t2_start);
    }
}
```

```
========== ++i ==========
2423488
========== i++ ==========
1371845
```

**测试结果与我想的不太一样，居然是`i++`的效率高了`++i`一倍左右。**



## 2. System.currentTimeMillis() 计时

测试代码与输出结果如下。

下面改用更大的`times`，使用`System.currentTimeMillis()`计算时间，单位是毫秒。

```Java
public class Main {
    public static void main(String[] args) {

        long times = 10000000000L;

        System.out.println("========== ++i ==========");
        long t1_start = System.currentTimeMillis();
        for (long i = 0L; i < times; ++i);
        long t1_end = System.currentTimeMillis();
        System.out.println(t1_end - t1_start);

        System.out.println("========== i++ ==========");
        long t2_start = System.currentTimeMillis();
        for (long i = 0L; i < times; i++) ;
        long t2_end = System.currentTimeMillis();
        System.out.println(t2_end - t2_start);
    }
}
```

```
========== ++i ==========
2908
========== i++ ==========
2633
```

**依然是 `i++` 略胜一筹。**

虽然我不明白究竟是什么原因导致了这个性能差距，但是我的实验结论与网络中的主流结论不符。

至少以后在看别人的实验时，不要只看结果，应该自己亲手实验一下试试。

-

**如果有缘，你看到了我的博客，并且知道为何是这样的实验结果，欢迎分享。**

[**点击给我发邮件**](mailto:forjobs@gukaifeng.me)



