---
title: "C++ 完美转发"
date: 2023-02-10 15:51:11
updated: 2023-02-10 15:51:11
categories:
- 编程语言基础
- Cpp
tags: [Cpp]
---



## 1. 什么是完美转发？



提到完美转发，就有必要先说一下，什么是转发，什么样的转发才称得上是完美转发。

在 C++ 中，转发指的就是函数之间的参数传递（例如函数 `f1` 接收了一个参数 `a`，而后又将此参数 `a` 传递给了其函数体内调用的另一个函数 `f2`）。

而完美转发指的就是在函数之间传递参数的过程中，参数在传递后的属性保持不变（如左值仍是左值，右值仍是右值，`const` 修饰也会保留）。





## 2. 常规转发存在的问题



对于普通的转发，参数在函数间传递时属性可能会发生改变，我们看一个例子。



```cpp
#include <iostream>
#include <utility>

void an_orther_fun(int a, int& b)
{
    std::cout << "in an_orther_fun(): a = " << a << ", b = " << ++b << std::endl;
}

void transmit(int a, int b)
{
    an_orther_fun(a, b);
}

int main()
{
    int a = 2, b = 3;

    std::cout << " before transmit(): a = " << a << ", b = " << b << std::endl;
    
    transmit(a, b);
    
    std::cout << "  after transmit(): a = " << a << ", b = " << b << std::endl;
    
    return 0;
}
```

其输出为：

```
 before transmit(): a = 2, b = 3
in an_orther_fun(): a = 2, b = 4
  after transmit(): a = 2, b = 3
```

注意，函数 `an_orther_fun()` 的第二个参数类型是引用 `int&`，并且我们在函数中给该引用的值加上了 1（`++b` 那里），也就是我们预期应当会修改其第二个参数的值加 1。但根据输出，虽然我们在 `an_orther_fun()` 打印出了加 1 后的 `b` 的值。但在外层，我们执行 `tansmit()` 后，`b` 的值并没有加 1。



这里的原因其实很显而易见的，就是我们 `a` 在从 `transmit()` 传递到 `an_other_fun()` 的时候，其属性已经改变了：`transmit()` 中的 `a` 是外层 `a` 的一个副本，而不是引用。



这就是常规引用可能带来的转发问题。

你可能会说，我们将 `transmit()` 的第二个参数类型也改为引用 `int&` 不就可以解决了吗？

```cpp
void transmit(int a, int& b) { ... }
```

在我们这个示例中，这样的修改确实是可以达成我们示例中的目的的。

但，这只是示例，实际还有更多的可能性。例如，如果上例中的 `b` 处是一个右值引用，怎么办？我们看下面的代码：

```cpp
#include <iostream>
#include <utility>

void an_orther_fun(int a, int&& b)
{
    std::cout << "in an_orther_fun(): a = " << a << ", b = " << b << std::endl;
}

void transmit(int a, int&& b)
{
    an_orther_fun(a, b);  // error: cannot bind rvalue reference of type ‘int&&’ to lvalue of type ‘int’
}

int main()
{
    int a = 2;

    std::cout << " before transmit(): a = " << a << ", b = " << b << std::endl;
    
    transmit(a, 3);
    
    std::cout << "  after transmit(): a = " << a << ", b = " << b << std::endl;
    
    return 0;
}
```

实际上还是会有报错，即便我们传递了一个右值。



要想在任何情况都能完美转发参数，保持其全部的性质，就要使用到完美转发。



## 3. 引用折叠规则



为了说明，这里引入 C++ 中的引用折叠规则：

1. 当我们将一个左值传递给函数的右值引用参数，且此右值引用参数指向模板类型参数时，编译器推断模板类型参数为实参的左值引用类型。
2. `X& &`、`X& &&` 和 `X&& &` 都会折叠为 `X&`。
3. `X&& &&` 会折叠为 `X&&`。



## 4. 使用完美转发

完美转发需要使用到标准库中的 `std::forward<>()` 函数，其定义在头文件 `<utility>`，**其必须通过显式模板实参来调用**。

其有两个重载，一个接收左值引用类型参数，另一个接收右值引用类型参数，定义如下：

```cpp
/**
 *  @brief  Forward an lvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type& __t) noexcept
  { return static_cast<_Tp&&>(__t); }

/**
 *  @brief  Forward an rvalue.
 *  @return The parameter cast to the specified type.
 *
 *  This function is used to implement "perfect forwarding".
 */
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
  {
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
      " substituting _Tp is an lvalue reference type");
    return static_cast<_Tp&&>(__t);
  }
```



