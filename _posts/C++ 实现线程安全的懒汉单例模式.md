---
title: "C++ 实现线程安全的懒汉单例模式"
date: 2022-09-17 14:56:11
updated: 2022-09-17 23:50:47
categories: [并发编程]
tags: [Cpp,并发编程,设计模式]
---

这篇文章不讲什么是单例模式，什么是饿汉和懒汉，假定读者已经知道这些。

我们知道，最朴素的懒汉单例模式是线程不安全的，这篇文章逐步将其升级，直到实现一个完全线程安全的。

另外，线程安全的懒汉式单例的实现依赖于特定语言，不同的语言有不同的实现，比如 Java 中依赖其虚拟机的实现，而本文要说的 C++ 实现则依赖标准库。

## 1. 朴素的单线程懒汉式单例实现

我们先看一个最朴素的实现：

```cpp
#include <iostream>

class LazySingleton {
    static LazySingleton* instance;
    LazySingleton() {}
    LazySingleton(LazySingleton const &) = delete;
    LazySingleton& operator=(LazySingleton const &) = delete;

public:
    static LazySingleton* get_instance() {
        if (instance == nullptr) {
            instance = new LazySingleton();
        }
        return instance;
    }
};

LazySingleton* LazySingleton::instance;

int main(void) {

    LazySingleton* a = LazySingleton::get_instance();
    LazySingleton* b = LazySingleton::get_instance();

    std::cout << a << std::endl;
    std::cout << b << std::endl; 

    return 0;
}
```

输出：

```
0x8b8eb0
0x8b8eb0
```

可以看出指针 `a` 和 `b` 的值相同，单例模式没有问题。

下面，我们将重点关注 LazySingleton 类在多线程环境下的对象创建操作，讨论此操作的线程安全性，且假定其他操作均是线程安全的。

## 2. 支持多线程的懒汉式单例

显然，上面的单例设计只适用于单线程程序，其是线程不安全的，有可能会实际创建多个对象。

想要是上面的程序线程安全，最简单直观的方法就是加锁，互斥锁。

我们看代码：

```cpp
#include <iostream>
#include <mutex>

class LazySingleton {
    static LazySingleton* instance;
    static std::mutex m;
    LazySingleton() {}
    LazySingleton(LazySingleton const &) = delete;
    LazySingleton& operator=(LazySingleton const &) = delete;

public:
    static LazySingleton* get_instance() {
        std::lock_guard lk(m);
        if (instance == nullptr) {
            instance = new LazySingleton();
        }
        return instance;
    }
};

LazySingleton* LazySingleton::instance;
std::mutex LazySingleton::m;

int main(void) {

    LazySingleton* a = LazySingleton::get_instance();
    LazySingleton* b = LazySingleton::get_instance();

    std::cout << a << std::endl;
    std::cout << b << std::endl; 

    return 0;
}
```

这段代码使用了 `std::mutex` 互斥，使得单例类线程安全了，现在可以保证是真正的“单例”了。

但问题也很明显，一旦有一个线程获取了锁，直到其释放锁前，其他线程都会阻塞。也就是说，即便单例对象早已经创建完成，其他的线程进行到这的时候还是经常会被阻塞，并发性能极差。如果并发线程多，可能导致大量线程在这里阻塞，甚至拖慢整个系统的效率。

## 3. 双重检验锁定模式（double-checked locking pattern）改进

双重检验锁定模式简单地说，与上面的方法的区别是：  
上面的方法中，是先获取锁，再检查空指针，如果指针为空，则创建对象实例；  
而双重检验锁定模式重，先检查一次空指针，如果指针为空，则获取锁，再检查指针，如果仍为空，则创建对象实例。

我们修改 `get_instance()` 方法如下：

```cpp
static LazySingleton* get_instance() {
    if (instance == nullptr) {
        std::lock_guard lk(m);
        if (instance == nullptr) {
            instance = new LazySingleton();
        }
    }
    return instance;
}
```

这个改动的好处是，只有当指针为空的时候，才会获取锁。而在之前的方法中，不管指针是否为空都要先获取锁。

而在获取锁后，还会再判断一次（即双重检验）指针是否为空，这是防止在第一个判断和获取锁之间，有其他进程改动过指针。

此方法减少了获取锁的可能，就减少了线程并发的代价，同样实现了线程安全，且有一定的性能提升。不过我认为这个优化带来的性能提升还是比较有限的。

\-

**但是，请注意，双重检验锁定模式并非适合所有场景。**

**双重检验锁定模式多用于延迟初始化**，我们上面的懒汉式单例也属于这种，只是过于简单，其创建实例的过程非常快，难以体现出此方法的弊端。

假设我们需求如下：通过 `get_instance()` 方法获取单例实例，如果实例不存在（指针为空），则创建实例，对实例对象进行一些初始配置（修改），然后返回指向实例的指针。

需求看似很简单，但如果我们再加一条要求呢：**初始化配置过程应该只进行一次，且耗时非常长！** 这并不是什么苛刻离谱的要求，而是一个合理合情的需求。

