


C++ 中并发操作的同步手段主要有三种，按复杂程度从低到高依次为：

1. `std::async`；
2. `std::packaged_task`；
3. `std::promise`。



其中 `std::async`是函数模板，另外两个为类模板。三者都定义在头文件 `<future>` 中，且由**不可拷贝的** `std::future` 类型的对象接收事件的结果。



我们逐个介绍上述三种，并在其中顺带演示如何使用 `std::future` 对象。



## 1. `std::async`

最简单的方法，`std::async` 函数接收 1 个可调用对象（通常是函数）以及此调用对象的参数。例如：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::future<int> f = std::async(sum, 3, 4);
    std::cout << f.get() << std::endl; // print "7"
    
    return 0;
}
```

\-

第二种方法用于执行类的成员函数，接收要执行的成员函数、要执行该成员函数的对象（静态成员函数不需要）以及相关参数。例如：

```cpp
#include <iostream>
#include <future>

class ForSum {
public:
    int sum(int a, int b) {
       return a + b;
    }
    static int sum_static(int a, int b) {
        return a + b;
    }
};

int main(int, char**) {
    ForSum fsum;
    std::future<int> f1 = std::async(&ForSum::sum, &fsum, 3, 4);
    std::future<int> f2 = std::async(&ForSum::sum, fsum, 3, 4);
    std::future<int> f3 = std::async(&ForSum::sum_static, 3, 4);  // static member function
    std::cout << f1.get() << std::endl;  // print "7"
    std::cout << f2.get() << std::endl;  // print "7"
    std::cout << f3.get() << std::endl;  // print "7"

    return 0;
}
```

这里注意下 `f1` 和 `f2` 对应的参数 `&fsum` 和 `fsum` 不同，前者使用的是我们自己声明的对象 `fsum` 执行成员函数，而后者则是其副本。



\-

`std::async` 还可以指定执行指定函数的时机，在参数列表的**首位**添加一个参数，类型是 `std::launch`，其值可以是：

* `std::launch::deferred`: 在**当前线程上**延后调用任务函数，等到在 `std::future` 对象上调用了 `get()` 或 `wait()`，任务函数才会执行。
* `std::launch::async`：必须开启专属的线程，在其上运行任务函数。
* `std::launch::deferred | std::launch::async`：缺省值，由 `std::sync` 的实现自己选择运行方式。

举个例子，此例中任务函数 `sum()` 在调用 `f.get()` 时才开始执行：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::future<int> f = std::async(std::launch::deferred ,sum, 3, 4);
    std::cout << f.get() << std::endl; // print "7"
    
    return 0;
}
```



## 2. `std::packaged_task`

从 `std::packaged_task` 的名字可以看出，此类对象是一个打包好的任务，这里这个任务实则就是一个函数。

`std::packaged_task` 用法很简单，其模板参数是一个函数签名。例如，我们想打包一个含有两个类型分别为 `std::vector<char>*` 和 `int` 的参数，返回值类型为 `std::string` 的函数，那么实例化对象时，写法如下（关注 `<>` 中的 `std::string(std::vector<char>*, int)`）：

```cpp
std::packaged_task<std::string(std::vector<char>*, int)> task(f);
```

其中 `f` 为要打包的具体函数名（函数指针）。

注意上面的实例化对象声明时没有传入被打包函数的参数，被打包函数的参数将在调用 `task` 时传入。

`task` 的调用和函数直接调用 `f` 看起来区别不大，举一个具体的例子：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    task(2, 3);
    sum(2, 3);
    
    return 0;
}
```

上面的 `task(2, 3)` 和 `sum(2, 3)` 都是计算 2+3 的和，那区别是什么呢？

主要区别在于，**`std::packaged_task` 打包后，函数的调用和获取返回值可以在不同的线程中进行。**`sum(2, 3)` 的返回值 5 可以直接获取，并且函数的调用和获取返回值都是在当前线程中进行的。而 `task(2, 3)` 的调用，需要使用其成员函数 `get_future()` 获取 `std::future` 对象，通过此对象获取返回值，`std::future` 对象的特化类型取决于 `std::packaged_task` 对象声明时参数中函数签名指定的返回值。

我们扩展上面的例子：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    std::future f = task.get_future();
    task(2, 3);
    std::cout << "task(2, 3) 的返回值为 " << f.get() << std::endl;
    std::cout << "sum(2, 3) 的返回值为 " << sum(2, 3) << std::endl;  // sum(2, 3) 的返回值可以直接获取
    
    return 0;
}
```