可以看到 `std::forward<Type>()` 返回其实参类型 `_Tp`的右值引用 `_Tp&&`。即 `std::forward<T>()` 的返回值是类型 `T&&`。

通常情况下，我们使用 `std::forward<Type>()` 传递那些定义为模板类型参数的右值引用的函数参数。通过其返回类型上的引用折叠，`std::forward<Type>()` 可以保持给定实参的左值/右值属性。



\-



修改后的 `transmit()` 像下面这样：

```cpp
template <typename F, typename T1, typename T2>
void transmit(F f, T1&& t1, T2&& t2)
{
    f(std::forward<T1>(t1), std::forward<T2>(t2));
}
```

下面我们修改上面的例子，作为演示。



### 4.1. 示例：传递左值



我们使用模板修改上面的代码，先以在函数间传递左值为例：

这里 `an_orther_fun()` 函数接受的第二个参数是引用，并且会修改该值：

```cpp
void an_orther_fun(int a, int& b)
{
    std::cout << "in an_orther_fun(): a = " << a << ", b = " << ++b << std::endl;
}
```

整体代码如下：

```cpp
#include <iostream>
#include <utility>

void an_orther_fun(int a, int& b)
{
    std::cout << "in an_orther_fun(): a = " << a << ", b = " << ++b << std::endl;
}

template <typename F, typename T1, typename T2>
void transmit(F f, T1&& t1, T2&& t2)
{
    f(std::forward<T1>(t1), std::forward<T2>(t2));
}

int main()
{
    int a = 2, b = 3;

    std::cout << " before transmit(): a = " << a << ", b = " << b << std::endl;
    
    transmit(an_orther_fun, a, b);
    
    std::cout << "  after transmit(): a = " << a << ", b = " << b << std::endl;
    
    return 0;
}
```

输出：

```cpp
 before transmit(): a = 2, b = 3
in an_orther_fun(): a = 2, b = 4
  after transmit(): a = 2, b = 4
```

可以看到，外层的变量 `b` 也确实被修改了，达到了我们预期的效果，下面解释。

\-

我们主要关注 `std::forward<T2>(t2))` 相关部分。

我们看整个调用过程：

```cpp
...

template <typename F, typename T1, typename T2>
void transmit(F f, T1&& t1, T2&& t2)
{
    f(std::forward<T1>(t1), std::forward<T2>(t2));
}

...
 
transmit(an_orther_fun, a, b);

...
```



在 `transmit` 调用处，根据引用折叠规则第 1 条，编译器推断 `T2` 类型为 `int&`。随后将 `T2` 赋予 `std::forward` 的模板类型，得到 `std::forward<int&>(t2)`。



最后进入 `std::forward` 内部，显然这里适用 `std:forward` 的第一个重载：

```cpp
template<typename _Tp>
  constexpr _Tp&&
  forward(typename std::remove_reference<_Tp>::type& __t) noexcept
  { return static_cast<_Tp&&>(__t); }
```

在这里，`_Tp` 为 `int&`，返回值会被 `static_cast` 转换为 `Tp&&` 类型，即 `int& &&` 类型，根据引用折叠规则第 2 条，即 `int&` 类型。





最终，我们在外部传递给 `transmit()` 的是一个左值参数，而 `transmit()` 将该值传递给 `an_orther_fun()` 时，该参数仍为左值，目标达成！





### 4.2. 示例：传递右值

有了上面的详细解释，这里可以简单说了。

假设调用过程如下（忽略其他可能需要修改以保证程序正确运行的地方，只看关键）：

```cpp
...
void an_orther_fun(int a, int&& b)  // 注意这里第二个参数是右值引用类型 int&&
{
    ...
}

...

template <typename F, typename T1, typename T2>
void transmit(F f, T1&& t1, T2&& t2)
{
    f(std::forward<T1>(t1), std::forward<T2>(t2));
}

...
 
transmit(an_orther_fun, a, 3);

...
```

这里最后调用 `transmit()` 时给其传递的第二个参数是一个右值 3，此时编译器会将模板类型 `T2` 推断为 `int`，随后将 `T2` 赋予 `std::forward` 的模板类型，得到 `std::forward<int>(t2)`。

在这里，`_Tp` 为 `int`，返回值会被 `static_cast` 转换为 `Tp&&` 类型，即 `int&&` 类型，即右值引用类型。

最终，我们在外部传递给 `transmit()` 的是一个右值参数，而 `transmit()` 将该值传递给 `an_orther_fun()` 时，该参数仍为右值，目标达成！





