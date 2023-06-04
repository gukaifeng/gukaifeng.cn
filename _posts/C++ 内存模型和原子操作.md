---
title: "C++ 内存模型和原子操作"
date: 2023-02-02 22:30:00
updated: 2023-02-08 21:50:00
categories: [并发编程]
tags: [Cpp,并发编程]
---





> C++ 新标准引入了不少新特性，其中一项非常重要，它既不是新的语法功能，也不是新的程序库工具，而是新的线程感知的内存模型，却被大多数程序员所忽略。内存模型精确定义了基础构建单元应当如何运转。唯有以内存模型为基石，C++ 提供的并发工具方能可靠的工作。多数程序员之所以会忽略内存模型，是因为我们只需要借助互斥保护数据，并采用条件变量、future、线程闩或线程卡来触发事件信号，就足以将多线程运用自如，结果甚少有人深究底层细节。只有当我们尽力“贴近计算机底层硬件”，内存模型的精确细节的重要作用才会彰显。
>
> 不论其他语言如何定位，C++ 都是操作系统级别的编程语言。C++ 标准委员会的一个目标是领 C++ 尽量贴近计算机底层，而不必改用其他更低级的语言，C++ 十分灵活，可满足程序员的许多需求，包括容许他们在必要时“贴近计算机底层硬件”，语言本身不应构成障碍。原子类型(atomic)及其操作应运而生，提供了底层同步操作的功能，其常常只需一两条 CPU 指令即可实现。



## 1. 内存模型基础

内存模型牵涉两个方面：基本结构和并发。基本结构关系到整个程序在内存中的布局。它对并发很重要，尤其是在我们分析底层原子操作的时候。所以这里从基本结构开始介绍。就 C++ 而言，归根结底，基本结构就是对象和内存区域。





### 1.1. 对象和内存区域

C++ 标准中定义的“对象”与我们平常理解的不同，其**将“对象”定义为“某一存储范围(a region of storage)”**。

由此定义出发，C++ 程序的数据全部都有对象构成。这些对象中，有的数据内建基本类型（如 int 和 float），用于存储简单的数值，其他则是用户自定义类型的实例。某些对象具有子对象，如数列、派生类的实例、含有非静态数据成员的类的实例等，而其他则没有。

不论对象属于什么类型，它都会存储在一个或多个内存区域中。每个内存区域或是对象/子对象，属于标量类型(scalar type)，如 unsigned int 和 my_class*，或是一串连续的位域（注意位域有一项重要的性质：尽管相邻的位域分属不同的对象，但照样算作同一内存区域）。



下图是一个示例：



