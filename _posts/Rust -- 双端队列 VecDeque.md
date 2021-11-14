---
title: Rust -- 双端队列 VecDeque
date: 2021-10-27 09:12:37
categories: [Technology]
tags: [Rust]
---



`VecDeque` 是 Rust 标准库中提供的一个使用可增长的循环缓冲区实现的双端队列。

你可以只使用 `push_back()` 函数从末端插入，使用 `pop_front()` 函数从前端删除，这样就是一个普通队列。

`VecDeque` 不可以在代码中直接使用，需要 `use std::collections::VecDeque;`。



`VecDeque` 原型如下：

```rust
pub struct VecDeque<T, A = Global> 
where
    A: Allocator, 
 { /* fields omitted */ }
```



这篇文章只会介绍几个基础用法，用以入门，详细内容请看官方文档 [VecDeque in std::collections - Rust](https://doc.rust-lang.org/stable/std/collections/struct.VecDeque.html)。



<!--more-->

## 1. 新建 VecDeque

新建 VecDeque 主要有两种方式：

1\. 创建一个空的 VecDeque；

2\. 从现有元素创建一个 VecDeque。

下面通过代码来看：

```rust
#![allow(unused)]

use std::collections::VecDeque;

fn main() {
  let mut vd0: VecDeque<u32> = VecDeque::new();  // 创建一个空的 VecDeque，其中元素类型为 u32
  let mut vd1: VecDeque<i32> = VecDeque::from([-1, 0, 1]);  // 从现有数据创建 VecDeque，其中元素类型为 i32
  
  let mut vd2 = VecDeque::new();  // 创建一个空的 VecDeque，元素类型将有 Rust 推断，要注意编译前必须有插入操作，否则会编译失败，Rust 需要在编译前就知道其中元素的类型。
  let mut vd3 = VecDeque::from([-1, 0, 1]);  // 从现有数据创建 VecDeque，其中元素类型由 Rust 推断
}
```



## 2. 更新 VecDeque



### 2.1. 插入元素

```rust
pub fn push_back(&mut self, value: T)
```

```rust
pub fn push_front(&mut self, value: T)
```

`push_back()` 在队尾插入元素。

 `push_front` 在队头插入元素。





### 2.2. 删除元素

```rust
pub fn pop_back(&mut self) -> Option<T>
```

```rust
pub fn pop_front(&mut self) -> Option<T>
```

`pop_back()` 删除队尾的元素，返回 `Option<T>`，即返回被删除的元素或返回 `None`。

`pop_front()` 删除队头元素，返回 `Option<T>`，即返回被删除的元素或返回 `None`。



## 3. 访问 VecDeque



### 3.1. 获取队头元素

```rust
pub fn front(&self) -> Option<&T>
```

```rust
pub fn front_mut(&mut self) -> Option<&mut T>
```

`front()` 和 `front_mut()` 函数都可以获取队头元素，`front()` 返回 `Option<&T>`，`front_mut()` 返回 `Option<&mut T>`，即你可以通过 `front_mut()` 修改队头元素，但 `front()` 不可以。

下面通过两段代码看这两个函数的用法：

```rust
#![allow(unused)]

use std::collections::VecDeque;

fn main() {
    let mut d = VecDeque::new();
    assert_eq!(d.front(), None);

    d.push_back(1);
    d.push_back(2);
    assert_eq!(d.front(), Some(&1));
}
```

```rust
#![allow(unused)]

use std::collections::VecDeque;

fn main() {
    let mut d = VecDeque::new();
    assert_eq!(d.front_mut(), None);

    d.push_back(1);
    d.push_back(2);
    match d.front_mut() {
        Some(x) => *x = 9,
        None => (),
    }
    assert_eq!(d.front(), Some(&9));
}
```



### 3.2. 获取队尾元素

```rust
pub fn back(&self) -> Option<&T>
```

```rust
pub fn back_mut(&mut self) -> Option<&mut T>
```

`back()` 和 `back_mut()` 函数都可以获取队尾元素，`back()` 返回 `Option<&T>`，`back_mut()` 返回 `Option<&mut T>`，即你可以通过 `back_mut()` 修改队尾元素，但 `back()` 不可以。

下面通过两段代码看这两个函数的用法：

```rust
#![allow(unused)]

use std::collections::VecDeque;

fn main() {
    let mut d = VecDeque::new();
    assert_eq!(d.back(), None);

    d.push_back(1);
    d.push_back(2);
    assert_eq!(d.back(), Some(&2));
}
```

```rust
#![allow(unused)]

use std::collections::VecDeque;

fn main() {
    let mut d = VecDeque::new();
    assert_eq!(d.back(), None);

    d.push_back(1);
    d.push_back(2);
    match d.back_mut() {
        Some(x) => *x = 9,
        None => (),
    }
    assert_eq!(d.back(), Some(&9));
}
```
