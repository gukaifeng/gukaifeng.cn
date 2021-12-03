---
title: Rust -- 泛型、trait 与生命周期
date: 2021-12-02 18:15:09
updated: 2021-12-02 18:15:09
categories: [编程语言基础]
tags: [Rust]
toc: true
---



简单来说，**泛型**就是一个减少重复的编程技术。泛型概念在大部分编程语言中都有，学习 Rust 的人一般都有其他语言的基础，这里就不详细介绍什么是泛型了。

**trait** 是 Rust 的一个特性，这使你可以以一个通用的方式定义行为。trait 可以与泛型结合，将泛型限制为拥有特定行为的类型，而不是任意类型。

**生命周期(lifetimes)**也是 Rust 中的一个特性，生命周期其实就是各种泛型，这些泛型为编译器提供引用之间如何相互关联的信息。Rust 的生命周期允许我们在许多情况下借用值，同时仍然允许编译器检查引用是否有效。



上面对泛型、trait 和生命周期的介绍可能不是很好理解，简单有个印象就好，我们在下面通过例子详细来说。

<!--more-->

## 1. 泛型



### 1.1. 在函数定义中使用泛型

下面的代码定义了一个泛型函数，获取一个数组中的最大值。

由于使用的是泛型，所以数组成员的类型可以是 `i32`，`f64`，`char` 等等。

这里就通过代码看看如何使用即可：

在函数声明中的函数名字右侧，用 `<>` 来包含你要使用的泛型类型（常用 `T`，`U` 等等，也可以是任意的，如果你要同时使用多个泛型类型，在 `<>` 中用逗号 `，` 隔开），然后你就可以在函数的参数、返回值、函数体中使用你在 `<>` 包含的泛型类型了。 

注意，使用同一个泛型的成员、参数、返回值等等，类型必须相同。如果要想不同，请使用不同的泛型。

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```



### 1.2. 在结构体定义中使用泛型



#### 1.2.1. 结构体定义

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

同样地，在结构体声明处，结构体名字的右侧，用 `<>` 包含泛型类型。

注意，使用同一个泛型的成员、参数、返回值等等，类型必须相同。如果要想不同，请使用不同的泛型。例如，如果你想让 x 和 y 是不同类型的（上面的代码都是 `T`，x 和 y 类型必须相同），可以再定义一个泛型：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

`T` 和 `U` 可以相同，也可以不同，不同泛型之间是相互独立互不影响的。



#### 1.2.2. 方法定义

下面的代码为 1.2.1 中第一个结构体定义方法：

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

主要方法定义那一块，我们的结构体是 `Point<T>`，在 `impl` 关键字后也要加 `<T>`，这个地方的 `<>` 和结构体名字后的 `<>` 必须是一样的，这样才能指定为泛型结构体 `Point<T>` 定义方法。

-

你可能会奇怪，为什么要像 `impl<T> Point<T> {` 重复写两个 `<T>`，不多余吗？

其实这是因为 Rust 还提供了一个功能，就是为指定的泛型类型定义专有的方法。

简单的说，我们可以定义一个方法，当 `T` 为 `i32` 时，`Point<i32>` 有这个方法，而 `T` 为其他值的时候，例如 `Point<f64>` 就没有这个方法！

当指定 `T` 为一个具体类型的时候，在 `impl` 后就不用加 `<T>` 了，当 `T` 没有指定的时候，在 `impl` 后面加 `<T>`，这样 Rust 才知道 `T` 是泛型，而不是具体类型（泛型的名字是随意的，不一定是 `T`，你也可以自己定义一个名字为 `T` 的类型，所以 Rust 需要你指定哪个才是泛型）。

我们下面再看一个例子，这个例子仅当 `T` 为 `i32` 的时候，给 `Point<i32>` 定义一个方法，当 `T` 为其他类型时，`Point<T>` 是没有这个方法的！

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

-

（这里应该从，方法可以使用与结构体不同的泛型开始）

### 1.3. 在枚举定义中使用泛型

我们之前已经见过一些使用了泛型定义的枚举了，

例如 `Option` 枚举和 `Result` 枚举：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

同样地，在结构体声明处，结构体名字的右侧，用 `<>` 包含泛型类型。

同样要注意，使用同一个泛型的成员，类型必须相同。



### 1.4. 泛型代码的性能问题

