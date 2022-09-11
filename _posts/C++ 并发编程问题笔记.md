---
title: C++ 并发编程问题笔记
date: 2022-09-11 15:38:00
updated: 2022-09-11 15:38:00
categories: [并发编程]
tags: [Cpp,并发编程]
---

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

> A2：是的

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

> A3：不是，应当根据需求判断。

要理解这个问题，首先我们要清楚 t.join() 和 t.detach() 做了什么，明白为什么要调用它们。

* join() 在概念上与 fork() 相对，fork 是分叉的意思，而 join 是汇合的意思。我们知道调用 t.join() 会阻塞当前线程直到 t 关联的线程 **（记为 t_thread）** 执行完成，这**看起来**就像把 t_thread 线程的内容的代码放到了 t.join() 调用处一样。

* t.detach() 会将线程 t_thread 与当前线程分离，t_thread 将在后台执行。分离后当前线程将无法再与  t_thread 直接通信，也无法等待 t_thread 完成，无法汇合 t_thread（即调用 t.join()）。t_thread 确实还在后台执行，但其归属权和控制权都将转移给 C++ 运行时库（runtime library，又名运行库），由此保证线程退出时与之关联的资源可以被正确回收。

**什么时候调用 t.join() 呢？**

我们需要当前线程等待 t_thread 执行结束再进行下一步时。例如，t_thread 用到了当前线程中的某局部变量，当前线程未来会销毁此局部变量，这时就需要调用 t.join() 保证 t_thread 执行完了以后再继续当前线程。

**那什么时候调用 t.detach() 呢？**

比如 t_thread 是一个守护进程，守护进程常运行在整个程序的生命周期内，以执行一些任务，比如文件系统监控、从对象缓冲中清除无用数据项等。

或者，由 t_thread 执行“启动后即可自主完成”（a fire-and-forget tast）的任务。例如，在一个支持多标签页的文本编辑器中，每个线程负责一个标签页。新建标签页时，由 t_thread 创建一个和其自己一样的线程来处理新标签页，这时就需要 detach，t_thread 和其复制从此各干各的（各负责一个标签页）。

我们还能通过分离线程实现一套机制，用于确认线程完成运行。

\-

我们知道了何时应该调用 join() 或 detach()，即调用它们的原因。那么反过来，如果我们没有调用它们的需求，那就不必调用。

我们通过上面描述的 *什么时候调用 t.join() 呢？* 举一个例子，如果 t_thread 用到了当前线程的局部变量，我们肯定要等待 t_thread 执行完成才能让当前线程继续推进（到可能销毁此局部变量的位置），所以需要调用 t.join()。但如果说，即便不调用 t.join() 阻塞当前线程，我们也能 100% 保证 t_thread 会在局部变量销毁前执行完毕（例如通过锁实现），那就可以不用调用 t.join()。

detach() 就更好理解了，我们没有让 t_thread 在后台运行的需求，就没必要调用。

### Q4. 为什么 t.join() 或 t.detach() 只能调用一次（两者共享一次），且是否可调用的判断条件均为 t.joinable() ?
