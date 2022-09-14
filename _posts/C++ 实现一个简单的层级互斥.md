---
title: "C++ 实现一个简单的层级互斥"
date: 2022-09-15 00:13:00
updated: 2022-09-15 01:33:00
categories: [并发编程]
tags: [Cpp,并发编程]
---

锁的层级划分就是按特定方式规定加锁次序，在运行期据此查验加锁操作是否遵从预设规则。按照构思，我们应该把应用程序分层，并且明确每个互斥位于哪个层级。若某线程已经对低层级互斥加锁，则不准它再对高层级互斥加锁。具体做法是将层级的编号赋予对应层级应用程序的互斥，并记录各线程分别锁定了哪些互斥。

层级互斥其实是比较常见的一种模式，但 C++ 标准库并未提供。我们这里实现一个简单的层级互斥，并且支持与 `std::lock_guard`、`std::scoped_lock` 等配套使用。要想我们自己实现的互斥支持 `std::lock_guard`、`std::scoped_lock`，有三个方法是必要的，即 `lock()`、`unlock()` 和 `try_lock()`。

我们下面看具体实现：

```cpp
#include <mutex>
#include <stdexcept>
#include <climits>

class hierarchical_mutex
{
    std::mutex internal_mutex;
    unsigned long const hierarchy_value;
    unsigned long previous_hierarchy_value;
    static thread_local unsigned long this_thread_hierarchy_value;

    void check_for_hierarchy_violation()
    {
        if (this_thread_hierarchy_value <= hierarchy_value)
        {
            throw std::logic_error("mutex hierarchy violated");
        }
    }
    void update_hierarchy_value()
    {
        previous_hierarchy_value = this_thread_hierarchy_value;
        this_thread_hierarchy_value = hierarchy_value;
    }

public:
    explicit hierarchical_mutex(unsigned long value) : hierarchy_value(value),
                                                       previous_hierarchy_value(0)
    {
    }
    void lock()
    {
        check_for_hierarchy_violation();
        internal_mutex.lock();
        update_hierarchy_value();
    }
    void unlock()
    {
        this_thread_hierarchy_value = previous_hierarchy_value;
        internal_mutex.unlock();
    }
    bool try_lock()
    {
        check_for_hierarchy_violation();
        if (!internal_mutex.try_lock())
            return false;
        update_hierarchy_value();
        return true;
    }
};
thread_local unsigned long
    hierarchical_mutex::this_thread_hierarchy_value(ULONG_MAX);

int main()
{
    hierarchical_mutex m1(42);
    hierarchical_mutex m2(2000);

    {
        // error: mutex hierarchy violated
        // order: m1 -> m2
        std::scoped_lock lk1(m1, m2);
    }
    {
        // correct
        // order: m2 -> m1
        std::scoped_lock lk2(m2, m1);
    }

    return 0;
}

```

这段代码有几个需要关注的点：

* `thread_local` 修饰的变量对每个线程都是独立的。`this_thread_hierarchy_value` 是线程专属变量，表示当前线程的层级编号（线程自身不属于任何层级，所以准确意义是当前线程最后一次加锁操作所牵涉的层级编号）。
* 因为 `this_thread_hierarchy_value` 声明由 `thread_local` 修饰，每个线程都具有自己的副本，所以该变量在某线程上的值与另一线程上的值完全无关。
* `this_thread_hierarchy_value` 的值初始化为 unsigned long 类型能表示的最大值 `ULONG_MAX`。故此，最开始时任意 hierachical_mutex 互斥都能被加锁，即任意 hierachical_mutex 互斥加锁都能通过检查。只要成功锁定，层级编号即能更新。
* 假设我们已持有 hierachical_mutex 的锁，那么 `this_thread_hierarchy_value` 的值反应出前者所在层级的编号，若要再对另一互斥加锁，后面互斥的层级必须低于前面已被锁定的互斥的层级，才能通过检查。
* 我们必须保证解锁的顺序严格是加锁的逆序，所以我们在给一个 hierachical_mutex 解锁时，必须保证其层级与当前线程的层级编号一致（见 `unlock()` 开始处的判断），才可以继续解锁。


>本文参考《C++ 并发编程实战（第 2 版）》