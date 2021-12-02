---
title: Rust -- 所有权(Ownership)
date: 2021-09-13 21:57:16
updated: 2021-09-13 21:57:16
categories: [编程语言基础]
tags: [Rust]
toc: true
---

**所有权(Ownership)**是 Rust 最独特的特性，它使 Rust 能够在不需要垃圾回收器(Garbage collector)的情况下保证内存安全。  
因此，理解 Rust 中的所有权如何工作是非常重要的。  
这篇文章里，我们将介绍 Rust 中的所有权以及几个相关特性：**借用(Borrowing)**、**切片(Slice)**以及 **Rust 如何在内存中放置数据**。

<!--more-->

## 1. 什么是所有权？



Rust 的核心特征是所有权。虽然这个特性很容易解释，但它对 Rust 语言的其余部分有很深的影响。

所有程序在运行时都必须管理它们使用计算机内存的方式。  
有些编程语言（比如 Java、Go 语言）有垃圾收集(Garbage collection)，它在程序运行时不断地寻找不再使用的内存；  
在另一些编程语言（比如 C/C++）中，程序员必须显式地分配和释放内存。

**Rust 使用了第三种方法：内存由所有权系统管理，所有权系统有一系列用来分配和释放内存的规则，编译器会在编译时检查这些规则。所有的所有权特性都不会在程序运行时减慢程序的速度。**

对很多程序员来说，所有权是一个比较新的概念，需要一些时间来适应。好消息是，你在 Rust 和所有权系统的规则方面的经验越多，你就越能自然地开发出安全高效的 Rust 代码。

当你理解了所有权，你就有了一个坚实的基础去理解所有权这个使得 Rust 独一无二的特性。  
这篇博客将通过一些例子介绍所有权，这些例子集中在一个非常常见的数据结构：字符串(String)。



> 在阅读后面的内容前，你需要对栈(Stack)和堆(Heap) 、变量的作用域（Rust 中和其他语言没有区别）、浅拷贝与深拷贝的区别有一定的了解。



### 1.1. 所有权规则

先看看 Rust 中的所有权规则，我们需要理解并记忆这些规则，后面会通过例子解释这些规则。

1. Rust 中的每个**值(value)**都有一个**变量(variable)**，称为**所有者(owner)**；
2. 同一个值，在同一时刻，有且仅有一个所有者；
3. 当所有者（即变量）离开其作用域后，这个值将被**删除(dropped)**。



  ### 1.2. Rust 中的 `String` 类型

这里需要提前简单介绍一下 Rust 中的 `String` 类型，才好继续介绍所有权。

更全面的 `String` 类型介绍看这篇文章：

我们之前见过字符串的大多都是字面值，字面值会被被硬编码到我们的程序中，快速且高效。字面值非常方便，原因之一是字面值不可变。但字面值不适合很多场景，比如我们在编译的时候不知道它的内容应该是什么，也不知道它会占用多大的内存空间。一个最简单的场景是，我们要存储一个由用户输入的字符串内容，这时候字符串字面值就不适用了。

Rust 中的 `String` 类型在堆上分配，因此能够存储我们在编译时未知的大量文本，下面看几个简单用法

```rust
let s = String::from("hello");
```

双冒号 `::` 是一个运算符，这个运算符和 C/C++ 等一些编程语言中的差不多。在上面的代码中，`::` 使得我们可以使用**命名空间(namespace)** `String` 下的一个名为 `from` 的特定函数。如果你对 C/C++ 这类语言有了解，那么你应该对  `::` 和 命名空间的概念都不陌生。关于命名空间的概念不是本篇文章要讲的，这里就先略过了。

再看下面的代码

```rust
fn main() {
    let mut s = String::from("hello");
    println!("{}", s);
    s.push_str(", world!"); // push_str() appends a literal to a String
    println!("{}", s); // This will print `hello, world!`
}
```

输出

```
hello
hello, world!
```

这回定义了一个可变的字符串 `s`，并使用其的成员函数 `push_str()` 在后面加了一段字符。