![图 1：将结构体分解为对象和内存区域](https://gukaifeng.cn/posts/c-nei-cun-mo-xing-he-yuan-zi-cao-zuo/c-nei-cun-mo-xing-he-yuan-zi-cao-zuo_1.png)



图的左边，是一个结构体的代码，右边是其对应的对象和内存区域。下面我们解释此图：



* 首先，整个结构体就是一个对象，此结构体对象由几个子对象构成。
* 每个数据成员即为一个子对象。
* 位域 bf1 和 bf2 共用一块内存区域，
* std::string 对象 s 则由几块内存区域构成。
* 其他数据成员都有各自的内存区域。



注意这里 bf3 是 0 宽度位域（其变量名将被注释掉，因为 0 宽度位域必须匿名），与 bf4 彻底分离，将 bf4 排除在 bf3 的内存区域之外，但 bf3 实际上并不占有任何内存区域。

我们可以从中总结出 4 个要点：

1. 每个变量都是对象，对象的数据成员也是对象；
2. 每个对象都占用至少一块内存区域；
3. 若变量类型属于内建基本类型（如 int 或 char），则不论其大小，都占用一块内存区域（且仅此一块），即便它们的位置相邻或它们是数列中的元素；
4. 相邻的位域同属于一块内存区域。



现在，我们介绍好了 C++ 中的内存模型，下面我们可以开始了解，内存模型和并发的关系。





### 1.2. 对象、内存区域和并发

首先我们要知道 C++ 程序的一个重要性质：**所有与多线程相关的事项都会牵涉内存区域。**

如果两个线程各自访问分离的内存区域，则相安无事，一切运作良好；反之，如果两个线程访问同一内存区域，我们就要警惕了。假使没有线程更新内存区域，则不必在意，只读数据无需保护或同步。

防止多线程资源竞争，最常见的办法就是使用互斥。不过在 C++ 中，提供了另一种方案：利用原子操作的同步性质，在目标内存区域（或相关内存区域）采取原子操作，从而强制两个线程遵从一定的访问次序。

我们后面会说明如何运用原子操作强制预定访问次序。假如多个线程访问相同的内存区域，则它们两两之间必须全部都有明确的访问次序。

如果我们把可能发生数据竞争的内存区域，全部通过原子变量来访问，可以避免未定义行为。但此方法不能预防数据竞争本身，因为我们依旧无法指定某一原子操作，令其首先踏足目标内存区域，但此法确实可以让程序正确运行，重回正规，符合已定义行为的规范。



在说原子操作前，我们还有一个牵涉对象和内存区域的重要概念需要理解：改动序列(modification order)。



### 1.3. 改动序列

在一个 C++ 程序中，**每个对象都具有一个改动序列，这个改动序列由所有线程在该对象上的全部写操作构成，其中第一个写操作即为对象的初始化。**在程序的任意一次执行过程中，所含的全部线程都必须形成相同的改动序列。变量的值会随着时间推移形成一个序列，如果在不同线程上观察到同一变量的改动序列不一样，就说明出现了数据竞争和未定义行为。如果多线程访问一个非原子变量，就需要我们自己进行同步操作，确保对于该变量，所有线程达成一致的改动序列。如果我们采用了原子操作，那么编译器就有责任确保必要的同步操作有效、到位。

为了实现上述保障，要求禁止某些预测执行(speculative execution)。原因是在改动序列中，只要线程看到过某个对象，则该线程的后续读操作必须获得相对新进的值，并且，该线程就同一对象的后续写操作，必须出现在改动序列后方。另外，如果某线程先向一个对象写数据，过后再读取它，那么必须读取前面写的值。若在改动序列中，上述读写操作之间还有别的写操作，则必须读取最后写的值。在程序内部，对于同一个对象，全部线程都必须就其形成相同的改动序列，并且在所有对象上都要求如此，而多个对象上的改动序列只是相对关系，线程之间不必达成一致。

> 预测执行(speculative execution)，又称推测执行、投机执行。这是一类底层优化技术，包括分支预测、数值预测、预读内存和预读文件等，目的是在多级流水 CPU 上提高指令的并发度。做法是提前执行指令而不考虑是否必要，若完成以后发现没必要，则抛弃或修正预执行的结果。



话说回来，原子操作由什么构成？如何利用它们强制操作服从预定次序呢？



## 2. C++ 中的原子操作及其类别

原子操作就是不可分割的操作，在系统的任一线程内，我们都不会观察到这种操作处于半完成的状态：要么已完成，要么没开始。

考虑读取某对象的过程，假如其内存加载行为属于原子操作，并且该对象的全部修改行为也都是原子操作，那么通过内存加载行为就可以得到该对象的初始值或得到某次修改而完整存入的值。



在 C++ 环境中，多数情况下，我们需要通过原子类型实现原子操作。





### 2.1. 标准原子类型

标准原子类型的定义位于头文件 `<atomic>` 内。这些类型的操作全是原子化的，并且，根据语言定义，C++ 内建的原子操作也仅仅支持这些类型，尽管可以通过采取互斥，我们能够令其他操作实现原子化。事实上，我们可以凭借互斥保护，模拟出标准的原子类型：它们全部（几乎）都具备成员函数 `is_lock_free()`，准许使用者判定某一给定类型上的操作是能由原子指令直接实现（`x.is_lock_free()` 返回 `true`，还是要借助编译器和程序库的内部锁来实现（`x.is_lock_free()` 返回 `false`）。



原子操作的关键用途是取代需要互斥的同步方式，得以性能提升。不过如果原子操作本身在内部使用了互斥，可能就无法有预期的性能提升，这种情况直接用互斥是更好的，并且不容易出错。



上述性质非常重要，C++ 程序库专门为此提供了一组宏。这些宏的作用是，针对由不同整数类型特化而成的各种原子类型，在**编译器期**判定其是否属于无锁数据结构。从 C++ 17 开始，全部原子类型都含有一个静态常量表达式成员变量(static constexpr member variable)，形如 `X::is_always_lock_free`，功能与那些宏相同：考察编译生成的一个特定版本的程序，当且仅当在所有支持该程序运行的硬件上，原子类型 X 全都以无锁结构形式实现，该成员变量的值才为 `true`。



这些宏定义在头文件 `<stdatomic.h>` 中，如下：

```cpp
// since C11
#define ATOMIC_BOOL_LOCK_FREE     /* implementation-defined */
#define ATOMIC_CHAR_LOCK_FREE     /* implementation-defined */
#define ATOMIC_CHAR16_T_LOCK_FREE /* implementation-defined */
#define ATOMIC_CHAR32_T_LOCK_FREE /* implementation-defined */
#define ATOMIC_WCHAR_T_LOCK_FREE  /* implementation-defined */
#define ATOMIC_SHORT_LOCK_FREE    /* implementation-defined */
#define ATOMIC_INT_LOCK_FREE      /* implementation-defined */
#define ATOMIC_LONG_LOCK_FREE     /* implementation-defined */
#define ATOMIC_LLONG_LOCK_FREE    /* implementation-defined */
#define ATOMIC_POINTER_LOCK_FREE  /* implementation-defined */

// since C23
#define ATOMIC_CHAR8_T_LOCK_FREE  /* implementation-defined */
```

| 值   | 解释                                       |
| ---- | ------------------------------------------ |
| `0`  | 该原子类型始终不是无锁结构（lock-free）    |
| `1`  | 该原子类型在运行时才能确定是否属于无锁结构 |
| `2`  | 该原子类型始终是无锁结构                   |



原子类型中只有一个类型不提供 `is_lock_free()` 成员函数，就是 `std::atomic_flag`。这是个简单的布尔标志，因此**必须是无锁操作**。其他原子类型都是由类模板 `std::atomic<>` 特化得出的，功能更齐全，但**可能不属于无锁结构**。我们希望这些类型在大多数主流平台上都具备无锁结构，但只是我们希望的，C++ 标准没有此要求。



\-

**标准的**原子类型有很多，还有别名，这里篇幅有限，参照 [std::atomic - cppreference.com - Type aliases](https://en.cppreference.com/w/cpp/atomic/atomic#Type_aliases)。

由于不具备拷贝构造函数和拷贝赋值操作符，因此按照传统做法，标准的原子类型无法复制，也无法赋值。然而，它们其实可以接受内建类型赋值，也支持隐式地转换成内建类型，还可以直接经由成员函数处理，如 `load()`、`store()`、`exchange()`、`compare_exchange_weak()`、`compare_exchange_strong()` 等。它们还支持复合赋值操作，如 `+=`、`-=`、`*=` 和 `|=` 等。而且，整形和指针的 `std::atomic<>` 特化都支持 `++` 和 `--` 运算。这些操作符有对应的具名成员函数，`fetch_add()` 和 `fetch_or()` 等。**注意，赋值操作符的返回值是存入后的值，而具名成员函数的返回值则是进行操作前的值。**习惯上，C++ 赋值操作符通常返回引用，指向接受赋值的对象，但原子类型的设计与此有别，要防止暗藏错误。否则，为了从引用获得存入的值，代码须执行单独的读取操作，是赋值和读取操作之间存在间隙，让其他线程有机可乘，得以改动该值，结果形成条件竞争。

\-



除了程序库已经提供给我们的繁多的原子类型外，我们也可以通过类模板 `std::atomic<>` 自己定义原子类型。该泛化模板锁具备的操作仅限于以下几种：`load()`、`store()` （接受用户自定义类型的赋值，以及转换为用户自定义类型）、`exchange()`、`compare_exchange_weak()`、`compare_exchange_strong()`。



接下来我们介绍几个通用操作，这些操作在每个标准原子类型上都能执行。



#### 2.2.1. 操作 `std::atomic_flag`

`std::atomic_flag` 是最简单的标准原子类型，表示一个布尔标志。该类型的对象只有两种状态：成立或置零。二者必居其一。

`std::atomic_flag` 经过刻意设计，相当简单，唯一用途是充当构建单元，因此我们认为普通开发者一般不会直接使用它。尽管这样，我们从 `std::atomic_flag` 切入，仍能借以说明原子类型的一些通用原则，方便进一步讨论其他原子类型。
`std::atomic_flag`  类型的对象必须由宏 `ATOMIC_FLAG_INIT` 初始化，它把标志初始化为置零状态：`std::atomic_flag f = ATOMIC_FLAG_INIT`。无论在哪里声明，也无论处于什么作用域，`std::atomic_flag` 对象永远以置零状态开始，别无他选。全部原子类型中只有 `std::atomic_flag` 必须采取这种特殊的初始化处理，它也是唯一保证无锁的原子类型。如果 `std::atomic_flag` 对象具有静态存储期(static storage duration)，它就会保证以静态方式初始化，从而避免初始化次序的问题(initialization-order issue)。对象在完成初始化才会操作其标志。

> 静态存储期：指某些对象随整个程序开始运行而分配到存储空间，等到程序结束运行才回收存储空间，包括静态局部变量、静态数据成员、全局变量等。



完成 `std::atomic _flag` 对象的初始化后，我们只能执行3 种操作：

1. 销毁，对应于析构函数。
2. 置零，对应于成员函数 `clear()`。
3. 读取原有的值并设置标志成立，对应于成员函数 `test_and_set()`。



我们可以为 `clear()` 和 `test_and set()` 指定内存次序。`clear()` 是存储操作，因此无法采用 `std:memory_order_acquire` 或 `std:memory_order_acq_rel` 内存次序，而 `test_and_set0` 是“读-改-写”操作，因此能采用任何内存次序。对于上面两个原子操作，默认内存次序都是`std::memory_order_seq_cst`（这里简单理解，内存次序在第 3 小节说）。




我们无法从 `std:atomic_flag` 对象拷贝构造出另一个对象，也无法向另一个对象拷贝赋值，这两个限制并非 `std:atomic_flag` 独有，所有原子类型都同样受限。原因是，按定义，原子类型上的操作全都是原子化的，但拷贝赋值和拷贝构造都涉及两个对象，而牵涉两个不同对象的单一操作却无法原子化。在拷贝构造或拷贝赋值的过程中，必须先从来源对象读取值，再将其写出到目标对象。这是在两个独立对象上的两个独立操作，其组合不可能是原子化的。所以，原子对象禁止拷贝赋值和拷贝构造。



由于 `std::atomic_flag` 严格受限，甚至不支持单纯的无修改查值操作，无法用作普通的布尔标志，因此最好还是使用 `std::atomic<bool>`。





#### 2.2.1 操作其他标准原子类型

1. `std::atomic<bool>` 是基于整数的最基本的原子类型。它是一个功能更齐全的布尔标志。

2. `std::atomic<T*>`：算术形式的指针运算。
3. 上面两个以外，最普遍的就是标准的整数原子类型。



这些标准原子类型的操作非常类似，我们简单介绍一下 `std::atomic<bool>`，然后给出详细的参考目录，自行查阅效率最高。



\-

尽管 `std:atomic<bool>` 也无法拷贝构造或拷贝赋值，但我们还是能依据非原子布尔量创建其对象，初始值是 true 或 false 皆可。该类型的实例还能接受非原子布尔量的赋值。

```cpp
std::atomic<bool> b(true);
b = false;
```



> 按 C++ 惯例，赋值操作符通常返回一个引用，指向接受赋值的目标对象(等号左侧的对象)。而非原子布尔量也可以向 `std::atomic<bool>` 赋值，但该赋值操作符的行为有别于惯常做法：它直接返回赋予的布尔值。这是原子类型的又一个常见模式：它们所支持的赋值操作符不返回引用，而是按值返回（该值属于对应的非原子类型）。假设返回的是指向原子变量的引用，若有代码依赖赋值操作的结果，那它必须随之显式地加载该结果的值，而另一线程有可能在返回和加载间改动其值。我们按值返回赋值操作的结果（该值属于非原子类型），就会避开多余的加载动作，从而确保获取的值正是赋予的值。

`std::atomic_flag` 的成员函数 `clear()` 严格受限，而 `std::atomic<bool>` 的写操作有所不同，通过调用 `store()` （true 和 false 皆可），它也能设定内存次序语义。类似地，`std::atomic <bool>` 提供了更通用的成员函数 `exchange()` 以代替 `test_and_set()`,它返回原有的值，还让我们自行选定新值作为替换。`std::atomic<bool>` 还支持单纯的读取(没有伴随的修改行为):隐式做法是将实例换为普通布尔值，显式做法则是调用 `load()`。不难看出，`store()` 是存储操作，而 `load()` 是载人操作，但 `exchange()` 是“读-改-写"操作。



还有一个操作是“比较-交换”，也是“读-改-写”操作，该操作时原子类型的编程基石。其主要实现是成员函数 `compare_exchange_weak()` 和 `compare_exchange_strong()`。这两个函数比较重要，详细请参阅 [std::atomic<T>::compare_exchange_weak, std::atomic<T>::compare_exchange_strong - cppreference.com](https://zh.cppreference.com/w/cpp/atomic/atomic/compare_exchange)。



这里以 `compare_exchange_weak()` 的函数重载**之一**为例做一个简单介绍：



```cpp
bool compare_exchange_weak( T& expected, T desired,
                            std::memory_order success,
                            std::memory_order failure) noexcept;
```

原子地比较 `*this` 和 `expected` 的[对象表示](https://zh.cppreference.com/w/cpp/language/object) (C++20 前)[值表示](https://zh.cppreference.com/w/cpp/language/object) (C++20 起)，而若它们逐位相等，则以 `desired` 替换前者（进行读-改-写操作），并返回 true。否则，将 `*this` 中的实际值加载进 `expected` （进行加载操作），并返回 false。后两个参数表示成功或失败时的内存次序，默认均为 `std::memory_order_seq_cst`（这里简单理解，内存次序在第 3 小节说）。

注意比较和复制是逐位的（类似 `std::memcmp` 和 `std::memcpy`），不使用构造函数、赋值运算符或比较运算符。



关于此函数的 `weak` 和 `strong` 两个版本的区别是：

* `compare_exchange_weak()` 可能会发生佯败(spurious failture)，所以往往必须配合循环使用。其败因不是因为变量本身有问题，而是函数执行的实机不对。具体来说，对于 `compare_exchange_weak0`，即使原子变量的值等于期望值，保存动作还是有可能失败，在这种情形下，原子变量维持原值不变，`compare_exchange_weak()` 返回 false。原子化的“比较-交换”必须由一条指令单独完成，而某些处理器没有这种指令，无从保证该操作按原子化方式完成。要实现“比较-交换”，负责的线程则须改为连续运行一系列指令，但在这些计算机上，只要出现线程数量多于处理器数量的情形，线程就有可能执行到中途因系统调度而切出，导致操作失败。

  ```cpp
  bool expected = false;
  extern atomic<bool> b; //由其他源文件的代码设定变量的值
  while(!b.compare_exchange_weak(expected, true) & !expected);
  ```

  此例中，如果 `b.compare_exchange_weak(expected, true)` 的返回值是 false，且 `expected` 变量也仍然是 false，就说明 `compare_exchange_weak()` 的调用发生佯败，我们就继续循环。（因为若正确运行，`b.compare_exchange_weak(expected, true)` 返回 false 的原因只能是 `b` 原来就是 true，与 `expected` 不等，但此情况下 `expected` 会被改为 true。若 `expected` 没有被改为 true，说明 `b` 在原本是 false 的情况下，`compare_exchange_weak()` 依然执行失败了，即佯败。）

* 只有当原子变量的值不符合预期时，`compare_exchange_strong()` 才返回 false。这让我们得以明确知悉变量是否成功修改，或者是否存在另一线程抢先切人而导致佯败，从而能够摆脱上例所示的循环。

那么如何选择`compare_exchange_weak()` 和 `compare_exchange_strong()` 呢？

假如经过简单计算就能得出要保存的值，而在某些硬件平台上，虽然使用 `compare_exchange_weak()` 可能导致佯败，但改用 `compare_exchange_strong()` 却会形成双重嵌套循环（因 `compare_exchange_strong()` 自身内部含有一个循环），那么采用 `compare.exchange weak()` 比较有利于性能。反之，如果存人的值需要耗时的计算，选择 `compare_exchange_strong()` 则更加合理。因为只要预期值没有变化，就可避免重复计算。就 `std::atomic<bool>` 而言，这并不是很重要，毕竟只有两种可能的值。但是对于体积较大的原子类型，这两种处理的区别很大。

\-



标准指针原子类型 `std::atomic<T*>`、标准整数原子类型和 `std::atomic<bool>` 区别不大，主要是多了一些操作和成员函`std::atomic<bool>` 其实是整数原子类型中的一个最基本的类型。

具体有哪些成员函数与操作，参见：

* 成员函数：https://en.cppreference.com/w/cpp/atomic/atomic#Member_functions
* 常量：https://en.cppreference.com/w/cpp/atomic/atomic#Constants
* 特殊成员函数和操作符重载：https://en.cppreference.com/w/cpp/atomic/atomic#Specialized_member_functions

关于指针有一点需要注意，其加减运算和内建指针类型一样，加减都是直接加指针的长度，即 +1 表示指向后一个元素的指针。





### 2.3. 泛化的 `std::atomic<>` 类模板



除了标准原子类型，程序员还可以通过类模板 `std::atomic<>` 定义自己的原子类型。

以 `std::atomic<bool>` 举例比较，假设我们有自定义类型 `UDT`，其原子化类型就是 `std::atomic<UDT>`，其提供的接口和 `std::atomic<bool>` 相同，不过要注意这些接口的参数和返回值都要改为 `UDT` 类型。

但是要注意，我们的 `UDT` 不能是任意的自定义类型，要想使用 `std::atomic<>` 模板，`UDT` **必须**满足以下特性：

* 具备平实拷贝赋值操作符（trivial copy-assignment operator)。若此类型具有基类或非静态数据成员，则它们同样必须具备平实拷贝操作符。
* 不得含有任何虚函数；
* 不得从虚基类派生得出；
* 必须由编译器代其隐式生成拷贝赋值操作符。



> 平实拷贝操作符：平直、简单的原始内存复制及其等效操作。



由于以上限制，赋值操作不涉及任何用户编写的代码，因此编译器可借用 `memcpy()` 或采取与之等效的行为完成它。

最后，值得注意的是，“比较-交换”操作所采用的是逐位比较(bitwise comparison)运算，效果等同于直接使用 `memcmp()` 函数。即使 `UDT` 自行定义了比较运算符，在这项操作中也会被忽略。若自定义类型含有填充位(padding bit)，却不参与普通比较操作，那么即使 `UDT` 对象的值相等，比较-交换操作还是会失败。

>填充位：
>
>编译器根据类定义的 alignas 说明符或编译命令，可能会在对象内各数据成员后方特意留出间隙，令它们按 2/4/8 字节或其他 2 次幂倍乘数字对齐内存地址，从而加速内存读写操作。这些间隙即为填充位，它们不具名，对使用者不可见。



### 2.4. 原子操作的非成员函数

我们前面提到过的都是原子类型的成员函数，标准库中还提供了很多非成员函数。



非成员函数往往有与成员函数相同的功能，并与之意义对应（如果有的话）。



举个例子，我们有一个数组类对象 `arr`，我们要对他排序的话，假如有以下两种方式：

1. `arr.sort();`：`sort()` 是 `arr` 的成员函数。
2. `sort(&arr);`：`sort()` 是非成员函数。



标准库提供的原子操作的相关非成员函数有挺多，详见 https://en.cppreference.com/w/cpp/header/atomic#Functions 。













## 3. 同步操作和强制次序



假设有两个线程共同操作一个数据结构，其中一个负责添加数据，一个负责读取数据。为了避免恶性条件竞争，写线程设置一个标志，用来表示数据已存储好，而读线程一直待命，等到标志成立才着手读取。下面的代码演示了这一点（例子中每个线程执行一个函数，并且仅执行一次）。



```cpp
#include <vector>
#include <atomic>
#include <iostream>
#include <chrono>
#include <thread>

std::vector<int> data;
std::atomic_bool data_ready(false);

void reader_thread()
{
    while (!data_ready.load()) // ①
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
    std::cout << "The answer=" << data[0] << "\n"; // ②
}
void writer_thread()
{
    data.push_back(42); // ③
    data_ready = true;  // ④
}
```

我们认真分析一下这段代码：首先，① 和 ④ 操作的是一个名为 `data_ready` 的原子变量，所以这两处的执行不会引起竞争；但是，② 和 ③ 操作的变量 `data` 既不是原子变量，也没有加锁！但我可以提前告知，该代码可正确运行，且不会引起关于 `data` 的竞争。下面解释。



原子变量 `data_ready` 的操作之间的竞争由编译器解决，满足内存模型关系“**先行(happens-before)**”和“**同步(synchronizes-with)**”，这些操作确定了必要的次序。关于“先行”和“同步”，我会在后面更详细的解释。



首先，在两个线程内，同线程内的 ①② 顺序和 ③④ 顺序一定是固定的，即 ① 在 ② 前发生，③ 在 ④前发生。由于有原子变量 `data_ready` 存在，④ 一定在 ① 成立前发生。先行关系可以传递，所以上述四句代码的最终顺序一定是 ③④①②。



这里 ④ 一定在 ① 之前发生，是因为 `data_ready` 是原子类型，原子操作自然成立（因为会按照默认次序）。但我们在代码中应该明确指定这种顺序，因为原子操作还能根据需要自己选择其他次序。在讲解这一点之前，我们先说一下什么是同步关系和先行关系。



* 同步关系：在 ④ 写入数据之后，才在 ② 中读取数据，就说这两个线程的读写操作存在同步关系。
* 先行关系：分为先行关系和严格先行关系，是程序中确定操作次序的基本要素，用途是清楚界定哪些操作能看见其他哪些操作产生的结果。严格先行关系顾名思义，即一处语句一定在另一处语句之前执行，例如上面 ③ 必定在 ④ 之前执行，就是严格先行关系。



在 C++ 中，像上面 ③ 和 ④ 的执行严格依据流程控制的，因为 ③ 在 ④ 的前面。但是，如果是在同一个语句中出现的两个操作，C++ 标准是没有规定次序的。我们看下面的代码：

```cpp
#include <iostream>

void foo(int a,int b)
{
    std::cout << a << "," << b << std::endl;
}

int get_num()
{
    static int i = 0;
    return ++i;
}

int main()
{
    foo(get_num(), get_num());
}
```

这段代码可能输出 “1,2”，也可能输出 “2,1”（跟编译器有关），因为 `foo()` 中的两个 `get_num()` 没有规定次序（注意这里有别于内建逗号表达式，内建逗号表达式会固定从左到右执行，且整体表达式的值为逗号表达式内最右的值）。



在线程先行关系和先行关系中，各种操作都会被标记为 `memory_order_consume`（这里先简单看看，下面细说），而严格先行关系不会。由于绝大多数代码都不会用 `memory_order_consume` 标记，因此这个区别对我们没啥影响，我们后面就全部都叫“先行关系”。





### 3.1. 原子操作的内存次序



原子操作内存次序是一个枚举类型，定义在头文件 `<atomic>` 中，如下：

```cpp
typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;
```

共有六种，其中 `memory_order_seq_cst` 是最严格的内存次序，各种原子类型的所有操作都默认遵从这个次序，除非我们另外指定。

虽然有六种次序，但只代表 3 种模式：

1. 先后一致次序（`memory_order_seq_cst`）
2. 获取-释放次序（`memory_order_consume`、`memory_order_acquire`、`memory_order_release` 和 `memory_order_acq_rel`）
3. 宽松次序（`memory_order_relaxed`）

>在不同的CPU架构上，这几种内存模型也许会有不同的运行开销。以某处理器执行相同的改动操作举例：其采用不同内存次序重复执行，比如原本按照先后一致次序，后来改成获取-释放次序，又如首先采取获取-释放次序，然后换作对宽松次序，若需精确控制该执行结果，在前后两种内存次序下均为别的处理器所见（或同样不可见），则系统有可能要插入额外的同步指令。如果这些系统具备的处理器数目众多，额外的同步指令也许会消耗大量时间，降低系统的整体性能。
>另一方面，采用 x86 或 x86-64 架构的 CPU （如在台式计算机中常见的 Intel 和 AMD 处理器）并不需要任何额外的同步指令，就能确保服从获取-释放次序的操作的原子化，甚至不采取特殊的载入操作就能保障先后一致次序，而且其存储行为的开销仅略微增加。
>C++提供了上述各种内存次序模型，资深程序员可以自由选用，籍此充分利用更为细分的次序关系，从而提升程序性能；还一些场景对性能不构成关键影响，普通开发者则能采取默认方式，按先后一致性次序执行原子操作（比起其他内存序，它分析起来要容易很多）。



代码中的内存次序关系因采用不同内存模型而异，只有认识不同的内存模型如何影响程序的行为，才可以理解其中机制，从而选出最合适的内存次序。接下来，我们逐分析每种内存次序和同步关系，及其产生的效果。





#### 3.1.1. 先后一致次序

先后一致次序最简单，最好理解，是默认次序。简单的说就是把一切时间视为按先后顺序发生，其操作与这种次序保持一致。在这种次序下，如果把多线程程序中的原子类型操作该到单线程中，那么执行结果没有任何区别。这个次序非常易于分析和推理，例如我们可以把多线程中全部的符合该顺序组合列出，从而验证代码是否符合预期。这种内存模型无法重新编排操作次序。如果一个线程内，某项操作优于另一项操作发生，那么其他线程所见的先后次序都必须如此。

若某项操作标记为 `memory_order_seq_cst` 则编译器和 CPU 须严格遵循源码逻辑流程的先后顺序。在相同的线程上，以该项操作为界，其后方的任何操作不得重新编排到它前面，而前方的任何操作不得重新编排到它后面，其中“任何”是指带有任何内存标记的任何变量之上的任何操作。



下面看一段代码，为了方便演示，我们都显式指定了 `memory_order_seq_cst` 内存次序，即便其是默认的。

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<bool> x, y;
std::atomic<int> z;

void write_x()
{
    x.store(true, std::memory_order_seq_cst); // ①
}

void write_y()
{
    y.store(true, std::memory_order_seq_cst); // ②
}

void read_x_then_y()
{
    while (!x.load(std::memory_order_seq_cst))
        ;
    if (y.load(std::memory_order_seq_cst)) // ③
        ++z;
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_seq_cst))
        ;
    if (x.load(std::memory_order_seq_cst)) // ④
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x);
    std::thread b(write_y);
    std::thread c(read_x_then_y);
    std::thread d(read_y_then_x);
    a.join();
    b.join();
    c.join();
    d.join();
    assert(z.load() != 0); // ⑤
}
```

这里 ⑤ 处的断言永远不会触发，因为 `z` 的值只可能是 1 或 2。按常规思路走代码即可得到，非常简单，这里不多解释了。

先后一致次序是最直观、最符合直觉的内存次序，但由于它要求在所有线程间进行全局同步，因此也是代价最高的内存次序。在多处理器系统中，处理器之间也许为此而需要频繁通信。
为了避免产生这种同步开销，我们需要突破先后一致次序的思维模式，考虑使用别的内存次序。



> **非先后一致次序**
>
> 
>
> 如果完全脱离了保持先后一致次序的环境，事情就开始复杂了。我们要面对的最大问题很可能是事件不再服从单一的全局次序。换言之，不同线程所看到的同一组操作的次序和效果可能呈现差异。在前文构思的模型中，不同线程上的操作独立而完整地交替执行，我们必须舍弃这种思维，而以真正的并发思维分析同时发生的事件，并且多个线程也不必就事件发生次序达成一致。如果想脱离默认的 `memory_order_seq_cst` 次序，采用其他内存次序编写代码（或仅仅是为了读懂代码），那么读者大有必要深究非先后一致次序。即使多个线程上运行的代码相同，由于某些线程上的操作没有显式的次序约束，因此它们有可能无法就多个事件的发生次序达成一致，而在不同的 CPU 缓存和内部缓冲中同一份内存数据也可能具有不同的值。以上认知非常重要，我们必须再次强调：**线程之间不必就事件发生次序达成一致。**
> 我们不仅须舍弃交替执行完整操作的思维模式，还得修正原来的认知，不再任由编译器或处理器自行重新排列指令。如果没有指定程序服从哪种内存次序，则采用默认内存次序。它仅仅要求一点：**全部线程在每个独立变量上都达成一致的修改序列。**不同变量上的操作构成其特有的序列，假设各种操作都受施加的内存次序约束，若线程都能看到变量的值相应地保持一致，就容许这个操作序列在各线程中出现差别。
> 完全脱离先后一致次序的最佳示范就是，将上例的全部操作改用 `memory_order_relaxed` 次序。读者一旦掌握其要领，就可以回头学习获取-释放次序。它针对某些操作建立次序关系，这更接近我们对指令重新编排的旧有认知。







#### 3.1.2. 宽松次序

如果采用宽松次序，那么原子类型上的操作不存在同步关系。

**在单一线程内，同一个变量上的操作仍然服从先行关系，但几乎不要求线程间存在任何次序关系。**

该内存次序的唯一要求是，在一个线程内，对相同变量的访问次序不得重新编排。对于给定的线程，一旦它见到某原子变量在某时刻持有的值，则该线程的后续读操作不可能读取相对更早的值。

`memory_order_relaxed` 次序无须任何额外的同步操作，线程间仅存的共有信息是每个变量的改动序列。
我们仅凭两个线程就足以说明，采用宽松次序的操作究竟能宽松到何种程度，如下代码：

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<bool> x, y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed);  // ①
    y.store(true, std::memory_order_relaxed);  // ②
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_relaxed))  // ③
        ;
    if (x.load(std::memory_order_relaxed))  // ④
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);  // ⑤
}
```

