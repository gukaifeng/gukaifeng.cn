---
title: C++11 chrono 高精度计时方法
date: 2021-07-21 13:13
categories: [技术杂谈]
tags: [C++]
toc: true
---

# C++11 chrono 高精度计时方法



在学习或开发中，给一段代码计时是一个非常常用的操作。  
在 C++11 以前，类似 time 库等方法的精度是比较低的，C++11提供了一个精度最高为纳秒级的时间库 chrono。  
这篇文章给大家演示如何用 chrono 给一段代码计时。

<!--more-->

使用 chrono 需要引入 C++11 提供的库：  

`#include <chrono>`

chrono 没有像大多 C++ 标准库一样，把库中所有元素都直接放在命名空间 `std`  中，而是把绝大部分库中元素都放在了 `std::chrono` 命名空间中，所以在使用 chrono 时，如果你引用的库中内容比较多，也可以考虑直接 using namespace，本文为了更清晰的演示各个元素的位置，没有 using namespace。

本文主要针对使用 chrono 高精度计时，所以只讲用法，不讲原理，同时对 chrono 提供的其他功能也不作解释。



想一想计时的常规思路是什么！  
1\. 记录被计时的代码段开始前的时间点；  
2\. 执行被计时的代码段；  
3\. 被计时的代码段执行完后，记录其结束时的时间点；  
4\. 用被计时代码段结束时的时间点与开始前的时间点作差，就可以得出被计时代码段的耗时了。



chrono 计时的操作方法也是这个思路，只是代码写起来略复杂一点点。

1\. 记录被计时的代码段开始前的时间点：

```cpp
std::chrono::high_resolution_clock::time_point tp1 = std::chrono::high_resolution_clock::now();
```

* `std::chrono`: 命名空间；
* `high_resolution_clock`: chrono 中的时钟类。一共有三种，另外两个分别是 `system_clock` 和 `steady_clock`，我们这里不关注他们之间的区别，暂时你只需要知道，用来计时的话 `high_resolution_clock` 的精度是最高的；
* `time_point`: 顾名思义，是一个时间点类；
* `now()`: 顾名思义，这个函数会返回此刻时间的时间点。

2\. 执行被计时的代码段：这个自己执行就行，放在第 1、3 点代码之间。我这里演示代码为

```cpp
for (size_t i = 0; i < 1000000000; ++i);
```

3\. 被计时的代码段执行完后，记录其结束时的时间点：

```cpp
std::chrono::high_resolution_clock::time_point tp2 = std::chrono::high_resolution_clock::now();
```

4\. 用被计时代码段结束时的时间点与开始前的时间点作差，就可以得出被计时代码段的耗时了：

```cpp
std::chrono::duration<size_t, std::nano> dur = tp2 - tp1;
```

* `duration`: chrono 中的一个描述持续时间的类，比较复杂，这里的 `<size_t, std::nano> ` 可以简单理解为，duration 对象 dur 用一个 size_t 类型的变量来计数，这个变量的 1 就代表 1/ 1000000000 秒，也就是 1 纳秒，也就是说 dur 对象中记录的持续时间以纳秒为单位。

```cpp
std::cout << "0 被计时代码耗时：" << dur.count() << " 纳秒" << std::endl;
std::cout << "1 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::nanoseconds>(dur).count() << " 纳秒" << std::endl;
std::cout << "2 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::microseconds>(dur).count() << " 微妙" << std::endl;
std::cout << "3 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::milliseconds>(dur).count() << " 毫秒" << std::endl;
std::cout << "4 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::seconds>(dur).count() << " 秒钟" << std::endl;
std::cout << "5 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::minutes>(dur).count() << " 分钟" << std::endl;
std::cout << "6 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::hours>(dur).count() << " 小时" << std::endl;
```

* `duration_cast<std::chrono::microseconds>	`: 上面说过，dur 的计时单位是纳秒(nano)，然后这里的意思是，把精度转换成微妙。可以注意下，0 和 1 两个其实是等价的，因为 dur 的计时单位已经是纳秒了。



好了，计时代码还是比较简单的，我们把上面的代码拼接起来，做一个示例：

```cpp
#include <iostream>
#include <chrono>

int main(int argc, char* argv[]) {

     std::chrono::high_resolution_clock::time_point tp1 = std::chrono::high_resolution_clock::now();

     for (size_t i = 0; i < 1000000000; ++i);

     std::chrono::high_resolution_clock::time_point tp2 = std::chrono::high_resolution_clock::now();

     std::chrono::duration<size_t, std::nano> dur = tp2 - tp1;

     std::cout << "0 被计时代码耗时：" << dur.count() << " 纳秒" << std::endl;
     std::cout << "1 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::nanoseconds>(dur).count() << " 纳秒" << std::endl;
     std::cout << "2 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::microseconds>(dur).count() << " 微妙" << std::endl;
     std::cout << "3 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::milliseconds>(dur).count() << " 毫秒" << std::endl;
     std::cout << "4 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::seconds>(dur).count() << " 秒钟" << std::endl;
     std::cout << "5 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::minutes>(dur).count() << " 分钟" << std::endl;
     std::cout << "6 被计时代码耗时：" << std::chrono::duration_cast<std::chrono::hours>(dur).count() << " 小时" << std::endl;

     return 0;
}
```

看看运行结果：  

```
0 被计时代码耗时：1304920522 纳秒
1 被计时代码耗时：1304920522 纳秒
2 被计时代码耗时：1304920 微妙
3 被计时代码耗时：1304 毫秒
4 被计时代码耗时：1 秒钟
5 被计时代码耗时：0 分钟
6 被计时代码耗时：0 小时
```

可以看到，结果和我们预想的是一样的，0 和 1 是等价的。  
另外，因为被计时代码耗时不足 1 分钟、1 小时，所以分钟和小时这两个精度下的耗时是 0 了。

chrono 是 C++11 提供的一个时间库，其包含的功能是十分强大的，本文的高精度计时仅是简单的一种 chrono 用法，chrono 的其他内容不是本篇文章所关注的。关于 chrono 高精度计时的内容就到这里了，希望能帮到你。

