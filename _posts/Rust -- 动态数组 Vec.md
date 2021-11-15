---
title: Rust -- 动态数组 Vec
date: 2021-10-26 23:44:41
updated: 2021-10-26 23:44:41
categories: [编程语言概念]
tags: [Rust]
toc: true
---



`Vec` 是 Rust 标准库中提供的集合之一。

`Vec` 一种连续的可增长数组类型，所有存储数据类型必须一致，写作 `Vec<T>`，发音为 "vector"。

原型如下：

```rust
pub struct Vec<T, A = Global> 
where
    A: Allocator, 
 { /* fields omitted */ }
```



这篇文章只会介绍几个基础用法，用以入门，详细内容请看官方文档 [Vec in std::vec - Rust](https://doc.rust-lang.org/stable/std/vec/struct.Vec.html)。

<!--more-->

## 1. 新建 Vec

新建 `Vec` 主要有两种方式：

1\. 调用函数 `Vec::new()` 创建一个空的 `Vec`；

```rust
pub const fn new() -> Vec<T, Global>
```

在元素被推入 vector 之前，vector 不会分配。

2\. 调用宏 `vec!` 从现有元素创建一个 `Vec`。



下面从代码示例了解如何新建 `Vec`：

```rust
let mut v0 = Vec::new();  // 新建一个空的 Vec，其中的类型 T 将在插入第一个元素时由 Rust 推断，要注意编译前必须有插入操作，否则会编译失败，Rust 需要在编译前就知道其中元素的类型。
let mut v1: Vec<i32> = Vec::new();  // 新建一个空的 Vec，指定元素类型为 i32

let mut v2 = vec![1, 2, 3];  // 从现有元素新建一个 Vec，其中的类型 T 将根据给定的元素时 Rust 推断
let mut v3: Vec<i32> = vec![1, 2, 3];  // 从现有元素新建一个 Vec，指定元素类型为 i32
```

**请注意，上面的几个写法均要加关键字 `mut`，否则创建出的 Vec 将是不可变的，你不能插入或者修改、删除元素。**





## 2. 更新 Vec



### 2.1. 在最后插入一个元素 `push`

```rust
pub fn push(&mut self, value: T)
```

下面看示例代码：

```rust
#![allow(unused)]
fn main() {
    let mut vec = vec![1, 2];
    vec.push(3);
    assert_eq!(vec, [1, 2, 3]);
}
```



### 2.2. `pop` 删除最后一个元素并返回其值

从 vector 中移除最后一个元素并返回它，如果为空则返回 `None`。

```rust
pub fn pop(&mut self) -> Option<T>
```

下面看示例代码：

```rust
#![allow(unused)]
fn main() {
    let mut vec = vec![1, 2, 3];
    assert_eq!(vec.pop(), Some(3));
    assert_eq!(vec, [1, 2]);
}
```

这里要注意的是，当 vector 为空的时候，调用 `pop()` 并不会报错，而是会返回 `None`，所以这里判断返回值的时候应该使用 `Option<T>`，即 `Some(T)` 和 `None`。



## 3. 读取单个元素

读取单个元素主要有两种方法：

1\. 通过 `[]` 运算符访问；

2\. 通过函数 `get()` 访问。

下面的代码中使用了分别使用了上述两种方法来获取 `v` 中索引为 `2` 的元素：

```rust
#![allow(unused)]
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {}", third);

    match v.get(2) {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
}
```

这两种方式是有所不同的。使用 `[]` 则和大多数编程语言类似，如果索引非法，则会使程序崩溃，Rust 中则是 Panic ！但是如果使用的是 `get()` 函数，返回的是 `Option<&T>`，即当索引非法的时候，会返回 `None`，也就需要额外的分支判断。

还有一个区别是，你可以通过 `[]` 访问的方式来修改 vector 中的值（前提是 Vec 声明时为 `mut`），但是 `get()` 不可以，因为其返回的 `Option<&T>` 是不可变引用。若想使用此方式修改 vector，使用函数 `get_mut()` 其返回的是 `Option<&mut T>`。

下面是 `get_mut()` 的示例代码：

```rust
#![allow(unused)]
fn main() {
    let x = &mut [0, 1, 2];

    if let Some(elem) = x.get_mut(1) {
        *elem = 42;
    }
    assert_eq!(x, &[0, 42, 2]);
}
```



## 4. 遍历 Vec

Rust 中遍历 vector 有两种常用的方法：  
一种是通过索引，这和大部分其他编程语言一样；  
另一种是通过 `for` 来获取 vector 中的每一个元素的可变或不可变引用。



### 4.1. 通过索引访问 Vec



如果你想通过索引访问整个 vector，则需要知道整个 vector 的长度，Rust 中的 `Vec` 提供了函数 `len()`。

当然通过索引，你也可以更方便的访问其中某一段的元素。

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    
    for i in 0..v.len() {  // 打印 vector 中的全部值
        println!("{}", v[i]);
    }
    for i in 0..v.len() {  // 给 vector 中的全部值都加 1
        v[i] += 1;
    }
    for i in 0..2 {  // 打印索引 0 和 1 的值
        println!("{}", v[i]);
    }
}
```





### 4.2. 通过引用来访问 Vec

如果想要依次访问 vector 中的每一个元素，我们可以使用引用来遍历其所有的元素，而无需通过索引一个一个来。

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    
    for i in &v {
        println!("{}", i);  // 打印 vector 中的全部值
    }
    for i in &mut v {
        *i += 1;  // 给 vector 中的全部值都加 1
    }
}
```