关于 Rust 中的 `String` 简单说到这里，就可以开始下面的内容了~





### 1.3. 内存与分配



对于字符串字面值，我们在编译前就知道其内容，所以编译器可以直接将其直接硬编码到最终的可执行程序中。这是字面值快速高效的原因，但这个特性也仅仅是因为字面值不可变罢了。但是我们不可能把一个在编译时不知道大小、不知道内容，而且在程序运行过程中大小和内容可能改变的内存放入一个二进制文件中。

Rust 中的 `String` 类型，为了支持其内容可变，其大小可变，需要在堆上分配一定量的内存，但在编译时我们并不知道应该分配多大的内存给它。

这就意味着

1. 内存需要在程序运行时向操作系统请求分配；
2. 当我们不再需要这个 `String` 的时候，我们要把它使用的内存还给操作系统。

对于第 1 点，是非常普遍的，大多数编程语言都这么做，我们之前用到的 `String::from()` 已经替我们做了这件事。

对于第 2 点，是比较特殊的，不同编程语言有着不一样的实现方式。一些语言使用垃圾回收器(GC, garbage collector)，GC 持续的查看追踪并清理不再使用的内存，我们不需要考虑其具体怎么做的。在没有 GC 的情况下，我们就有责任做好内存的分配和回收工作，当不再使用一块内存事，调用某段代码将其还给操作系统。但事实上这个难度是非常大的，一旦有些疏忽，就会造成一些影响很大的问题，例如忘记释放内存、过早的释放内存、重复释放内存等。

Rust 采用了一个与众不同的方式来回收内存：**当一个变量离开其作用域时，它所使用的内存将被自动回收。**

下面用一个简单的代码说明 Rust的这个方式

```rust
fn main() {
    {
        let s = String::from("hello"); // s is valid from this point forward
        // do stuff with s
    }                                  // this scope is now over, and s is no
                                       // longer valid
}
```

当离开 `s` 的作用域时，我们可以自然地将 `String` 所使用的内存还给给分配器。

当一个变量超出作用域时，Rust 会为我们调用一个名为 `drop` 特殊函数,，`String` 的所有者可以在这个函数中放置代码来释放内存。Rust会在右花括号处自动调用 `drop`。



>注意:在c++中，这种在 item 生命周期结束时释放资源的模式有时被称为 Resource Acquisition is Initialization (RAII)。如果你使用过 RAII 模式，那么 Rust 中的 drop 函数将会非常熟悉。



这种模式对 Rust 代码的编写方式有着深远的影响。

上面的情景看起来可能很简单，但在更复杂的情况下，例如当我们想让多个变量使用在堆上分配的数据时，代码的行为可能会出乎意料。

下面让我们探讨其中一些情况。



#### 1.3.1. 变量和数据的交互方式 1: Move

在 Rust 中，多个变量可以通过不同的方式与同一数据交互。下面看个简单的例子

```rust
fn main() {
    let x = 5;
    let y = x;
}
```

我们大概可以猜到上面的代码在做什么：”将值 `5` 绑定到 `x`；然后复制 `x` 中的值，并将其绑定到 `y`“。  
我们现在有两个变量，`x` 和 `y`，都等于 `5`。因为整数是已知的、固定大小的简单值，这两个同为 `5` 的值会被压入栈。

