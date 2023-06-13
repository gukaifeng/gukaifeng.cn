

## 1. 什么是 C++ 最麻烦的解释？



C++ 最麻烦的解释（C++'s most vexing parse）是半开玩笑的说法，由 Scott Meyers 在《Effective STL》中提出，含义是：

> 针对存在二义性的 C++ 语句，只要它有可能被解释成函数声明，编译器就肯定将其解释成函数声明。

C++ 正式标准中明确了这一法则。详见标准文件 ISO/IEC 14882:2011 的 8.2 节。



## 2. 二义性的例子

我们先看一个代码：

```cpp
class background_task
{
public:
    void operator()() const
    {
        do_something();
        do_something_else();
    }
}
background_task f;
std::thread my_thread(f);
```

上面的代码中，我们设计了一个重载了 `()` 的类，并实例化了对象 f，然后将其传给 std::thread 对象的构造函数中。

> 与 C++ 标准库中许多类型相同，任何可调用类型（callable type，包含函数指针、函数对象、lambda 等，能让适用者对其进行函数调用操作）都适用于 std::thread。所以我们可以设计一个带有函数调用操作符的类，并将该类实例传递给 std::thread 的构造函数。

我们上面的 f 是一个具名变量，假设我们现在要给 std::thread 的构造函数传递一个临时变量：

```cpp
std::thread my_thread(background_task());
```

我们看这行代码，会不会有什么问题？

我们的本意是，创建一个 std::thread 类型的对象 my_thread，将 background_task() 返回的临时变量传递给 my_thread 的构造函数。

但实际呢，这行代码还可以有另一种解释：我们声明了一个函数，函数名为 my_thread，返回值为 std::thread 类型，函数接受一个函数指针的参数，该指针指向一个不接受参数，返回值类型为 background_task 的函数。

这里就出现了二义性，而我们前面说过：**针对存在二义性的 C++ 语句，只要它有可能被解释成函数声明，编译器就肯定将其解释成函数声明。**

所以我们的这个语句错了，因为它没能表达成我们期望中的含义，这个语句将被解释为函数声明。



## 3. 如何解决

我目前知道有三种方法：

1. 多用一对圆括号。

2. 采用新式的统一初始化语法（uniform initialization syntax，又名列表初始化）。

3. 使用 lambda。

### 3.1. 多用一对圆括号

多用一对圆括号将 `background_task()` 括起来，可以解决问题。因为我们知道函数声明是不可以直接这样写两层括号的，这样就没有歧义了。

```cpp
std::thread my_thread((background_task()));
```

### 3.2. 列表初始化

我们知道 C++ 中对象的初始化，除了传统的参数传递外，也可以使用列表初始化。

即用 大括号 `{}` 将参数括起来，而不再使用小括号 `()`。

```cpp
std::thread my_thread{background_task()};
```

### 3.3. 使用 lanbda

我们上面的例子是设计了一个带有函数调用操作符的类，并将该类实例传递给 std::thread 的构造函数。假设我们并不需求这样做，也可以将其执行内容直接写成 lambda，从而避免二义性问题、

```cpp
std::thread my_thread([]{
    do_something();
    do_something_else();
});
```

注意下这个方法虽然解决了二义性问题，但我们的需求也发生改变了（即不再使用带有函数调用操作符的类的临时变量）。所以只有前两种方法是真正解决了二义性问题，这个 lambda 方法实际上是改变了我们的编码策略。
