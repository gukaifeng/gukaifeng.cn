---
title: C++11 中 lambda 的使用方法.md
date: 2021-12-09 17:45:10
updated: 2021-12-09 17:45:10
categories:
- 编程语言基础
- Cpp
tags: [Cpp]
toc: true
---

lambda 是 C++11 中的特性，可以简化我们的代码，非常好用。

lambda 的基本功能最常用，且简单。

但其实 lambda 也有很多花里胡哨的功能，这篇文章主要讲基本功能，参考资料来自 [Lambda expressions (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture)，如果你想要学习那些本文中没有介绍的花里胡哨的用法，可以自己看看。

## 1. 一个简单的示例演示

这里先通过一个很简单的示例来看看 lambda 的使用场景。

我们知道 C++ 中的 sort() 函数可以对数组进行排序，如果你排序的是一个 int 类型的数组，那 sort() 的默认行为是**从小到大**排序，而 sort 的第三个参数可以自定义比较函数，我们这里使用 lambda 让其**从大到小**排序 int 数组。

```cpp
// exec sort() using our own compare function (lambda)
sort(vec.begin(), vec.end(), [](int a, int b){ return a > b; });
```

我们写一个简单地完成程序测试这一点，下面的代码在给 vector 插入 20 个 [0,100) 的随机数，输出一次，然后执行 sort()，再执行一次。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>

int main(int argc, char *argv[]) {
    srand((unsigned int)time(nullptr));
    const int vec_len = 20;
    const int range = 100;
    std::vector<int> vec;
    for (int i = 0; i < vec_len; i++) {
        vec.push_back(rand() % range);
    }

    // before sort()
    for (int num: vec) {
        std::cout << num << " ";
    }
    std::cout << '\n';

    // exec sort() using our own compare function (lambda)
    sort(vec.begin(), vec.end(), [](int a, int b){ return a > b; });

    // after sort()
    for (int num: vec) {
        std::cout << num << " ";
    }
    std::cout << '\n';

    return 0;
}
```

输出如下（由于是随机数，你的输出大概和我的不同，但相同点应该是，第一行输出无序，第二行输出是**从大到小**有序的）：

```
43 57 57 53 70 71 84 85 15 98 78 33 23 48 71 92 69 23 54 39 
98 92 85 84 78 71 71 70 69 57 57 54 53 48 43 39 33 23 23 15 
```

这个示例用来入门，下面我们来详细说 lambda 语法。

## 2. lambda 基本用法

一个全功能的 lambda 语句应该是这样的：

**（这是 C++11 版本，更高 C++ 版本有新特性，后面再说）**

```cpp
[ captures ] ( params ) specs { body }
```

* `[ captures ]`: 用来捕获自身作用域中的局部变量，多个捕获由 `,` 隔开。
  
  * 空值：不捕获，此时此 lambda 中无法使用其所在作用于的局部变量，但可以使用全局变量，和参数传入的变量。
  
  * `=`: 默认以拷贝的方式捕获（和参数传递类似，拷贝方式不会修改实参原值）。
  
  * `&`: 默认以引用的方式捕获。
  
  * 另外，也可以直接在捕获列表中写变量名，指定具体某个变量使用何种方式捕获。
  
  * **注意：捕获列表中的变量名不能和参数列表中的形参名有相同。**
  
  下面是几个关于 `[]` 的示例，节选自 [cppreference.com](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture)。
  
  ```cpp
  struct S2 { void f(int i); };
  void S2::f(int i)
  {
      [&]{};          // OK: by-reference capture default
      [&, i]{};       // OK: by-reference capture, except i is captured by copy
      [&, &i] {};     // Error: by-reference capture when by-reference is the default
      [&, this] {};   // OK, equivalent to [&]
      [&, this, i]{}; // OK, equivalent to [&, i]
  }
  ```
  
  ```cpp
  struct S2 { void f(int i); };
  void S2::f(int i)
  {
      [=]{};        // OK: by-copy capture default
      [=, &i]{};    // OK: by-copy capture, except i is captured by reference
      [=, *this]{}; // until C++17: Error: invalid syntax
                    // since C++17: OK: captures the enclosing S2 by copy
      [=, this] {}; // until C++20: Error: this when = is the default
                    // since C++20: OK, same as [=]
  }
  ```
  
  ```cpp
  struct S2 { void f(int i); };
  void S2::f(int i)
  {
      [i, i] {};        // Error: i repeated
      [this, *this] {}; // Error: "this" repeated (C++17)
  
      [i] (int i) {};   // Error: parameter and capture have the same name
  }
  ```

* `( params )`: 函数的形参，若没有可以省略。不过在 C++11 中，当使用 `specs` 了时，即便不需要形参也要写个 `()`。具体定义方法和常规函数定义的形参一样，就不解释了。

* `specs`: 指示符，可以为空。在 C++11 中，其可选值只有两个（更高版本有扩展，但非本文重点）：
  
  1.  `mutable`: 如果写上这个关键字，那么在 lambda 函数体中可以修改以拷贝形式捕获的对象，以及调用该对象的非 const 的方法。否则，以拷贝形式捕获的对象是不可以在 lambdy 函数体中修改的。
  
  2. `-> ret`: lambda 的返回值。当 lambda 有返回值且返回值可以自动推导出时，可以省略，返回值类型将自动推导。否则需要手动注明。

* `{ body }`: 函数体，同样和常规函数定义的一样。



\-

另外，如果在 lambda 定义的最后加一个 `()`，表示这个 lambda 会立即调用。

不过此法并不是很常用。举个例子：

```cpp
#include <iostream>

int main(int argc, char *argv[]) {
    int a = 10;
    int b = [=]{ return a + 1; }();
    std::cout << "a = " << a << ", b = " << b << std::endl;
    return 0;
}
```

输出

```
a = 10, b = 11
```

也就是说，lambda 函数立即执行了。



## 3. lambda 最常用方法

```cpp
[ capture ]( params ) -> return_type { body }
```

这个是最简单最常用的用法了，没有用那些高级特性。

注意下 `( params )` 和 `-> return_type` 在某些时候可以省略就好了。