下面再看看 `String` 的版本

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
}
```

这看起来与前面的代码非常相似，所以我们可以假设它的工作方式是相同的。  
也就是说，第二行将复制 `s1` 中的值并将其绑定到 `s2`，**但事实并非如此**。

我们看下图，Rust 中的 `String` 由 3 部分组成，一种指向存储字符串内容的指针 `ptr`、一个 `len` 和 一个 `capacity`。**左图中的内容存在栈上，右图中的内容存在堆上（`String` 中存储的数据在堆上）。**

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_01.svg" style="width:300px;" />

和很多一样，我们 `s2 = s1` 进行的是类似**浅拷贝**的操作，即只拷贝了左图中的 `ptr`、`len` 和 `capacity`。现在左图的信息有两份，即 `s1` 和 `s2`，但右图中的内容还是只有一份，`s1` 和 `s2` 在共用同一块堆上的数据，两个指针指向同一块内存空间，如下图。

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_02.svg" style="width:300px;" />

如果是**深拷贝**的话，那应该是对右图中的内容也拷贝一份（在堆中分配一块同样大的空间，然后拷贝数据过来），结果应该如下图这样。



<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_03.svg" style="width:300px;" />

如果 Rust 执行了深拷贝，那么当右图中的内容（即堆中数据）很大的情况下，`s2 = s1` 这个语句执行的代价可能是巨大的。

既然在 `s2 = s1` 这个语句执行时，Rust 的策略类似浅拷贝，那前面说过，当一个变量离开其作用域时，Rust 会自动调用 `drop` 函数，并为该变量清理堆内存。但类似浅拷贝的策略下，`s1` 和 `s2` 中的数据指针 `ptr` 指向同一个位置。当 `s1`和 `s2` 离开作用域时，它们都将试图释放相同的内存。这被称为 `double free` 错误，是我们前面提到的内存安全错误之一。释放内存两次可能会导致内存损坏，从而可能导致安全漏洞。

为了确保内存安全，在这种情况下 Rust 不再尝试复制已分配的内存，而是认为 `s1` 不再有效，因此，当 `s1` 离开作用域时，Rust 不需要释放任何东西。下面通过代码看看在创建了 `s2` 之后尝试使用 `s1` 时会发生什么！

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
}
```

编译报错（这里只看 error，忽略 warning）

```
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 | 
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move

For more information about this error, try `rustc --explain E0382`.
```

这里真想称赞一下 Rust 的报错信息如此清晰~~（看 C/C++ 的报错信息简直要吐血）~~

第 5 行错误信息显示，`s1` 是 `String` 类型，执行了 move 操作，而不是 copy。  
第 7 行错误信息显示，`s1` 的内容移动到了 `s2` 这里。  
第 10 行错误信息显示，`s1` 的值在执行完 move 操作后已经被**借走(borrowed)**了。

关于借用的概念我们会在后面说，这里先有个大致的印象就可以。

现在就很清楚了，在执行完 `s2 = s1` 以后，Rust 执行了类似浅拷贝的操作，然后禁用了 `s1`（而不是像大多编程语言那样，浅拷贝后 `s1` 和 `s2` 都是可用的），`s1` 就相当于不存在了，所以在 `s1` 离开作用域的时候，Rust 不需要释放任何内存，而由于 `s1` 原本的内容已经 move 到了 `s2` 这里，所以当 `s2` 离开作用域时，Rust 才需要释放内存（右图中的那部分）。

**上面我一直在描述 Rust 的这个策略是类似浅拷贝的，而不是浅拷贝，因为 Rust 使得一开始的 `s1` 变得无效了，而浅拷贝这个概念本身并没有这个操作，所以这里一直说的都是类似浅拷贝。**

**在 Rust 中，上述的操作就被称为移动(Move)。**

在上面的代码中，实际发生的事应该是  
Rust 拷贝（类似浅拷贝）了一份 `s1`，即 `s2`，然后 `s2` 实际占有了 `s1` 的 `ptr` 指针所指向的内存空间，`s1` 之后失效。只有当 `s2` 离开作用域时，其使用的堆内存才会被释放。

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_04.svg" style="width:300px;" />



此外，这还暗示了一个设计选择：Rust 永远不会自动创建数据的**深度**副本。

因此，就运行时性能而言，Rust 中任何自动的复制都是廉价的。



#### 1.3.2. 变量和数据的交互方式 2: Clone