这次，断言可能会触发断言错误 ⑤，因为即使变量的载入操作读取了 true 值 ③，且变量 `x` 的存储操作 ① 在 `y` 的存储操作之前发生 ②，变量 `x` 的载入操作也可能读取 false 值 ④。变量 `x` 和 `y` 分别执行操作，让各自的值发生变化，但它们是两个不同的原子变量，因此宽松次序不保证其变化可为对方所见。
不同变量上的宽松原子操作可自由地重新排列，前提是这些操作受到限定而须服从先行关系（譬如在同一个线程内执行的操作），不会产生同步关系。

在变量 `x` 和 `y` 的两项存储操作之间，以及它们的两项载入操作之间，确实有着先行关系。但是，任一存储操作与任一载入操作之间却不存在这种关系，所以两项载入操作都可能见到两项存储操作以乱序执行。



宽松次序会使程序对人而言变得复杂，尤其当线程数和变量多了的时候，复杂程度非常恐怖。有一种方法可以实现更佳的同步效果：运行获取-释放次序。该方法同样避免了“绝对先后一致次序”的额外开销。

#### 3.1.3. 获取-释放顺序

获取-释放次序比宽松次序严格一些，它会产生一定程度的同步效果，而不会形成服从先后一致次序的全局总操作序列。