想象一下，如果我们将耗时很长的初始化过程也放在获取锁内的时间内执行，固然可以完成上述需求，但此时除了正在执行初始化配置以外的其他线程，调用 `get_instance()` 时都会阻塞，最终结果就是大量线程阻塞，导致系统整体性能下降。**所以我们需要将耗时的初始化过程放在锁外进行。**

但锁外执行初始化操作，也存在问题，我们看下面的代码：

```cpp
static LazySingleton* get_instance() {
    if (instance == nullptr) {
        std::lock_guard lk(m);
        if (instance == nullptr) {
            instance = new LazySingleton();
        }
    }
    // initialize something
    // takes a long time
    return instance;
}
```

假设我们现在不存在实例，有两个线程 A 和 B 开始调用 `get_instance()`。

1. 线程 A 执行到第 2 行的 `if (instance == nullptr)`，然后线程 B 开始执行；
2. 线程 B 一口气执行完了 `get_instance()` 全部的内容，即最终创建了一个单例对象，并做了一些初始配置（由第 7、8 行注释表示），并给调用者返回了指向实例的指针。
3. 线程 A 继续执行，执行到第二重检查时（第 4 行），发现指针已非空，于是跳到了第 7 行，**开始初始化配置**。

问题就在线程 A 最后这个初始化配置，线程 B 将实例指针返回给调用者后，该实例是有可能被修改的。而线程 A 在我们计划外的，对实例进行了第 2 次初始化，使得结果偏离预期。

上述场景，双重检验锁定模式就不合适了，我们看下一个。

## 4. 利用 std::call_once() 函数实施线程安全的延迟初始化

在讲这一节前，我们需要先介绍一下，`std::once_flag` 类 和 `std::call_once()` 函数，他们均在 C++11 中引入，在 "mutex" 头文件中。

`std::once_flag` 类是函数 `std::call_once()` 的辅助类，传递给多个 `std::call_once()` 调用的 `std::once_flag` 对象让这些调用**相互协调**，使得最终仅有一个调用真正运行完成。这个类不可复制亦不可移动。

`std::call_once()` 方法接受一个 `std::once_flag` 引用，以及一个可调用类型 `f` 及其参数。如果有多个 `std::call_once()` 调用接受了同一个 `std::once_flag` 引用，那么所有的 `std::call_once()`  调用只会有一个执行完成。另外要注意，如果多个传入的函数调用不一样，也是只会调用一次，但实际调用哪个是不确定的，属于未定义行为。

函数 `std::call_once()` 的声明如下：

```cpp
template< class Callable, class... Args >
void call_once( std::once_flag& flag, Callable&& f, Args&&... args );
```

> Callable 指的是任何可调用类型（callable type，包含函数指针、函数对象、lambda 等，能让适用者对其进行函数调用操作）。

总之，到这里，我们知道了一件事：**C++11 标准库中提供了方法，可以让某件事只做一次（比如初始化一次），不管调用了几次，不管是不是多线程调用的，最终一定只执行一次。并且可以保证这一次调用执行完成后，所有线程才会继续推进。**

### 4.1. 针对第 2 小节的问题优化

在 2. 小节中，我们说过那个方法会可能导致有很多线程阻塞。这个问题的主要原因是，我们先获取锁，再检查指针，导致不必要的获取锁和阻塞。

我们这里通过一个 `create_instance_flag`，让多个线程都调用 `std::call_once()`，保证对象的创建由其中某线程安全唯一的完成（通过合适的同步机制，有可能是系统提供的方法，也可能是加锁等）。这样做的性能比其他同类操作要好。

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class LazySingleton {
    static LazySingleton* instance;
    static std::once_flag create_instance_flag;
    LazySingleton() {}
    LazySingleton(LazySingleton const &) = delete;
    LazySingleton& operator=(LazySingleton const &) = delete;
    static void create_instance() {
        instance = new LazySingleton();
    }

public:
    static LazySingleton* get_instance() {
        std::call_once(create_instance_flag, create_instance);
        return instance;
    }
};

LazySingleton* LazySingleton::instance;
std::once_flag LazySingleton::create_instance_flag;

int main(void) {

    return 0;
}
```

### 4.2. 针对第 3 小节的问题优化

对于第 3 小节后面提到的，将耗时的初始化内容与对象的创建操作分离，但同时保证只执行一次，我们可以再加入一个 `init_instance_flag` 用于保证这一点。

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class LazySingleton {
    static LazySingleton* instance;
    static std::once_flag create_instance_flag;
    static std::once_flag init_instance_flag;
    LazySingleton() {}
    LazySingleton(LazySingleton const &) = delete;
    LazySingleton& operator=(LazySingleton const &) = delete;
    static void create_instance() {
        instance = new LazySingleton();
    }
    static void init_instance() {
        // initialize something
        // takes a long time
    }

public:
    static LazySingleton* get_instance() {
        std::call_once(create_instance_flag, create_instance);
        std::call_once(init_instance_flag, init_instance);
        return instance;
    }
};

LazySingleton* LazySingleton::instance;
std::once_flag LazySingleton::create_instance_flag;
std::once_flag LazySingleton::init_instance_flag;

int main(void) {

    return 0;
}
```