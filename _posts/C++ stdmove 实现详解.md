
本文不介绍 `std::move()` 的功能与用法，有了解需要可参考 [std::move - cppreference.com](https://en.cppreference.com/w/cpp/utility/move)。

本文主要研究 `std::move()` 的实现原理，从其定义出发，自顶向下剖析。





## 1. `std::move()` 的定义

移动函数 `std::move()` 定义在头文件 `<utility>` 中。

下面的定义取自 GCC 源码：

```cpp
/**
 *  @brief  Convert a value to an rvalue.
 *  @param  __t  A thing of arbitrary type.
 *  @return The parameter cast to an rvalue-reference to allow moving it.
*/
template<typename _Tp>
  constexpr typename std::remove_reference<_Tp>::type&&
  move(_Tp&& __t) noexcept
  { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

从大体上，我们可以看到 `std::move()` 是一个模板函数，其接受一个模板类型为 `_Tp` 的参数 `__t`，返回右值引用 `std::remove_reference<_Tp>::type&&>(__t)`。其注释说明了以下信息：

* 此函数用于将一个值转换为一个右值（注意这里没有强调被转换的值是“左值”还是“右值”，仅说了“值”）。
* `__t` 是任意类型的参数。
* 函数将参数 `__t` 转换为一个右值引用并返回，以使其可以移动。



模板函数的语法和相关关键字 `template` 和 `typename` 这里就不解释了。

另外 `noexcept` 表示该函数不会抛出异常，也不是本文重点。



下面我们将上面的函数定义分解讲解。





## 2. `constexpr` 常量表达式



`constexpr` 是 C++ 11 标准中加入一个关键字，其和 `const` 类似，都是表示这是一个常量。

二者略有区别：`constexpr` 是常量表达式（后面的 "expr" 是表达式的缩写）。此关键字表示，其修饰的是一个可以在**编译期**就确定具体值的表达式。

编译器也会对此关键字修饰的函数进行优化，实际的执行会将 `constexpr` 修饰的函数或静态数据成员优化为内联的。

综上，在 `std::move()` 的函数定义中，`constexpr` 代表着 `std::move()` 在编译时就可以计算出结果，且会被优化为内联的（inline）。



## 3. `static_cast` 类型转换



在 `std::move()` 定义中的 `return` 语句中，使用了 `static_cast`，这是静态类型转换，即在编译期完成的类型转换。

我们这里只需要了解，该 `return` 语句中，将参数 `__t` 转换为 `std::remove_reference<_Tp>::type&&` 类型并返回。





## 4. `std::remove_reference`

`std::remove_reference` 也是个函数模板，从名字上就比较好理解，即为移除引用。所以我们只想知道，它是怎样移除引用的。

我们先看 `std::remove_reference` 的定义：



```cpp
/// remove_reference
template<typename _Tp>
  struct remove_reference
  { typedef _Tp   type; };

template<typename _Tp>
  struct remove_reference<_Tp&>
  { typedef _Tp   type; };

template<typename _Tp>
  struct remove_reference<_Tp&&>
  { typedef _Tp   type; };
```



我们回看 `std::move()` 定义里 `return` 语句中的 `std::remove_reference<_Tp>::type&&` 部分，结合上述 `remove_reference` 类型的定义。我们可以看到，对于模板变量类型 `_Tp`，不论我们传入的是此变量本身的类型，还是引用类型 `_Tp&`，还是右值引用 `_Tp&&`，最终得到的 `std::remove_reference<_Tp>::type` 都是最原始的 `_Tp` 类型，这里的设计十分巧妙。



所以 `std::remove_reference<_Tp>::type&&` 中的 `_Tp` 无论是原始类型、引用类型还是右值引用类型，最终得到的都是一个右值引用（注意最后的 `&&`）。







## 5. 综合流程分析



为了方便，避免来回翻页，这里再贴一下 `std::move()` 的定义：

```cpp
template<typename _Tp>
  constexpr typename std::remove_reference<_Tp>::type&&
  move(_Tp&& __t) noexcept
  { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```



本文没有介绍 `std::move()` 的用法，但是为了能继续讲解，我们有必要知道，其参数定义虽然为右值引用（类型为 `__Tp&&`，形参为`__t`），但实际可接收的参数类型有三种（即原始类型，引用类型，右值引用类型都可以），而返回值的类型则始终是右值引用类型。我们下面逐个解释。



为了说明，这里引入 C++ 中的引用折叠规则：

1. 当我们将一个左值传递给函数的右值引用参数，且此右值引用参数指向模板类型参数时，编译器推断模板类型参数为实参的左值引用类型。
2. `X& &`、`X& &&` 和 `X&& &` 都会折叠为 `X&`。
3. `X&& &&` 会折叠为 `X&&`。



下面，假定类型 `T` 为原始类型，`T&` 为其引用类型，`T&&` 为其右值引用类型；参数为 `t`。



### 5.1. 情况一：传入的类型为原始类型（左值） `T`

此时，调用 `std::move(t)`，参数 `t` 的类型为 `T`。

根据引用折叠规则第 1 条，在 `std::move(t)` 内部，模板类型 `_Tp` 将被推断为实参 `t` 的左值引用类型 `T&`，形参 `__t` 的类型就是 `T& &&`，根据折叠引用规则第 2 条，即为 `T&`。

然后由 `std::remove_reference<>` 摘掉 `T&` 的引用得到 `T`，通过 `static_cast` 转换为 `T&&` 类型并返回。





> **从一个左值 `static_cast` 到一个右值引用是允许的**
> 通常情况下，`static_cast` 只能用于其他合法的类型转换。但是，这里又有一条针对右值引用的特许规则：虽然不能隐式地将一个左值转换为右值引用，但我们可以用 `static_cast` 显式地将一个左值转换为一个右值引用。

### 5.2. 情况二：传入的类型为左值引用类型 `T&`

此时，调用 `std::move(t)`，参数 `t` 的类型为 `T&`。

在 `std::move(t)` 内部，形参 `__t` 的类型是 `T&`，即 `_Tp&&` 等价 `T&`。

根据引用折叠规则第 2 条，`_Tp` 等价 `T&`（因为 `T& &&` 才可折叠为 `T&`）。

然后由 `std::remove_reference<>` 摘掉 `T&` 的引用得到 `T`，通过 `static_cast` 转换为 `T&&` 类型并返回。







### 5.3. 情况三：传入的类型为右值引用类型 `T&&`



此时，调用 `std::move(t)`，参数 `t` 的类型为 `T&&`。

在 `std::move(t)` 内部，形参 `__t` 的类型是 `T&&`，即 `_Tp&&` 等价 `T&&`，推断出 `_Tp` 类型即为 `T`。

然后由 `std::remove_reference<>` 直接会得到 `T`，由于 `__t` 已经是 `T&&` 类型，所以 `static_cast` 什么也没做，直接返回。