在该内存模型中，原子化载入即为获取操作（`memory_order_acquire`），原子化存储即为释放操作（`memory_order_release`），而原子化“读-改-写”操作（像 `fetch_add()` 和 `exchange()`）则为获取或释放操作，或二者皆是（`memory_order_acq_rel`）。

这种内存次序在成对的读写线程之间起到同步作用。释放与获取操作构成同步关系，前者写出的值由后者读取。换言之，若多个线程服从获取-释放次序，则其所见的操作序列可能各异，但其差异的程度和方式都受到一定条件的制约。

下面的代码是 3.1.1 中代码改用获取-释放次序语义的重写版本：

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<bool> x, y;
std::atomic<int> z;

void write_x()
{
    x.store(true, std::memory_order_release);
}

void write_y()
{
    y.store(true, std::memory_order_release);
}

void read_x_then_y()
{
    while (!x.load(std::memory_order_acquire))
        ;
    if (y.load(std::memory_order_acquire))  // ①
        ++z;
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_acquire))
        ;
    if (x.load(std::memory_order_acquire))  // ②
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x);
    std::thread b(write_y);
    std::thread c(read_x_then_y);
    std::thread d(read_y_then_x);
    a.join();
    b.join();
    c.join();
    d.join();
    assert(z.load() != 0);  // ③
}
```

本例中，变量 `x` 和 `y` 的载入操作 ②① 有可能都读取 false 值（与宽松次序的情况一样），因此有可能令断言触发错误 ③。变量 `x` 和 `y` 分别由不同线写出，所以两个释放操作都不会影响到对方线程。

两个读线程所见的操作序列可能并不相同，因为本例的先行关系尚不充足，未能强制各操作服从一定的次序。

为了分析获取-释放次序的优势，我们需要考虑同一线程上的两个存储操作，我们修改 3.1.2 中的代码。若我们将变量 `y` 的存储操作改用 `memory_order_release` 次序将 `y` 的载入操作改用 `memory_order_acquire` 次序，就会强制变量 `x` 上的宽松操作服从一定的次序。如下代码所示：



```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<bool> x, y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed);  // ①
    y.store(true, std::memory_order_release);  // ②
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_acquire))  // ③，以自旋方式等待变量 y 的值设置为 true
        ;
    if (x.load(std::memory_order_relaxed))  // ④
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);  // ⑤
}
```

变量 `y` 的存储操作 ② 最终会为其载入操作 ③ 所见，后者会读取前者写出的 true 值，因为存储操作采用 `memory_order_release` 次序，而载入操作采用 `memory_order_acquire`次序，所以两者同步。变量 `x` 的存储操作 ① 和 `y` 的存储操作 ② 同属一个线程，所以操作 ① 会在操作 ② 之前发生。又因为变量 `y` 的存储操作 ② 与载入操作 ③ 同步，且变量 `x` 的存储操作 ① 在变量 `y` 的载入操作 ③ 之前发生，进而可知，变量 `x` 的存储操作 ① 也在 `x` 的载入操作 ④ 之前发生。因此，变量 `x` 的载入操作必然读取 true 值，而断言不会触发。

若变量 `y` 的载入操作并未置于 while 循环内，情况就可能不同。变量 `y` 的载入操作可能读取 false 值，那么从变量 `x` 读取的值就不确定。

获取和释放操作唯有成对才可以产生同步。释放操作所存储的值必须为获取操作所见，才会产生有效同步。若上例中的存储操作 ② 或载入操作 ③ 是宽松原子操作，那么他们对变量 `x` 的两次访问不存在强制次序，载入操作 ④ 并不一定会读取 true 值，因此断言有可能触发错误。



\- 

**通过获取-释放次序传递同步**

我们回顾线程间先行关系，它有一个重要性质：可传递。

若操作甲跨线程地先于操作乙发生，且操作乙跨线程地先于操作丙发生，则操作甲跨线程地先于操作丙发生。按此定义，获取-释放次序可用于多线程之间的数据同步，即使“过渡线程”的操作不涉及目标数据，也照样可行。

我们看下面的代码：

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<int> data[5];
std::atomic<bool> sync1(false), sync2(false);

void thread_1()
{
    data[0].store(42, std::memory_order_relaxed);
    data[1].store(97, std::memory_order_relaxed);
    data[2].store(17, std::memory_order_relaxed);
    data[3].store(-141, std::memory_order_relaxed);
    data[4].store(2003, std::memory_order_relaxed);
    sync1.store(true, std::memory_order_release);  // ①，设置 sync1 城里
}

void thread_2()
{
    while (!sync1.load(std::memory_order_acquire))  // ②，一直循环，直到 sync1 成立
        ;
    sync2.store(std::memory_order_release);  // ③，设置 sync2 成立
}

void thread_3()
{
    while (!sync2.load(std::memory_order_acquire))  // ④，一直循环，到 sync2 成立为止
        ;
    assert(data[0].load(std::memory_order_relaxed) == 42);
    assert(data[1].load(std::memory_order_relaxed) == 97);
    assert(data[2].load(std::memory_order_relaxed) == 17);
    assert(data[3].load(std::memory_order_relaxed) == -141);
    assert(data[4].load(std::memory_order_relaxed) == 2003);
}

int main()
{
    std::thread t1(thread_1);
    std::thread t2(thread_2);
    std::thread t3(thread_3);
    t1.join();
    t2.join();
    t3.join();
}
```