输出如下：

```
task(2, 3) 的返回值为 5
sum(2, 3) 的返回值为 5
```

可以看到，我们使用 `std::future` 类型对象来接收 `task(2, 3)` 的结果。

但注意，此例子中，我们 `task(2, 3)` 的执行和结果获取仍是在同一个线程中进行的，这并没有发挥出其优势，仅用于举例。

下面的代码中，我们在另一个线程中执行此任务，在当前线程中获取并打印结果：

```cpp
#include <iostream>
#include <future>

int sum(int a, int b) {
    std::cout << "starting calculatation, the thread id is " 
              << std::this_thread::get_id() << std::endl;
    return a + b;
}

int main(int, char**) {
    std::packaged_task<int(int, int)> task(sum);
    std::future f = task.get_future();
    std::thread t(std::move(task), 2, 3);
    t.join();
    std::cout << "the result of task(2, 3) is " << f.get()
              << ", and the thread id is " 
              << std::this_thread::get_id() << std::endl;
    
    return 0;
}
```

输出：

```
starting calculatation, the thread id is 139965868611328
the result of task(2, 3) is 5, and the thread id is 139965886650176
```

可以看到，我们的计算线程和获取结果的线程是不同的。

\-



在上面关于 `std::async` 和 `std::packaged_task` 的例子中，我们演示了二者基本的使用。

有些读者可能会有和我一样的疑问，这两者似乎做了一样的事情（其一可以做的事情，另一个也都能做），但 `std::packaged_task` 更复杂些。这是为什么呢？

我的理解是，`std::packaged_task` 比 `std::async` 能多做的有：





* `std::packaged_task` 可由程序员决定任务何时开始（调用 `task()` 才开始），而 `std::async` 是由其内部实现决定的，程序员不可控。
* `std::packaged_task`可以指定一个特定线程来完成任务（对象可以移动，就可以移动给指定线程，由指定线程执行）。而 `std::async` 要么是当前线程，要么是一个新线程。





本文也主要是笔记，有理解不全或不对的地方，欢迎指出！



如果你不需要 `std::packaged_task`  额外的功能，那么 `std::async`  更简单。下面要说的 `std::promise` 比 `std::packaged_task` 能做的要更多。

## 3. `std::promise`

`std::promise<T>` 给出了一种异步求值的方法（类型为 `T`），某个 `std::future<T>` 对象与结果关联，能延后读出需要求取的值。配对的 `std::promise` 和 `std::future` 可实现下面的工作机制：等待数据的线程在 future 上阻塞，而提供数据的线程利用相配的 promise 设定关联的值，使 future 准备就绪。

若需从给定的 `std::promise` 实例获取关联的 `std::future` 对象，调用前者的成员函数 `get_future()` 即可，这与 `std::packaged_task` 一样。promise 的值通过成员函数 `set_value()` 设置，只要设置好，future 即准备就绪，凭借它就能获取该值。如果 std::promise 在被销毁时仍未曾设置值，保存的数据则由异常代替。后面的小节会介绍线程间如何传递异常。

> `std::promise` 可以在 future 中存储任意的值。在 `std::async` 和 `std::packaged_task` 中，future 中只能存储函数的返回值。





下面通过一个例子简单解释，在该例中：



* 线程 t1 负责计算 2 + 3 的值。
* 线程 t2 负责接收 t1 的计算结果并打印。
* 使用 `std::promise` 搭配 `std::future` 完成。