这里要注意 `*i += 1;` 中，在 `i` 前面加了 `*` 解引用，这是因为 `i` 是一个引用（可变引用），我们可以通过引用访问其值，但是若要修改就需要解引用。

 





## 5. 扩展：一个常见的错误

下面看一段代码：

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {}", first);
}
```

虽然这段代码看起来没什么问题，但是编译时还是会报错如下：

```
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 | 
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 | 
8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
```



我在 [Rust -- 所有权(Ownership)](https://gukaifeng.cn/archives/34/) 这篇文章中，有一个关于引用的总结如下：

>1. 在同一时间，对于一个值，你最多可以有一个可变引用，可以有任意数量的不可变引用；
>2. 引用必须始终有效；
>3. 如果你有一个（或多个）不可变引用，那么从这个不可变引用的声明开始，直到这个不可变引用的最后一次使用，其间不可以引入（声明）可变引用。当引入（声明）了可变引用以后，这个不可变引用将不再可用；
>4. 如果你有一个可变引用，那么从这个可变引用的声明开始，直到这个可变引用的最后一次使用，其间不可以引入（声明）不可变引用。当引入（声明）了不可变引用以后，这个可变引用将不再可用。



现在再看上面的代码，报错信息显示在 `first = &v[0];` 处发生了不可变引用，而 `v.push(6);` 处发生了可变引用，最后打印 `first` 的时候使用了不可变引用，

我们知道三处错误所说的引用目标，只能是 `v[0]`，但是为什么 `v.push(6);` 在 `v` 的末端插入元素会影响到 `v[0]` 呢？这跟 Rust 中 `Vec` 的工作方式有关：在 vector 的结尾增加新元素时，若没有足够空间将所有所有元素依次相邻存放，可能会要求分配新内存并将旧的元素拷贝到新的空间中。这时，第一个元素 `v[0]` 的引用就指向了被释放的内存，所以 `push()` 操作会创建一个 `v[0]` 的可变引用（虽然不一定用得到）来改变其所引用的位置。

---

那我们把 `let first = &v[0];` 改为 `let mut first = &v[0];` 会怎样呢？

就像下面的代码这样，这样就不存在可变引用与不可变引用共存的问题了。

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &mut v[0];

    v.push(6);

    println!("The first element is: {}", first);
}
```

这段代码也是错误的，结合上一段错误代码的分析，再看我们之前的总结第一条：

>1. 在同一时间，对于一个值，你最多可以有一个可变引用，可以有任意数量的不可变引用；

我们的 `let first = &mut v[0];` 和 `v.push(6);` 都会创建一个 `v[0]` 的可变引用，这是非法的！