这段代码通过两个原子布尔类型变量 `sync1`  和 `sync2` 进行同步，达成了同步效果，且线程 t2 没有接触到任何数据（即 t2 是过渡线程）。

上例中，我们还能进一步将变量 `sync1` 和 `sync2` 融合成单一变量，在线 thread_2上对其执行“读-改-写”操作，该操作采用 `memory_order_acq_rel` 次序。我们可以选用 `compare_exchange_strong()` 来执行操作。通过这个函数我们可以保证，只有见到线程 thread_1 完成存储之后，才更新该单一变量。代码改动如下：

```cpp
std::atomic<int> sync(0);

void thread_1()
{
    // ...
    sync.store(1, std::memory_order_release);
}

void thread_2()
{
    int expected = 1;
    while (!sync.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) 
        expected = 1;
}

void thread_3()
{
    while (sync2.load(std::memory_order_acquire) < 2)
        ;
    // ...
}
```

>如果我们使用“读-改-写”操作，选择满足需要的内存次序语义则是关键。上面的场景中，我们同时需要获取语义和释放语义，所以选择次序 `memory_order_acq_rel` 正适（尽管我们也可以选用其他内存次序）。举个反例，采用 `memory_order_acquire` 次序的 `fetch_sub()` 不会与任何操作同步，因为它不是释放操作。类似地，存储操作无法与采用 `memory_order_release` 次序的 `fetch_or()` 同步因为 `fetch_or()` 所含的读取行为并不是获取操作。若“读-改-写”操作采用 `memory_order_acq_rel` 次序，则其行为是获取和释放的结合，因此前方的存储操作会与之同步，而它也会与后方的载入操作同步，正如本例所示。
>若将获取-释放操作与保序操作（即保持原来的操作次序，就是最严格的次序）交错混杂，那么保序载入的行为就与服从获取语义的载入相同，保序存储的行为则与服从释放语义的存储相同。如果“读-改-写”操作采用保序语义，则其行为是获取和释放的结合。混杂其间的宽松操作仍旧宽松，但由于获取-释放语义引人了同步关系（也附带引人了先行关系），这些操作的宽松程度因此受到限制。
>尽管锁操作的运行效果有可能远离预期，但如果我们使用过锁，就要面对一个次序问题：给互斥加锁是获取操作，解锁互斥则是释放操作。我们都清楚，互斥的有效使用方法是必须确保锁住了同一个瓦斥，才读写受保护的相关变量。同理，获取和释放操作唯有在相同的原子变量上执行，才可确保这些操作服从一定的次序。如果要凭借互斥保护数据，由于锁具有排他性质，因此其上的加锁和解锁行为服从先后一致次序，就如同我们明令强制它们采用保序语义一样（实际上我们并没有这样设定）。类似地，假设我们利用获取-释放次序实现简单的锁，那么考察一份使用该锁的代码，其行为表现将服从先后一致次序，而加锁和解锁之间的内部行为则不一定。
>如果原子操作对先后一致的要求不是很严格，那么由成对的获取-释放操作实现同步，开销会远低于由保序操作实现的全局一致顺序。这种做法很耗脑力，要求周密思量线程间那些违背一般情况的行为，从而保证不出差错，让程序服从施加的次序正确运行。





