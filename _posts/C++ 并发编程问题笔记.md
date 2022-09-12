---
title: C++ 并发编程问题笔记
date: 2022-09-11 15:38:00
updated: 2022-09-12 22:12:00
categories: [并发编程]
tags: [Cpp,并发编程]
---



这篇文章是我阅读《C++ 并发编程实战（第 2 版）》的疑问笔记。

从开始到复杂，有些问题现在回看其实十分基础，甚至有些好笑，但我还是决定留下它们。

一些问题初次整理时可能理解上还是有问题的，在后期学习如有发现会更正。

## 第 1 章：你好，C++ 并发世界

## 第 2 章：线程管控

### Q1. join() 函数会阻塞当前线程直至关联的线程结束吗？

> A1：是的，只有 join 的线程结束后，当前线程才会继续执行。

我们这里引用 [cppreference.com](https://en.cppreference.com/w/cpp/thread/thread/join) 的关于 join() 描述：

> Blocks the current thread until the thread identified by *this finishes its execution.
> 
> 阻塞当前线程直至 *this 所标识的线程结束其执行。

我们看一段代码：

```cpp
#include <iostream>
#include <thread>
#include <unistd.h>

class SleepLoop {
public:
    SleepLoop(const uint times, const uint span)
        : _times(times), _span(span) {}
    void operator()() {
        for (uint i = 0; i < _times; i++) {
            std::cout << "A: sleep loop: time = " << i << ", span = " << _span << std::endl;
            sleep(_span);
        }
    }
private:
    const uint _times, _span;
};

int main(int argc, char *argv[]) {
    std::thread t((SleepLoop(3, 1)));
    t.join();
    std::cout << "B: this statement is after t.join()" << std::endl;
    return 0;
}
```

这段代码执行了一个线程，该线程会执行一个循环 3 次，每次间隔 1 秒，每次循环会输出一个 A 语句。然后在 `t.join()` 后，主线程输出 B 语句。

结合上面 [cppreference.com](https://en.cppreference.com/w/cpp/thread/thread/join) 的描述，执行 `t.join()` 语句的线程即为当前线程，`t` 所关联的线程即为 `*this` 所标识的线程。

我这里先给出一个执行结果：

```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z main]$ ./muti-threads 
A: sleep loop: time = 0, span = 1
A: sleep loop: time = 1, span = 1
A: sleep loop: time = 2, span = 1
B: this statement is after t.join()
```

可以看到 B 语句总是在 A 语句全部输出完以后才输出，这印证了我们之前的结论。

你可以会说我试验次数太少或不够严谨，你可以通过修改 SleepLoop 类临时变量构造函数的参数，来试验更多可能，我这里因为篇幅问题只是举了个例子。B 一定是始终在 A 输出完以后输出的。

### Q2. 新线程是从 std::thread 对象创建时就开始执行的吗？

> A2：多数情况下是的。但使用默认构造时不是，因为默认构造函数没有给其传递任何参数，无法构建有效的线程。

我们还是看上面的代码，加了 C 语句：

```cpp
#include <iostream>
#include <thread>
#include <unistd.h>

class SleepLoop {
public:
    SleepLoop(const uint times, const uint span)
        : _times(times), _span(span) {}
    void operator()() {
        for (uint i = 0; i < _times; i++) {
            std::cout << "A: sleep loop: time = " << i << ", span = " << _span << std::endl;
            sleep(_span);
        }
    }
private:
    const uint _times, _span;
};

int main(int argc, char *argv[]) {
    std::thread t((SleepLoop(3, 1)));
    std::cout << "C: this statement is used to test the start time of a new thread" << std::endl;
    t.join();
    std::cout << "B: this statement is after t.join()" << std::endl;
    return 0;
}
```

下面是执行了两次的输出：

```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z main]$ ./muti-threads 
A: sleep loop: time = C: this statement is used to test the start time of a new thread0, span = 1

A: sleep loop: time = 1, span = 1
A: sleep loop: time = 2, span = 1
B: this statement is after t.join()
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z main]$ ./muti-threads 
C: this statement is used to test the start time of a new threadA: sleep loop: time = 
0, span = 1
A: sleep loop: time = 1, span = 1
A: sleep loop: time = 2, span = 1
B: this statement is after t.join()
```

我们可以看到 A 和 C 语句的输出顺序是比较混乱的，但可以看到有 A 先于 C 输出的样例。我这里不打算深究其原因，因为此现象已经足以证明当前问题。



### Q3. 对于一个新建 std::thread 对象 t，在当前线程的某一时刻执行 t.join() 或 t.detach() 是必须的吗？

> A3：是必须的。更严格的说，应当是在对象 t 析构前执行 t.join() 或 t.detach()。



要理解这个问题，首先我们要清楚 t.join() 和 t.detach() 做了什么，明白为什么要调用它们。

* join() 在概念上与 fork() 相对，fork 是分叉的意思，而 join 是汇合的意思。我们知道调用 t.join() 会阻塞当前线程直到 t 关联的线程 **（记为 t_thread）** 执行完成，这**看起来**就像把 t_thread 线程的内容的代码放到了 t.join() 调用处一样。**线程的资源将在调用 t.join() 后且线程执行完成后被回收。**

* t.detach() 会将线程 t_thread 与当前线程分离，t_thread 将在后台执行。分离后当前线程将无法再与  t_thread 直接通信，也无法等待 t_thread 完成，无法汇合 t_thread（即调用 t.join()）。t_thread 确实还在后台执行，但**其归属权和控制权都将转移给 C++ 运行时库（runtime library，又名运行库），由此保证线程退出时与之关联的资源可以被正确回收。**



也就是说，**必须调用 join() 或 detach() 和线程的资源释放问题有关。**

那为什么应当在对象 t 析构前来执行这两个方法呢？

我们看 std::thread 类的析构函数：

```cpp
  ~thread()
  {
    if (joinable())
std::terminate();
  }
```

我们知道，对于一个正常的线程对象，如果没有调用过 join() 或 detach()，那么它的 joinable() 返回值为 true，也就是说，如果我们没有 join() 或 detach()，也没有转移 std::thread 对象的所有权，那么在 std::thread 对象析构时，**整个进程会被直接终止，甚至析构函数也不会执行，风险极大**，所以应当在对象 t 析构前来执行这两个方法。



### Q4. 为什么 t.join() 或 t.detach() 只能调用一次（两者共享一次），且是否可调用的判断条件均为 t.joinable() ?

当 t 关联的线程是一个活跃线程时，joinable() 返回 true，否则返回 false。 

什么叫活跃线程呢，其实就是正在执行的线程。还有就是，已经执行完但还未调用过 t.join() 或 t.detach() 时，也属于活跃线程。

现在，这个问题可以转换为，**为什么只有 t 关联的线程是活跃线程时才可以执行 t.join() 或 t.detach()，以及为什么一旦执行 t.join() 或 t.detach() 后，t 关联的线程就不再是活跃线程了呢？**

首先我们看 joinable() 方法的实现，看看其做了什么：

```cpp
class thread
{
// ...
private:
    id _M_id;
// ...
public:
//...
    bool
    joinable() const noexcept
    { return !(_M_id == id()); }
// ...
}
```

这里的实现是节选的（全部的话可太长了），我们目前只要知道：

* `_M_id` 是 id 类对象，表示 thread 类对象当前关联的线程 ID。

* `id()` 是无惨构造 id 类对象，此构造法得到的 id 对象是特殊的，**其值（为 0）不代表任何线程**。

也就是说，当 t 对象中存储的线程 ID 无法表示任何线程时，就认为是其是非 joinable() 的。

我们看一段代码：

```cpp
#include <ios>
#include <iostream>
#include <thread>
#include <unistd.h>
#include <utility>

class ThreadFunc {
public:
    ThreadFunc() {}
    void operator()() {
        
    }
};

int main(int argc, char *argv[]) {
    std::cout << "main: thread id is " << std::this_thread::get_id() << std::endl;

    std::thread t1((ThreadFunc()));
    std::thread t2((ThreadFunc()));
    
    std::cout << std::boolalpha;
    std::cout << "  t1: thread id is " << t1.get_id() << ", and joinable() is " << t1.joinable() << std::endl;
    std::cout << "  t2: thread id is " << t2.get_id() << ", and joinable() is " << t2.joinable() << std::endl;
    
    t1.join();
    t2.detach();
    std::cout << "  t1: after   join(), thread id is " << t1.get_id() << ", and joinable() is " << t1.joinable() << std::endl;
    std::cout << "  t2: after detach(), thread id is " << t2.get_id() << ", and joinable() is " << t2.joinable() << std::endl;
    
    return 0;
}
```

输出如下：

```
main: thread id is 140367827535680
  t1: thread id is 140367809496832, and joinable() is true
  t2: thread id is 140367801104128, and joinable() is true
  t1: after   join(), thread id is thread::id of a non-executing thread, and joinable() is false
  t2: after detach(), thread id is thread::id of a non-executing thread, and joinable() is false
```

通过观察输出，再进一步解释，我们可以明白一件事：

当调用过 t.join() 或 t.detach() 时，这两个方法会将 t 中表示线程 id 的对象值改为 0，即不再代表任何线程。也就是说，**一旦调用过 t.join() 或 t.detach()，t 中表示的线程 id 将失效，t 将无法再控制与其关联的线程！**

**t 无法再控制与其关联的线程，自然也就无法再次 join() 或 detach()，所以这俩方法只能执行一次。**

还有就是，不是说 joinable() 为 false 后线程就没了，只是 t 跟这线程没关系了，线程还是那个线程，只是不再满足 “只有 t 关联的线程是活跃线程时才可以执行 t.join() 或 t.detach()” 中的“关联”二字。



### Q5. 线程对象何时是 joinable 的？



**除了**下列情况，线程对象都是 joinable 的：

1. 其是默认构造的。

2. 如果其已经被用来使用移动构造或赋值构造创建了另一个 std::thread 对象。

3. 其成员函数 join() 或 detach 已经被调用过。

我们下面通过代码看上述三种场景：

```cpp
#include <ios>
#include <iostream>
#include <thread>
#include <unistd.h>
#include <utility>

class ThreadFunc {
public:
    ThreadFunc() {}
    void operator()() {}
};


int main(int argc, char *argv[]) {

    std::cout << std::boolalpha;
    
    std::cout << "\n-------------------------------------------------------------------\n\n";

    std::thread t1;

    std::cout << "t1 was constructed by default-constructor,  t1's joinable() is " << t1.joinable() << std::endl;

    std::cout << "\n-------------------------------------------------------------------\n\n";
    
    std::thread t2((ThreadFunc()));
    std::thread t3((ThreadFunc()));
    
    std::cout << "t2, before t2.join(), t2's joinable() is " << t2.joinable() << std::endl;
    std::cout << "t3, before t3.detach(), t3's joinable() is " << t3.joinable() << std::endl << std::endl;
    
    t2.join();
    t3.detach();
    
    std::cout << "t2, after t2.join(), t2's joinable() is " << t2.joinable() << std::endl;
    std::cout << "t3, after t3.detach(), t3's joinable() is " << t3.joinable() << std::endl;

    std::cout << "\n-------------------------------------------------------------------\n\n";

    std::thread t4((ThreadFunc()));
    std::thread t5((ThreadFunc()));

    std::cout << "t4, t4's joinable() is " << t4.joinable() << std::endl;
    std::cout << "t5, t5's joinable() is " << t5.joinable() << std::endl << std::endl;

    std::thread t6(std::move(t4));
    std::thread t7 = std::move(t5);

    std::cout << "t6 was constructed by \"std::move(t4)\", so t4's joinable() is " << t4.joinable() << ", t6's joinable() is " << t6.joinable() << std::endl;
    std::cout << "t7 was constructed by \"= std::move(t5)\", so t5's joinable() is " << t5.joinable() << ", t7's joinable() is " << t7.joinable() << std::endl;

    std::cout << "\n-------------------------------------------------------------------\n\n";

    return 0;
}
```

输出：

```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z main]$ ./muti-threads 

-------------------------------------------------------------------

t1 was constructed by default-constructor,  t1's joinable() is false

-------------------------------------------------------------------

t2, before t2.join(), t2's joinable() is true
t3, before t3.detach(), t3's joinable() is true

t2, after t2.join(), t2's joinable() is false
t3, after t3.detach(), t3's joinable() is false

-------------------------------------------------------------------

t4, t4's joinable() is true
t5, t5's joinable() is true

t6 was constructed by "std::move(t4)", so t4's joinable() is false, t6's joinable() is true
t7 was constructed by "= std::move(t5)", so t5's joinable() is false, t7's joinable() is true

-------------------------------------------------------------------

terminate called without an active exception
Aborted (core dumped)
```



上面的输出中应该已经比较清楚地说明了上面三点了。

最后的出错信息与本例无关，出错的原因是在整个程序结束时（main() 结束代表整个程序结束），还有线程正在运行。这里在下个问题解释。





### Q6. terminate called without an active exception?



报错信息：

```
terminate called without an active exception
Aborted (core dumped)
```

在 C++ 多线程编程中，该报错信息表示，有 std::thread 对象在销毁时（如离开作用域），其关联的线程仍然活跃（是 joinable 的）。

我们在 2.Q3. 提到过，只有调用 join() 或 detach() 后，线程资源才能被正确回收，并且这个调用必须在 std::thread 对象销毁前，因为其析构函数会直接终止 joinable 的线程，且会导致内存泄漏。

所以我们要保证在 std::thread 对象销毁前，根据需求，执行其成员函数 join() 或 detach() 即可解决问题。

还有个小坑容易踩：我们可以认为整个程序是一个最大的作用域，当整个程序结束时（main() 结束），同样可能出现此问题，比如声明过一个全局的 std::thread 对象却没有在合适的位置 join() 或 detach()。





### Q7. 如何向线程函数传递非 const 引用参数？

今天给线程函数传参时传了个引用，本以为可以像传指针那样，可以在线程内修改变量，但编译报错了，这里举一个代码示例如下：

```cpp
#include <iostream>
#include <thread>

void func(int& num) {
    num++;
}


int main(int argc, char *argv[]) {

    int a = 10;

    std::thread t(func, a);
    t.join();

    std::cout << "a =  " << a << std::endl;

    return 0;
}
```



报错关键信息如下：

```
/usr/include/c++/8/thread:120:17: error: static assertion failed: std::thread arguments must be invocable after conversion to rvalues
  static_assert( __is_invocable<typename decay<_Callable>::type,
                 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
           typename decay<_Args>::type...>::value,
```

最关键的一句话是 "std::thread arguments must be invocable after conversion to rvalues"，即参数必须是可调用的或者可以转成右值的。

可调用，指的是第一个参数，就是线程的函数（或其他调用）入口。

可转成右值，就是我们刚刚报错的主要原因，因为引用不能转成右值。

我们看一下这个构造函数的源码：

```cpp
  template<typename _Callable, typename... _Args,
      typename = _Require<__not_same<_Callable>>>
    explicit
    thread(_Callable&& __f, _Args&&... __args)
    {
static_assert( __is_invocable<typename decay<_Callable>::type,
            typename decay<_Args>::type...>::value,
  "std::thread arguments must be invocable after conversion to rvalues"
  );
```

可以看到其中有参数 `_Args&&... __a`，这里的 `&&` 表示的就是右值。我们刚刚的关键报错信息也在这里。

如何解决呢？首先，你可以选用其他方法实现同样的功能，比如指针，或者说 const 引用，因为 const 引用可以转成右值。

\-



**那如果我就想传递一个非 const 引用怎么办？**

其实是可以的，但在这之前，我们应该知道为什么不可以是非 const 引用。

我们知道线程是有自己的内存存储空间的，在 std::thread 类的构造中，参数会先按照默认方式复制到线程的存储空间中，然后新创建的线程才能访问它们。这些副本被当做临时变量，以**右值**的方式传递给新线程上的函数或者可调用对象。即便函数相关的参数按设想应该是引用（比如即便你在构造时传递的是 `int const &` 类型，新线程最后传递的其实还是个 `int &&`）。

按上面代码，参数 `num` 应当以引用方式传入的，但我们执行 `std::thread t(func, a);` 时，线程库还是会把 `a` 复制一份到新线程的内存存储空间中，然后将这个副本以 move-only（只能移动，不能复制） 的方式传递给一个右值给 `func()`，因为这个函数预期接收一个非 const 引用，右值不能转为非 const 引用，所以就会编译出错。

**解决方法：使用 `std::ref()`。** 将传递的非 const 引用用 `std::ref()` 包装。

```cpp
#include <iostream>
#include <thread>

void func(int& a) {
    a++;
}


int main(int argc, char *argv[]) {

    int a = 10;

    std::thread t(func, std::ref(a));
    t.join();

    std::cout << "a =  " << a << std::endl;

    return 0;
}
```

输出：

```
a =  11
```

可以看到程序如预期般正确运行了！





## 第 3 章：在线程间共享数据