如果我们确实需要深拷贝 `String` 的堆数据，而不仅仅是栈数据，Rust 提供了一种名为 `clone` 的通用方法。

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```

输出

```
s1 = hello, s2 = hello
```

这段代码真正实现了图中的结果。  
但当我们实际这样做的时候，还是要提醒自己，当堆中的数据过大的时候，这个操作的代价可能是很大的。

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_03.svg" style="width:300px;" />



#### 1.3.3. 变量和数据的交互方式 3 (仅在栈上的数据): Copy

我们在 1.3.1. 节一开始的时候，简单讨论下一段代码，

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
}
```

为什么这样的代码，没有使用 `clone` 却是有效的呢？

原因是，在编译时具有已知大小的类型（如整数）完全存储在栈上，因此可以快速复制其实际值。这意味着我们没有理由在创建变量 `y` 后使 `x` 变得无效。换句话说，这里的深拷贝和浅拷贝没有区别，所以调用 `clone` 与通常的浅拷贝没有什么不同，我们可以忽略它。

Rust 有一个叫做 `Copy` trait 的特殊注释，我们可以把它用在类似整型这样存储在栈的类型上。如果一个类型实现了 `Copy` trait，那么在把旧变量的值赋值给其他变量后，旧变量仍然可用。如果一个类型或其任何部分实现了 `Drop` trait，Rust 就不允许其再使用  `Copy` trait。如果我们对一个当其值离开作用域时需要做一些特殊处理的类型使用 `Copy` trait，就会得到一个编译错误。

**关于 trait 的具体介绍[看篇文章]()。**

那么具体哪些类型实现了 `Copy` 特性呢？你可以通过查看给定类型的文档来了解。  
但作为一般规则，任何由简单标量值组合的类型都可以实现 `Copy`，任何需要分配或某种形式资源的类型都不能实现 `Copy`。

下面是一些常见的实现了 `Copy` trait  的类型

* 所有的整数类型，比如 `u32`；
* 所有的浮点数类型，比如 `f64`；
* 布尔类型 `bool`；
* 字符类型 `char`；
* 只包含同样实现了 `Copy` trait 类型的元组类型，比如 (`i32`, `i32`) 可以，但 (`i32`, `String`) 不可以。



### 1.4. 所有权与函数

向函数传递值的语义类似于向变量赋值的语义。将变量传递给函数会移动或复制，就像赋值一样。

下面的代码有一个示例，其中一些注释显示了变量进入和超出作用域的地方，注意区分 `move` 和 `copy`。

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

输出

```
hello
5
```

如果我们试图在调用 `takes_ownership` 之后使用 `s`, Rust 会抛出一个编译时错误。这些静态检查可以防止我们出错。你可以自己尝试在 `main` 中添加使用了 `s` 和 `x` 的代码，看看哪些地方可以使用它们，以及在所有权规则之下，哪些地方不能使用。









### 1.5. 返回值与作用域

返回值也可以转移所有权。