\-



前文说过内存次序 `memory_order_consume` 是获取-释放次序中的一种，但有两个明确的结论是：

1. **C++ 17 标准明确建议我们不采用此内存次序。**
2. **任何使用到 `memory_order_consume` 的地方，都应用 `memory_order_acquire` 替代。**



`memory_order_consume` 可以算作 `memory_order_acquire` 的一种特例，可能会造成数据依赖。我认为没有必要再去了解，感兴趣的可以自己查阅。

### 3.2. 释放序列和同步关系

我们看一段简单的代码：

```cpp
#include <atomic>
#include <thread>
#include <vector>
std::vector<int> queue_data;
std::atomic<int> count;

void wait_for_more_items() {}
void process(int data) {}

void populate_queue()
{
    unsigned const number_of_items = 20;
    queue_data.clear();
    for (unsigned i = 0; i < number_of_items; ++i)
    {
        queue_data.push_back(i);
    }

    count.store(number_of_items, std::memory_order_release);  // ①，最初的存储操作
}

void consume_queue_items()
{
    while (true)
    {
        int item_index;
        if ((item_index = count.fetch_sub(1, std::memory_order_acquire)) <= 0)  // ②，一项“读-改-写”操作
        {
            wait_for_more_items();  // ③，等待装入新数据
            
            continue;
        }
        process(queue_data[item_index - 1]);  // ④，安全
    }
}

int main()
{
    std::thread a(populate_queue);
    std::thread b(consume_queue_items);
    std::thread c(consume_queue_items);
    a.join();
    b.join();
    c.join();
}
```