```cpp
#include <iostream>
#include <future>

void my_sum(int a, int b, std::promise<int> p)
{
    p.set_value(a + b);
    std::cout << "my_sum()    's thread id is " 
              << std::this_thread::get_id() 
              << ", my_sum() finished, "
              << std::endl;
}

void print_sum(std::future<int> f)
{
    std::cout << "print_sum() 's thread id is "
              << std::this_thread::get_id()
              << ", the sum of my_sum() is "
              << f.get() << std::endl;
}

int main(int, char **)
{

    std::promise<int> p;
    std::future f = p.get_future();

    std::cout << "main()      's thread id is " 
              << std::this_thread::get_id() 
              << std::endl;

    std::thread t1(my_sum, 2, 3, std::move(p));
    std::thread t2(print_sum, std::move(f));

    t1.join();
    t2.join();

    return 0;
}
```

输出：

```
main()      's thread id is 140445886191424
my_sum()    's thread id is 140445868152576, my_sum() finished, 
print_sum() 's thread id is 140445859759872, the sum of my_sum() is 5
```

可以看出，`main()`、`my_sum()` 和 `print_sum()` 分别在三个不同的线程中执行，并且我们的程序结果符合预期。







## 4. 将异常保存到 `std::future` 中





使用 `std::async` 和 `std::packaged_task`，如果发生异常，那么异常将会被存储到对应 future 中，而不会立即抛出，调用其 `get()` 方法时才会抛出此异常。

> C++ 标准没有明确规定应该重新抛出原来的异常，还是其副本；为此，不同编译器和库有不同的选择。



对于 `std::promise`，因为其对应的 future 是调用 `set_value()` 方法设置的，所以不会自动存储异常，而需要我们使用 `set_expection()` 方法代替 `set_value()`，从而在 future 中存储异常。`set_exception()` 的调用可以放在 `catch` 段中，例如：

```cpp
extern std::promise<double> some_promise;

try
{
    some_promise.set_value(calculate_value());
}
catch(...)
{
    some_promise.set_exception(std::current_exception());
}
```

上述伪代码中可以看到，当程序正确运行时，我们使用 `set_value()` 设置值，当程序异常时在 `catch` 段使用`set_exception()` 存储异常。最后，该异常同样会在 future 上调用 `get()` 时抛出。

这里的 `std::current_exception()` 用于捕获抛出的异常。此外，我们还能用 `` 直接保存新异常，而不触发抛出行为。

```cpp
some_promise.set_exception(std::make_exception_ptr(std::logic_error("foo ")));
```

如果我们能预知异常的类型，那么，相较 try/catch 块，后面的代替方法不仅简化了代码，还更有利于编译器优化代码，因而应优先采用。



还有一种方法可将异常保存到 future 中：我们不调用 promise 的两个 set 成员函数，也不执行包装的任务，而是直接销毁与 future 关联的 `std::promise` 或 `std::packaged_task` 对象。如果关联的 future 未能准备就绪，无论销毁两者中的哪一个，其析构函数都会将异常 `std::future_error` 存储为异步任务的状态数据，它的值是错误代码 `std::future_errc::broken_promise`。我们一旦创建 future 对象，便是许诺会按异步方式给出值或异常，但可以销毁他们的生产来源，就无法提供所求的值或出现的异常，导致许诺被破坏。在这种情形下，倘若编译器不向 future 存入任何数据，则等待的线程有可能永远等不到结果。





到目前位置，所有代码范例都使用了 `std::future`。然而，`std::future` 自身存在限制，关键问题是：它只容许一个线程等待结果。若我们要让多个线程等待同一个目标时间，则需要改用 `std::shared_future`。

## 5. 多个线程一起等待 `std::shared_future`





`std::future` 只能让一个线程等待结果，其 `get()` 只能调用一次，因为调用后会触发移动操作，其内的值将不存在。

并且 `std::future` 不会自动同步，也就是多线程同时访问一个 `std::future` 且没有做任何同步处理的话，可能导致资源竞争，出现未定义的结果。



`std::future` 仅支持移动构造和移动赋值，无法拷贝，所以虽然它可以在多个线程中移动，但同一时刻仅能有一个线程持有其实例。而 `std::shared_future` 就解决了这个问题，因为其支持拷贝。所以我们可以持有该类的多个对象，且全部指向同一异步任务的状态数据。