下面通过代码来理解，注意区分注释中的 `move` 和 `copy`。

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("hello"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// takes_and_gives_back will take a String and return one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```



变量的所有权总是遵循同样的模式：当把一个值赋值给另一个变量时，其所有权就会被移动(move)。当一个包含了堆中数据的变量离开其作用域时，其值就会被 `drop` 清理掉，除非其值的所有权已经被移动给了另一个变量。

对于一个函数，先获得一个值的所有权，然后进行一些操作后，再返回(return)所有权，是有些无聊的。如果我们想让一个函数使用一个值，但不获取其所有权，该怎么办？非常恼人的是，如果我们想再次使用传入函数的任何东西，我们的函数除了要返回一些我们需要的数据外，还需要额外返回这个值。就像下面的代码

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

上面的代码中，我们有一个值为 `hello` 的 `String`，我们使用函数 `calculate_length()` 来计算并返回它的长度，但是由于我们后面还要使用宏 `println!` 打印这个字符串的内容，就需要函数 `calculate_length` 除了返回字符串的长度外，还要把这个字符串一起返回。

这种场景在开发中是非常非常常见的，但像上面代码中那么做显然不太合适。幸运的是，Rust 为了解决这个问题，提供了一个功能，叫做**引用(References)**。



## 2. 引用(References)与借用(Borrowing)



### 2.1. 普通引用（不可变引用）



我们期望使用一个变量，但不获取其所有权，在 Rust 中这可以通过引用实现。

看下面的代码

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {  // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, nothing happens.
```

输出

```
The length of 'hello' is 5.
```

注意我们将 `&s1` 传递给 `calculate_length()`，并且在 `calculate_length()` 的定义中，我们使用 `&String` 而不是 `String`。

**这里 `&` 符号代表引用(references)**，它可以让你的引用一些值，而无需获取其所有权。

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_05.svg" style="width:300px;" />

我们可以这样理解，变量 `s` 中实际存储的只是一个指向 `s1` 的指针，而不是像一个普通 `String` 类型那样存储指向堆中数据指针、`len`、`capacity` 等信息。然后我们在通过 `s` 来使用变量 `s1` 的值时，可以直接把 `s` 当成 `s1` 本身来用，和 `s1` 用法完全相同（仅仅是看起来相同哈~）。

变量 `s` 有效的作用域与任何函数形参的作用域相同，但当它超出作用域时，我们不会删除引用指向的对象，因为我们没有所有权。当函数使用引用作为参数而不是实际值时，我们将不需要用返回值来返回所有权，因为我们从未拥有所有权。

Rust 中将获取引用作为函数参数的行为称为**借用(Borrowing)**。就像在现实生活中，如果一个人拥有某样东西，你可以向他借，但当你用完的时候，你得把它还回去。



### 2.2. 可变引用(Mutable References)



> 如果一个人拥有某样东西，你可以向他借，但当你用完的时候，你得把它还回去。  
> 如果你在没有提前和人家说的情况下对他的东西做一些改动，比如在借来的书中涂改，可能会让人很生气。  
> Rust 中的引用也是这样会的。

**Rust 中的普通引用不能修改被引用值的内容！**看下面的代码

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

这段代码编译时会报错

```
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
```



正如 Rust 中的变量默认不可以修改一样，Rust 中的引用默认也不可以修改。



要想让上面的代码编译通过，我们需要做一些改动。

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

首先，我们必须把 `s` 变成 `mut` 的，然后我们必须使用 `&mut s` 创建一个可变引用，并使用 `some_string: &mut String` 接受一个可变引用。现在这段代码就是可以编译通过的，注意，从定义 `s` 开始，到 `&mut s`，再到函数 `change()` 的参数类型 `&mut String`，都要有 `mut` 关键字，任何一个地方少了都不行！

---



**但是可变引用有一个很大的限制：对特定范围内的特定数据块只能有一个可变引用。**

下面这段代码将会编译失败

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
}
```

编译报错

```
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 | 
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
```

报错信息显示，以可变的方式借用 `s` 的次数同一时间最多只能有一次。

这种限制就要要求你必须以一个非常可控的方式通过引用修改变量的值，这对于新 Rustaceans 是比较难适应的，毕竟大部分编程语言都可以让你随便修它。

有这个限制的好处是 Rust 可以防止在编译时的**数据竞争(data race)**。  
数据竞争类似于竞争条件，当以下三种行为发生时就会发生：

* 两个或多个指针同时访问相同的数据；
* 至少有一个指针被用于写入数据；
* 没有用于同步数据访问的机制。

数据竞争会导致未定义的行为，你可能很难诊断和修复由于数据竞争造成的 bug。  
Rust 阻止了这个问题的发生，因为在 Rust 中带有数据竞争的代码就不会通过编译！

我们修改下上面的代码，把 `r1` 放进一个花括号里，这样 `r1` 的作用域限于花括号以内，`当 r1` 离开其作用域时，我们就可以创建一个新的引用了。

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
}
```

---

把不可变引用与可变引用组合使用，可能也会导致编译时错误，我们看下面的代码

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```

编译报错

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 | 
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
```

这说明，当我们有一个不可变引用时，我们也不能有一个可变引用。

使用不可变引用的用户可不会希望它在用的值突然被改变了。

但是，多个不可变引用是可以的，因为仅仅读取数据的人没有能力影响其他人读取数据的能力。



**注意，引用的作用域从它被引入的地方开始，一直持续到最后一次使用该引用。**

例如，这段代码可以编译通过，因为不可变引用的最后一次使用发生在引入可变引用之前。

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
}
```

输出

```
hello and hello
hello
```



### 2.3. 悬垂引用(Dangling References)

在一些使用了指针的编程语言中，很容易错误地搞出来一个悬垂指针。通过释放一些内存，同时保留指向这些内存的指针，那这些指针就变成了悬垂指针，其指向的内存空间已经不属于其原来的应用程序了，甚至可能已经被操作系统分配给了其他应用程序，继续通过这些指针操作内存空间的话，很容易导致不可预期的后果。

相反，在 Rust 中，由编译器保证引用永远不会是悬垂引用。如果你有一个对某些值的引用，编译器将确保这个值不会在它的引用离开作用域之前离开作用域，也就是说，只有值的所有引用都已经离开了这些引用的作用域，这个值才可以离开这个值的作用域。

---



让我们尝试创建一个悬垂引用，Rust 会用一个编译时错误阻止它：

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

编译错误

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                ^^^^^^^^

For more information about this error, try `rustc --explain E0106`.
```

这个错误信息有关我们还没有介绍的一个 Rust 特性：生命周期(lifetimes)。

关于 Rust 中的声明周期，查看这里。

但是，如果我们先忽略生命周期部分，消息确实包含了为什么这段代码存在问题的关键：

`this function's return type contains a borrowed value, but there is no value for it to be borrowed from`

这句话显示，我们的函数的返回值类型包含了一个被借来的值，但是没有值可以借。

分析一下我们刚刚的 `dangle()` 函数

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

因为 `s` 是在 `dangle()` 中创建的，当 `dangle()` 的代码完成时，`s` 将被释放。但我们试图返回对它的引用。这意味着这个引用将指向一个无效的 `String`。Rust 不允许这么做！

这里的解决方案是直接返回 `String` 本身：

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

现在这段代码就没有问题了，`s` 的所有权被转移，并且没有任何东西被释放掉。



### 2.4. 引用规则总结

回顾一下我们之前讲过的，整理一下引用的规则。

1. 在同一时间，对于一个值，你最多可以有一个可变引用，可以有任意数量的不可变引用；
2. 引用必须始终有效；
3. 如果你有一个（或多个）不可变引用，那么从这个不可变引用的声明开始，直到这个不可变引用的最后一次使用，其间不可以引入（声明）可变引用。当引入（声明）了可变引用以后，这个不可变引用将不再可用；
4. 如果你有一个可变引用，那么从这个可变引用的声明开始，直到这个可变引用的最后一次使用，其间不可以引入（声明）不可变引用。当引入（声明）了不可变引用以后，这个可变引用将不再可用。





接下来，我们将研究一种不同类型的引用：**切片(Slice)**。



## 3. 切片(Slice)







Rust 中除了引用，另一个没有所有权的数据类型是**切片(slice)**。Slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。

> 这里有一个编程小习题：编写一个函数，该函数接收一个字符串，并返回在该字符串中找到的第一个单词。如果函数在该字符串中并未找到空格，则整个字符串就是一个单词，所以应该返回整个字符串。

让我们考虑一下这个函数的声明：

```rust
fn first_word(s: &String) -> ?
```

`first_word` 函数有一个参数 `&String`。因为我们不需要所有权，所以这没有问题。不过应该返回什么呢？我们并没有一个真正获取**部分**字符串的办法。不过，我们可以返回单词结尾的索引。试试下面的代码



```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

`first_word` 函数返回 `String` 参数的一个字节索引值。

因为需要逐个元素的检查 `String` 中的值是否为空格，需要用 `as_bytes()` 方法将 `String` 转化为字节数组：

```rust
let bytes = s.as_bytes();
```

接下来，使用 `iter` 方法在字节数组上创建一个迭代器：

```rust
for (i, &item) in bytes.iter().enumerate() {
```

迭代器不是本篇文章要说的，这里先不展开说它。暂时你只需要知道 `iter` 方法返回集合中的每一个元素，而 `enumerate` 包装了 `iter` 的结果，将这些元素作为元组的一部分来返回。`enumerate` 返回的元组中，第一个元素是索引，第二个元素是集合中元素的引用，这比我们自己计算索引要方便一些。

因为 `enumerate` 方法返回一个元组，我们可以方便的解构，所以在 `for` 循环中，我们指定了一个模式，其中元组中的 `i` 是索引而元组中的 `&item` 是单个字节。因为我们从 `.iter().enumerate()` 中获取了集合元素的引用，所以模式中使用了 `&`。

在 `for` 循环中，我们通过字节的字面值语法来寻找代表空格的字节。如果找到了一个空格，返回它的位置。否则，使用 `s.len()` 返回字符串的长度：

```rust
    if item == b' ' {
        return i;
    }
}

s.len()
```

现在有了一个找到字符串中第一个单词结尾索引的方法，不过这有一个问题。我们返回了一个独立的 `usize`，不过它只在 `&String` 的上下文中才是一个有意义的数字。换句话说，因为它是一个与 `String` 相分离的值，无法保证将来它仍然有效。考虑一下下面的代码

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 的值为 5

    s.clear(); // 这清空了字符串，使其等于 ""

    // word 在此处的值仍然是 5，
    // 但是没有更多的字符串让我们可以有效地应用数值 5。word 的值现在完全无效！
}
```

示例 4-8：存储 `first_word` 函数调用的返回值并接着改变 `String` 的内容

这个程序编译时没有任何错误，而且在调用 `s.clear()` 之后使用 `word` 也不会出错。因为 `word` 与 `s` 状态完全没有联系，所以 `word `仍然包含值 `5`。可以尝试用值 `5` 来提取变量 `s` 的第一个单词，不过这是有 bug 的，因为在我们将 `5` 保存到 `word` 之后 `s` 的内容已经改变。

我们不得不时刻担心 `word` 的索引与 `s` 中的数据不再同步，这很啰嗦且易出错！如果编写这么一个 `second_word` 函数的话，管理索引这件事将更加容易出问题。它的声明看起来像这样：

```rust
fn second_word(s: &String) -> (usize, usize) {
```

现在我们要跟踪一个开始索引和一个结尾索引，同时有了更多从数据的某个特定状态计算而来的值，但都完全没有与这个状态相关联。现在有三个飘忽不定的不相关变量需要保持同步。

幸运的是，Rust 为这个问题提供了一个解决方法：字符串分片(string slices)。

### 

### 3.1. 字符串分片(String Slices)



#### 3.1.1. 字符串分片(String Slices)



**字符串分片(string slice)**是 `String` 中一部分值的引用，它看起来像这样：

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

这类似于引用整个 `String` 不过带有额外的 `[0..5]` 部分。它不是对整个 `String` 的引用，而是对部分 `String` 的引用。

可以使用一个由中括号中的 `[starting_index..ending_index]` **（左闭右开）**指定的 range 创建一个 slice，其中 `starting_index` 是 slice 的第一个位置，`ending_index` 则是 slice 最后一个位置的后一个值。在其内部，slice 的数据结构存储了 slice 的开始位置和长度，长度对应于 `ending_index` 减去 `starting_index` 的值。所以对于 `let world = &s[6..11];` 的情况，`world` 将是一个包含指向 `s` 第 7 个字节（索引为 6）的指针和长度值 5 的 slice。

下图展示了一个示例。

<img src="https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/Rust--ownership_06.svg" style="width:300px;" />

对于 Rust 的 `..` range 语法，如果想要从第一个索引 0 开始，可以不写两个点号之前的值。如下两个语句是相同的：

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

依此类推，如果 slice 包含 `String` 的最后一个字节，也可以舍弃尾部的数字。如下也是相同的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

也可以同时舍弃这两个值来获取整个字符串的 slice。所以如下亦是相同的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：字符串 slice range 的索引必须位于有效的 UTF-8 字符边界内，如果尝试从一个多字节字符的中间位置创建字符串 slice，则程序将会因错误而退出。出于介绍字符串 slice 的目的，本部分假设只使用 ASCII 字符集。

在记住所有这些知识后，让我们重写 `first_word` 来返回一个 slice。字符串 slice 的类型声明写作 `&str`：

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

我们还是用相同的方式获取单词结尾的索引，通过寻找第一个出现的空格。当找到一个空格，我们返回一个字符串 slice，它使用字符串的开始和空格的索引作为开始和结束的索引。

现在当调用 `first_word` 时，会返回与底层数据关联的单个值。这个值由一个 slice 开始位置的引用和 slice 中元素的数量组成。

`second_word` 函数也可以改为返回一个 slice：

```rust
fn second_word(s: &String) -> &str {
```

现在我们有了一个不易混淆且直观的 API 了，因为我们之前说过编译器会确保指向 `String` 的引用持续有效。还记得之前的程序中，那个当我们获取第一个单词结尾的索引后，接着就清除了字符串导致索引就无效的 bug 吗？那些代码在逻辑上是不正确的，但却没有显示任何直接的错误。问题会在之后尝试对空字符串使用第一个单词的索引时出现。slice 就不可能出现这种 bug 并让我们更早的知道出问题了。使用 slice 版本的 `first_word` 会抛出一个编译时错误：

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // 错误!

    println!("the first word is: {}", word);
}
```

这里是编译错误：

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here
```