这里是一个生产者-消费者模型。我们注意有 b 和 c 两个消费者线程。他们在 ③ 处的访问是会符合预期的，因为后执行到此处的线程读取到的是先到此处的线程改写后的值（即“释放”操作后的值），所以 ④ 处对数据的访问索引是不会重复的，也就没有数据竞争，是安全的。



下图中虚线表示释放序列，实线表示先行关系。

![图 2：释放序列与上述代码中共享容器的操作](https://gukaifeng.cn/posts/c-nei-cun-mo-xing-he-yuan-zi-cao-zuo/c-nei-cun-mo-xing-he-yuan-zi-cao-zuo_2.png)

### 3.3. 栅栏(fence)

原子类型上的操作具有各种内存次序语义，大多数同步关系据此形成。尽管如此我们还可以使用栅栏引入别的次序约束。



如果缺少栅栏(fence)功能，原子操作的程序库就不完整。

栅栏具备多种操作，用途是强制施加内存次序，却无须改动任何数据。通常，它们与服从 `memory_order_relaxed` 次序的原子操作组合使用，可以令宽松操作服从一定的次序。

栅栏操作全部通过全局函数执行。当线程运行至栅栏处时，它便对线程中其他原子操作的次序产生作用。

栅栏也常常被称作“内存卡”或“内存屏障”，其得名原因是它们在代码中划出界线，限定某些操作不得通行（如线程闩、线程卡相关）。

针对不同变量上的宽松操作，编译器或硬件往往可以自主对其进行重新编排。栅栏限制了这种重新编排。在一个多线程程序中，可能原来并非处处具备先行关系和同步关系，栅栏则在欠缺之处引入这两种关系。



栅栏定义在头文件 `<atomid>` 中，其参数仅有一个内存次序，如下：

```cpp
extern "C" void atomic_thread_fence( std::memory_order order ) noexcept;
```





3.1.2 中的代码涉及两个线程，分别含有两项原子操作。现在我们在它们中间都加入栅栏 ②⑤，代码如下：

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

std::atomic<bool> x, y;
std::atomic<int> z;

void write_x_then_y()
{
    x.store(true, std::memory_order_relaxed);  // ①
    std::atomic_thread_fence(std::memory_order_release);  // ②
    y.store(true, std::memory_order_relaxed);  // ③
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_relaxed))  // ④
        ;
    std::atomic_thread_fence(std::memory_order_acquire);  // ⑤
    if (x.load(std::memory_order_relaxed))  // ⑥
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);  // ⑦
}
```

② 和 ⑤ 会形成同步关系，使得 ⑥ 一定在 ① 之后执行，所以 ⑥ 处的判断必为 true，所以 ⑦ 断言不会出错。





### 3.4. 凭借原子操作令非原子操作服从内存次序

我们利用原子操作强制施行内存次序，其中真正的奥妙在于它们可以强制非原子操作服从一定的内存次序，并避免因数据竞争而引发的未定义行为。

我们再次修改 3.3 中的代码，将原子布尔类型变量 `x` 改成非原子化的普通 bool 类型。代码如下：

```cpp
#include <atomic>
#include <thread>
#include <assert.h>

