---
title: "C++ 读写锁的用法"
date: 2022-09-17 00:45:00
updated: 2022-09-17 02:13:00
categories: [并发编程]
tags: [Cpp,并发编程]
---

读锁和写锁其实就是通常意义上的共享锁和排他锁。

在 C++14 以前，标准库中是没有共享锁提供的，我们需要使用其他库（如 boost），或自行实现（如利用两个 `std::mutex` 实现）。C++14 以后标准库中才有了对共享锁 `std::shared_timed_mutex` 的支持。不过我们更常用的应当是 C++17 中提供的共享锁 `std::shared_mutex`。至于两者区别，不是本文重点，读者可以自行了解。本文的建议是除非特别需要，请使用 C++17 中的 `std::shared_mutex`。

本文默认读者有一点 C++ 并发编程基础。

我们首先使用 C++17 提供的互斥 `std::shared_mutex`，从名字上也能看出来了，这是个共享互斥。  
在使用上，读锁（共享锁）使用 `std::shared_lock()` 操作互斥；建议写锁（排他锁）使用 `std::lock_guard()` 或 `std::unique_lock()` 等操作互斥。因为这些包装类提供了 RAII 机制。


我们下面看代码理解：

```cpp
class TestSharedMutex {
    mutable std::shared_mutex _m;
    std::vector<int> _data;
public:
    TestSharedMutex() 
        // init
    {}
    void read_data() const {
        std::shared_lock lk(_m);
        // read data
    }
    void write_data(int value) {
        std::lock_guard lk(_m);
        // write data
    }
};
```

上面的类中有两个成员函数，`read_data()` 会读取类成员数据，`write_data()` 会修改数据。

我们只要在 `read_data()` 中使用 `std::shared_lock` 共享地获取并锁住互斥，然后读，  
而在 `write_data()` 中使用 `std::lock_guard` 排他地获取并锁住互斥，然后写就好了。

当读线程持有的共享锁全部释放之前，写线程获取排他锁时将被阻塞；  
反过来一样，在写线程释放排他锁之前，读线程获取共享锁时将被阻塞。