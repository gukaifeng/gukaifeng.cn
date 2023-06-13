


**多线程：**多线程（multithreading），是指从软件或者硬件上实现多个线程并发执行的技术。具有多线程能力的计算机因有硬件支持而能够在同一时间执行多于一个线程，进而提升整体处理性能。在一个程序中，这些独立运行的程序片段叫作“线程”（Thread），利用它编程的概念就叫作 “多线程处理（Multithreading）”。



## 1. Thead类

Java 中的 JVM 允许程序运行多个线程，它通过 java.lang.Thread 类来实现。

<!--more-->

### 1.1. Thread类的特性

1. 每个线程都是通过某个特定的 Thread 对象的 run() 方法来完成操作的。

    经常把 run() 方法的主体称为线程体。

2. 通过该 Thread 对象的 start() 方法来调用这个线程。

### 1.2. Thread类的构造方法

```java
Thread(); // 创建新的 Thread 对象
Thread(String threadname); // 创建线程并指定线程实例名
Thread(Runnable target); // 指定创建线程的目标对象，实现了 Runnable 接口中 run() 方法
Thread(Runnable target, String name); // 创建新的 Thread 对象
```

### 1.3. Thread类的有关方法

| Methods                                                      | Functions                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public void start()                                          | 启动线程                                                     |
| public void run()                                            | 线程被调度后执行的操作                                       |
| public final String getName()                                | 返回线程的名称                                               |
| void setName(String name)                                    | 设置线程的名称                                               |
| public static Thread currentThread()                         | 返回当前的线程                                               |
| public final int getPriority()                               | 返回当前线程的优先级                                         |
| public final void setPriority(int newPriority)               | 设置新的线程优先级                                           |
| public static void yield()                                   | 进程让步                                                     |
| public static void sleep(long millis) throws InterruptedException | 占着 CPU，休眠                                               |
| ~~public final void stop()~~                                 | ~~强制终止线程~~                                             |
| public final boolean isAlive()                               | 判断线程是否还活着                                           |
| public final void join() throws InterruptedException         | 当某个程序执行流中调用其他线程的 join() 方法时，阻塞当前线程，直到被 join 线程执行完毕。 |

注：线程的优先级控制（线程创建时继承父类线程的优先级）。

1. MAX_PRIORITY (10);
2. MIN_PRIORITY (1);
3. NORM_PRIORITY (5); `默认值`



## 2. 创建线程的两种方式

### 2.1.继承 Thread 类

1. 定义子类继承 Thread 类；
2. 子类中重写 Thread 类中的 run() 方法；
3. 创建 Thread 子类对象，即创建了线程对象；
4. 调用线程对象的 start() 方法：启动线程，调用 run() 方法。

代码示例：

```java
// 继承了 Thread 的类
package TestThread;

public class Thread1 extends Thread {
    @Override
    public void run() {
        System.out.println("Thread1's code.");
        for(int i = 0; i < 5; ++i)
            System.out.println("this is Thread1's code " + i);
    }
}
```

```java
// 主程序
package TestThread;

public class ThreadMain {
    public static void main(String[] args) {
        Thread t1 = new Thread1();
        t1.start(); // 启动线程，若直接使用 run() 就是普通调用
        System.out.println("-------1-------");
        System.out.println("-------2-------");
        System.out.println("-------3-------");
    }
}

```

```
// 输出 线程之间并发
-------1-------
Thread1's code.
-------2-------
-------3-------
this is Thread1's code 0
this is Thread1's code 1
this is Thread1's code 2
this is Thread1's code 3
this is Thread1's code 4
```

### 2.2. 实现 Runnable 接口（常用）

1. 定义子类，实现 Runnable 接口；
2. 子类中重写 Runnable 接口中的 run() 方法；
3. 通过 Thread 类的含参构造器创建线程对象；
4. 将 Runnable 接口的子类对象作为实际参数传递给 Thread 类的构造方法中；
5. 调用 Thread 类的 start() 方法：开启线程，调用 Runnable 子类接口的 run() 方法。

代码示例：

```java
// 实现 Runnable 接口的类
package TestThread;

public class Thread2 implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " " +  "Thread2's code.");
        for(int i = 0; i < 5; ++i)
            System.out.println(Thread.currentThread().getName() + " " +  "this is Thread2's code " + i);
    }
}
```