bool x = false;  // ①，x 现已改为普通的非原子变量
std::atomic<bool> y;
std::atomic<int> z;

void write_x_then_y()
{
    x = true;  // ②，变量 x 的存储操作位于栅栏前面
    std::atomic_thread_fence(std::memory_order_release);
    y.store(true, std::memory_order_relaxed);  // ③，变量 y 的存储操作位于栅栏后面
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_relaxed))
        ;
    std::atomic_thread_fence(std::memory_order_acquire);
    if (x)  // ⑤，这里读取 ① 处所写出的值
        ++z;
}

int main()
{
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);  // ⑥，此断言不会触发
}
```

注意这里虽然改了 `x` 的类型为普通 bool 类型，但 ⑥ 处的断言同样不会触发！因为存在栅栏的原因，⑤ 必然在 ① 后执行。

### 3.5. 强制非原子操作服从内存次序



我们在 3.4 已经演示了一个简单地强制非原子操作服从内存次序的示例，即关于变量 `x` 的存取。



C++ 标准库的高层级工具，如互斥和条件变量，以原子变量、内存次序等作为基础逻辑。如互斥上的 `lock()` 和 `unlock()` 方法。



`lock0` 的实现方式是在循环中反复调用 `flag.text_and_set0`，其中所采用的次序为 `std:memory_order_acquire`；`unlock()` 实质上则是服从 `std::memory_order_release` 次序的 `flag.clear()` 操作。第一个线程调用 `lock()` 时，标志 `flag` 正处于置零状态，`test_and_set()` 的第一次调用会设置标志成立并返回 false，表示负责执行的线程已获取了锁，遂循环结束，互斥随即生效。该线程可修改受其保护的数据而不受干扰。此时标志已设置成立，如果任何其他线程再调用 `lock()`，都会在 `test_and_set()` 所在的循环中阻塞。

当持锁线程完成了受保护数据的改动，就调用 `unlock()`，再进一步按 `std::memory_order_release` 次序语义执行 `flag.clear()`。若第二个线程因调用 `lock()` 而反复执行 `flag.test_and_set()`，又因该操作采用了 `std::memory_order_acquire` 次序语义，故标志 `flag` 上的这两项操作形成同步。

根据互斥的使用规则，首先，受保护数据的改动须按流程顺序在调用 `unlock()` 前完成；其次，只有在解锁以后才能重新加锁；最后，第二个线程需要先获取锁，接着才可以访问目标数据。所以，改动先于 `unlock()` 发生，自然也先于第二个线程的 `lock()` 调用，进而更加先于第二个线程的数据访问。



尽管其他互斥实现的内部操作各有不同，但其基本原理都一样：`lock()` 与 `unlock()` 都是某内部内存区域之上的操作，前者是获取操作，后者则是释放操作。



我们了解了 C++ 中的同步机制，这些机制根据同步关系，按各种形式为相关内存次序提供了保证。正因如此，我们得以运用这些机制来同步数据，建立相关的内存次序。这些工具所给出的同步关系如下：

1. `std::thread`
2. `std::mutex`、`std::timed_mutex`、`std::recursive_mutex` 和 `std::recursive_timed_mutex`
3. `std.shared_mutex`  和 `std::shared_timed_mutex`
4. `std::promise`、`std::future` 和 `std::shared_future`
5. `std::packaged_task`、`std::future` 和 `std::shared_future`
6. `std::async`、`std::future` 和 `std::shared_future`
7. `std:experimental::future`、`std::experimental::shared_future` 和后续函数
8. `std::experimental::latch`
9. `std::experimental::barrier`
10. `std::experimental::flex barrier`
11. `std:condition_variable` 和 `std::condition_variable_any`
