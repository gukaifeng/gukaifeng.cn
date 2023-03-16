---
title: "C++20 协程(Coroutines)详解"
date: 2023-03-16 08:55:00
updated: 2023-03-16 08:55:00
categories:
- 编程语言基础
- Cpp
tags: [Cpp,协程]
---



## 1. 什么是协程(Coroutines)



在 C++ 20 标准中，**协程(Coroutines)**是一个可以暂停执行并在稍后继续执行的**函数**。

协程是无栈的(Stackless)：协程通过返回给调用者暂停执行，并且恢复执行所需的数据是与栈分开存储的。这样就可以异步执行顺序的代码（例如，不显式使用回调来处理非阻塞 I/O），也可以支持延迟计算算法，或者其他用途等等。





如果一个函数的定义中包含下面至少任意一种，则这个函数是一个协程：

* `co_await` 表达式 —— 暂停执行。

  ```cpp
  task<> tcp_echo_server()
  {
      char data[1024];
      while (true)
      {
          std::size_t n = co_await socket.async_read_some(buffer(data));
          co_await async_write(socket, buffer(data, n));
      }
  }
  ```

* `co_yield` 表达式 —— 暂停执行并返回一个值。

  ```cpp
  generator<int> iota(int n = 0)
  {
      while (true)
          co_yield n++;
  }
  ```

* `co_return` 语句 —— 完成执行并返回一个值。

  ```cpp
  lazy<int> f()
  {
      co_return 7;
  }
  ```

  



## 2. 协程的定义





本文参考：

1. [P0912R5 - Merge Coroutines TS into C++20 working draft](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0912r5.html)
2. [Coroutines (C++20) - cppreference.com](https://en.cppreference.com/w/cpp/language/coroutines)
3. [Coroutine support (C++20) - cppreference.com](https://en.cppreference.com/w/cpp/coroutine)

