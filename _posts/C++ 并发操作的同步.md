---
title: "C++ 并发操作的同步"
date: 2023-02-01 18:05:11
updated: 2023-02-01 23:12:11
categories: [并发编程]
tags: [Cpp,并发编程]
---



C++ 中并发操作的同步手段主要有三种，按复杂程度从低到高依次为：

1. `std::async`；
2. `std::packaged_task`；
3. `std::promise`。



其中 `std::async`是函数模板，另外两个为类模板。三者都定义在头文件 `<future>` 中，且由**不可拷贝的** `std::future` 类型的对象接收事件的结果。



我们逐个介绍上述三种，并在其中顺带演示如何使用 `std::future` 对象。



## 1. `std::async`

最简单的方法，`std::async` 函数接收 1 个可调用对象（通常是函数）以及此调用对象的参数。例如：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::future<int> f = std::async(sum, 3, 4);
    std::cout << f.get() << std::endl; // print "7"
    
    return 0;
}
```

\-

第二种方法用于执行类的成员函数，接收要执行的成员函数、要执行该成员函数的对象（静态成员函数不需要）以及相关参数。例如：

```cpp
#include <iostream>
#include <future>

class ForSum {
public:
    int sum(int a, int b) {
       return a + b;
    }
    static int sum_static(int a, int b) {
        return a + b;
    }
};

int main(int, char**) {
    ForSum fsum;
    std::future<int> f1 = std::async(&ForSum::sum, &fsum, 3, 4);
    std::future<int> f2 = std::async(&ForSum::sum, fsum, 3, 4);
    std::future<int> f3 = std::async(&ForSum::sum_static, 3, 4);  // static member function
    std::cout << f1.get() << std::endl;  // print "7"
    std::cout << f2.get() << std::endl;  // print "7"
    std::cout << f3.get() << std::endl;  // print "7"

    return 0;
}
```

这里注意下 `f1` 和 `f2` 对应的参数 `&fsum` 和 `fsum` 不同，前者使用的是我们自己声明的对象 `fsum` 执行成员函数，而后者则是其副本。



\-

`std::async` 还可以指定执行指定函数的时机，在参数列表的**首位**添加一个参数，类型是 `std::launch`，其值可以是：

* `std::launch::deferred`: 在**当前线程上**延后调用任务函数，等到在 `std::future` 对象上调用了 `get()` 或 `wait()`，任务函数才会执行。
* `std::launch::async`：必须开启专属的线程，在其上运行任务函数。
* `std::launch::deferred | std::launch::async`：缺省值，由 `std::sync` 的实现自己选择运行方式。

举个例子，此例中任务函数 `sum()` 在调用 `f.get()` 时才开始执行：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::future<int> f = std::async(std::launch::deferred ,sum, 3, 4);
    std::cout << f.get() << std::endl; // print "7"
    
    return 0;
}
```



## 2. `std::packaged_task`

从 `std::packaged_task` 的名字可以看出，此类对象是一个打包好的任务，这里这个任务实则就是一个函数。

`std::packaged_task` 用法很简单，其模板参数是一个函数签名。例如，我们想打包一个含有两个类型分别为 `std::vector<char>*` 和 `int` 的参数，返回值类型为 `std::string` 的函数，那么实例化对象时，写法如下（关注 `<>` 中的 `std::string(std::vector<char>*, int)`）：

```cpp
std::packaged_task<std::string(std::vector<char>*, int)> task(f);
```

其中 `f` 为要打包的具体函数名（函数指针）。

注意上面的实例化对象声明时没有传入被打包函数的参数，被打包函数的参数将在调用 `task` 时传入。

`task` 的调用和函数直接调用 `f` 看起来区别不大，举一个具体的例子：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    task(2, 3);
    sum(2, 3);
    
    return 0;
}
```

上面的 `task(2, 3)` 和 `sum(2, 3)` 都是计算 2+3 的和，那区别是什么呢？

主要区别在于，**`std::packaged_task` 打包后，函数的调用和获取返回值可以在不同的线程中进行。**`sum(2, 3)` 的返回值 5 可以直接获取，并且函数的调用和获取返回值都是在当前线程中进行的。而 `task(2, 3)` 的调用，需要使用其成员函数 `get_future()` 获取 `std::future` 对象，通过此对象获取返回值，`std::future` 对象的特化类型取决于 `std::packaged_task` 对象声明时参数中函数签名指定的返回值。

我们扩展上面的例子：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    std::future f = task.get_future();
    task(2, 3);
    std::cout << "task(2, 3) 的返回值为 " << f.get() << std::endl;
    std::cout << "sum(2, 3) 的返回值为 " << sum(2, 3) << std::endl;  // sum(2, 3) 的返回值可以直接获取
    
    return 0;
}
```

输出如下：

```
task(2, 3) 的返回值为 5
sum(2, 3) 的返回值为 5
```

可以看到，我们使用 `std::future` 类型对象来接收 `task(2, 3)` 的结果。

但注意，此例子中，我们 `task(2, 3)` 的执行和结果获取仍是在同一个线程中进行的，这并没有发挥出其优势，仅用于举例。

下面的代码中，我们在另一个线程中执行此任务，在当前线程中获取并打印结果：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    std::cout << "starting calculatation, the thread id is " 
              << std::this_thread::get_id() << std::endl;
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    std::future f = task.get_future();
    std::thread t(std::move(task), 2, 3);
    t.join();
    std::cout << "the result of task(2, 3) is " << f.get()
              << ", and the thread id is " 
              << std::this_thread::get_id() << std::endl;
    
    return 0;
}
```

输出：

```
starting calculatation, the thread id is 139965868611328
the result of task(2, 3) is 5, and the thread id is 139965886650176
```

可以看到，我们的计算线程和获取结果的线程是不同的。

\-



在上面关于 `std::async` 和 `std::packaged_task` 的例子中，我们演示了二者基本的使用。

有些读者可能会有和我一样的疑问，这两者似乎做了一样的事情（其一可以做的事情，另一个也都能做），但 `std::packaged_task` 更复杂些。这是为什么呢？

我的理解是，`std::packaged_task` 比 `std::async` 能多做的有：





* `std::packaged_task` 可由程序员决定任务何时开始（调用 `task()` 才开始），而 `std::async` 是由其内部实现决定的，程序员不可控。
* `std::packaged_task`可以指定一个特定线程来完成任务（对象可以移动，就可以移动给指定线程，由指定线程执行）。而 `std::async` 要么是当前线程，要么是一个新线程。







本文也主要是笔记，有理解不全或不对的地方，欢迎指出！



如果你不需要 `std::packaged_task`  额外的功能，那么 `std::async`  更简单。下面要说的 `std::promise` 比 `std::packaged_task` 能做的要更多。

## 3. `std::promise`