这里需要注意的是，即便使用了 `std::shared_future`，但如果多个线程访问同一个对象而不做同步，依然会出现数据竞争，并且每个 `std::shared_future` 对象的 `get()` 同样只能调用一次，多次调用会报错。正确的方法是，**给每个线程传递一个 `std::shared_future` 副本，这样每个副本作为其线程内的局部变量，标准库会解决访问时的竞争问题。**通过线程自有的 `std::shared_future` 副本来访问状态数据是安全的。





由于不论是 `std::async` 返回的，还是我们通过 `get_future()` 方法从 `std::packaged_task` 和 `std::promise` 中获取的都是 `std::future` 对象，而不是 `std::shared_future` 对象。所以我们要想使用 `std::shared_future`，就需要从 `std::future` 转换而来。



从 `std::future` 转换到 `std::shared_future` 有几种方法，区别不大，这里举例说明：



```cpp
std::promise<int> p;
std::shared_future sf = p.get_future();  // 隐式转换 std::future 为 std::shared_future
std::shared_future sf = p.get_future().share();  // 显式转换

// f is a std::future object
std::shared_future sf = f; // 隐式转换，f 将失效，f.valid() 为 false
std::shared_future sf = f.share();  // 显式转换，f 将失效，f.valid() 为 false
```

这里  `std::shared_future<T>` 中的 `T` 和 `std::future` 同样可自动推导，所以可以不写。



我们修改第 3 小节中的示例代码，让两个线程打印计算的结果：

```cpp
#include <iostream>
#include <future>

void my_sum(int a, int b, std::promise<int> p)
{
    p.set_value(a + b);
    std::cout << "my_sum()    's thread id is " 
              << std::this_thread::get_id() 
              << ", my_sum() finished, "
              << std::endl;
}

// void print_sum(std::shared_future<int>& sf)  // 错误，每个线程内使用的都是同一个 std::shared_future 对象
void print_sum(std::shared_future<int> sf)  // 正确，每个线程内使用的都是传入参数的副本
{
    std::cout << "print_sum() 's thread id is "
              << std::this_thread::get_id()
              << ", the sum of my_sum() is "
              << sf.get() << std::endl;
}

int main(int, char **)
{

    std::promise<int> p;
    std::shared_future sf = p.get_future().share();

    std::cout << "main()      's thread id is " 
              << std::this_thread::get_id() 
              << std::endl;

    std::thread t1(my_sum, 2, 3, std::move(p));
    std::thread t2(print_sum, sf);
    std::thread t3(print_sum, sf);

    t1.join();
    t2.join();
    t3.join();

    return 0;

}
```

结果如下：

```
main()      's thread id is 140658286561088
my_sum()    's thread id is 140658268522240, my_sum() finished, 
print_sum() 's thread id is 140658260129536, the sum of my_sum() is 5
print_sum() 's thread id is 140658117519104, the sum of my_sum() is 5
```

符合预期，两个执行 `print_sum()` 的线程都正确打印出了计算结果。





### 6. 限时等待



我们前面介绍的内容中，线程在等待 future 就绪前会阻塞，而且是无休止的阻塞。有时我们想设定一个超时时间，如果设定时间到达后 future 还没有就绪，那么就转而去做其他事情。

C++ 有两种超时(timeout)机制可用：

* 迟延超时(duration-based timeout)：线程根据指定的时长而继续等待（如 30 毫秒）。
* 绝对超时(absolute timeout)：在某特定的时间点(time point)来临之前，线程一直等待、

大部分等待函数都具有变体，专门处理这两种机制的超时。处理迟延超时的函数变体以 `_for` 为后缀，而处理绝对超时的函数变体以 `_until` 为后缀。

例如，条件变量 `std::condition_variable` 含有成员函数 `wait_for()` 和 `wait_until()`，它们各自具备两个重载，分别对应 `wait()` 的两个重载：其中一个重载停止等待的条件是收到信号、超时、或发生伪唤醒；我们需要向另一个重载函数提供断言，在对应线程被唤醒之时，只有该断言城里（向条件变量发送信号），它才会返回，如果超时，这个重载函数也会返回。





> TODO