```java
// 主程序
package TestThread;

public class ThreadMain {
    public static void main(String[] args) {
        Thread t2 = new Thread(new Thread2());
        Thread t3 = new Thread(new Thread2(), "Thread-3");
        t2.start();
        t3.start();
        System.out.println("-------1-------");
        System.out.println("-------2-------");
        System.out.println("-------3-------");
    }
}
```

```
// 输出
-------1-------
-------2-------
-------3-------
Thread-0 Thread2's code.
Thread-3 Thread2's code.
Thread-3 this is Thread2's code 0
Thread-3 this is Thread2's code 1
Thread-3 this is Thread2's code 2
Thread-3 this is Thread2's code 3
Thread-0 this is Thread2's code 0
Thread-3 this is Thread2's code 4
Thread-0 this is Thread2's code 1
Thread-0 this is Thread2's code 2
Thread-0 this is Thread2's code 3
Thread-0 this is Thread2's code 4
```



## 3. 多线程资源共享

使用实现接口的方法，可以避免单继承的局限性，多个线程也可以共享资源。

共享资源代码示例：

```java
// 实现 Runnable 接口的类
package TestThread;

public class Thread2 implements Runnable {
    int count = 0;
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " " +  "Thread2's code.");
        for(int i = 0; i < 5; ++i, ++count)
            System.out.println(Thread.currentThread().getName() + " " +  "this is Thread1's code " + count);
    }
}
```

```java
// 主程序，线程之间共享变量 count
package TestThread;

public class ThreadMain {
    public static void main(String[] args) {
        Runnable r = new Thread2();
        Thread t1 = new Thread(r, "Thread-1");
        Thread t2 = new Thread(r, "Thread-2");
        t1.start();
        t2.start();
    }
}

```

```
// 输出
Thread-1 Thread2's code.
Thread-2 Thread2's code.
Thread-2 this is Thread1's code 0
Thread-2 this is Thread1's code 1
Thread-2 this is Thread1's code 2
Thread-2 this is Thread1's code 3
Thread-2 this is Thread1's code 4
Thread-1 this is Thread1's code 0
Thread-1 this is Thread1's code 6
Thread-1 this is Thread1's code 7
Thread-1 this is Thread1's code 8
Thread-1 this is Thread1's code 9
```

## 4. 多线程程序的优点

1. 提高应用程序的响应速度。对图形界面更有意义，提高用户体验；
2. 提高 CPU 利用率；
3. 改成程序结构。将既长又复杂的进程分为多个线程，提高可读性，便于维护。

## 5. 线程的声明周期

JDK 中用 Thread.State 枚举表示了线程的几种状态

线程的一个完整声明周期中通常要经历五种状态：新建、就绪、运行、阻塞、思维。

## 6. 线程的同步问题

参照 OS 中的进程同步、死锁。

Java 中提供了用于解决多线程同步的机制 `synchronized` 同步锁。

```java
// synchronized 可以放在方法声明中，表示整个方法为修饰方法，一个线程的该方法执行完前，不会执行其他线程的该方法（事实上是🔐的整个对象，不同的对象是不同的🔐，如果对象之间也需要同步（如全局变量），就不可用）。如果在 static 方法上加🔐，这个🔐就是对所有对象有效。
public synchronized void show(String name) { ... }

// 调用 synchronized() 实现同步
// 如 synchronized(this) { // 同步代码 } 表示给当前对象的代码块加锁
// synchronized(this) 锁住的多个代码块，共享一个锁，可以实现同步
// 若要实现多个锁，在方法内传递该对象的参数，外部调用时把对象本身也传进去
synchronized(object obj) { // 需同步代码 }
```

## 7. 线程通信

| Methods     | Functions                                |
| ----------- | ---------------------------------------- |
| wait()      | 挂起                                     |
| notify()    | 唤醒同步资源的等待队列指中优先最高的线程 |
| notifyAll() | 唤醒同步资源的等待队列中的全部线程       |

java.lang.Object 提供的这三个方法只有在 synchronized 方法或 synchronized 代码块中才能使用，否则会报 java.lang.IllegalMonitorStateException 异常