回忆一下借用规则，当拥有某值的不可变引用时，就不能再获取一个可变引用。因为 `clear` 需要清空 `String`，它尝试获取一个可变引用，Rust不允许这样做，因而编译失败。

Rust 不仅使得我们的 API 简单易用，也在编译时就消除了一整类的错误！

#### 3.1.2. 字符串字面值就是 slice

还记得我们讲到过字符串字面值被储存在二进制文件中吗？现在知道 slice 了，我们就可以正确地理解字符串字面值了：

```rust
let s = "Hello, world!";
```

这里 `s` 的类型是 `&str`：它是一个指向二进制程序特定位置的 slice。这也就是为什么字符串字面值是不可变的；`&str` 是一个不可变引用。

#### 3.1.3. 字符串 slice 作为参数

在知道了能够获取字面值和 `String` 的 slice 后，我们对 `first_word` 做了改进，这是它的声明：

```rust
fn first_word(s: &String) -> &str {
```

而更有经验的 Rustacean 会编写出下面这样的，因为它使得可以对 `&String` 值和 `&str` 值使用相同的函数：

```rust
fn first_word(s: &str) -> &str {
```

如果有一个字符串 slice，可以直接传递它。如果有一个 `String`，则可以传递整个 `String` 的 slice。定义一个获取字符串 slice 而不是 `String` 引用的函数使得我们的 API 更加通用并且不会丢失任何功能：

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let my_string = String::from("hello world");

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which is equivalent
    // to a slice of the whole `String`
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### 3.2. 其他类型的 slice

字符串 slice，正如你想象的那样，是针对字符串的。

不过也有更通用的 slice 类型，考虑一下这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

就跟我们想要获取字符串的一部分那样，我们也会想要引用数组的一部分。我们可以这样做：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

这个 slice 的类型是 `&[i32]`。它跟字符串 slice 的工作方式一样，通过存储第一个集合元素的引用和一个集合总长度。你可以对其他所有集合使用这类 slice。

## 4. 总结

所有权、借用和 slice 这些概念让 Rust 程序在编译时确保内存安全。Rust 语言提供了跟其他系统编程语言相同的方式来控制你使用的内存，但拥有数据所有者在离开作用域后自动清除其数据的功能意味着你无须额外编写和调试相关的控制代码。





























