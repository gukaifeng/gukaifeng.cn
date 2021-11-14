---
title: Rust -- 结构体
date: 2021-09-29 00:19:00
categories: [Technology]
tags: [Rust]
---



## 1. 定义结构体

Rust 中定义结构体的关键字为 `struct`，和 C 语言一样。用法如下示例：

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

以其中 `username: String,` 语句为例，`usrname` 是成员变量名，`String` 是成员变量类型。  
每个成员声明之后都还有个 `,` 不要忘了（最后一个可以不写），`struct` 最后不用写分号。

<!--more-->

## 2. 实例化结构体



### 2.1. 普通实例化

上面定义了结构体，下面示例看如何实例化结构体：

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

和大多 `let` 语句大体没什么区别，看看后面的格式就好，给各个成员变量赋值。  
同样记得给每个成员变量赋值后面都有个 `,`（最后一个可以不写），`let` 语句最后不要忘记加分号。  
另外实例化的时候给每个变量的赋值顺序和上面结构体定义中的顺序不需要相同。

注意上面的 `let user1`，`user1` 是不可变的，要想其可变，需要加 `mut` 关键字，即改为 `let mut user1`。  
Rust 中不允许把单个成员变量声明为  `mut`，只能在实例化的时候给整个结构体声明为 `mut`。即要能改就全能改，要不能改就都不能改。



### 2.2. 以函数返回值实例化

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

上面的代码比较简单，就是通过函数参数设置结构体中的 `email` 和 `username`，然后把另外两个变量设个值，返回。

大部分编程语言都是像上面这么写的，在 Rust 中可以简化一下。下面看代码：

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

当结构体初始化的时候，如果成员变量的名字和要赋值的函数参数名字一样，就可以省略参数。上面代码中从 `email: email,` 变为了 `email,`，`username: username,` 变为了 `username,`，其他不变，记得每个赋值及时省略了参数，最后也还是要加 `,`（最后一个可以不写）。



### 2.3. 从另一个结构体示例来创建新实例

假设我们已经有一个 `User` 类型的实例 `user1`，现在要实例化一个 `user2`。

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

上面的代码单独设置了 `email` 和 `username` 两个成员，然后另外两个成员设成了和 `user1` 中一样的值。

在大部分语言中，这样写是没什么问题的，但是在 Rust 中，这也可以简化。看代码：

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

注意上面的写法格式，我们先单独赋值了 `email` 和 `username`，然后通过 `..user1` 把 `user2` 中剩下的没有赋值的变量设成和 `user1` 中对应变量一样的值。







## 3. 获取结构体中变量的值

使用 `.` 符号。

对于普通结构体，直接使用 `.` 后接成员变量名访问指定成员即可。

例如 `user1.email`。

要是想修改值的话，直接赋值就行，例如  `user1.email = '123@456.com'`，但前提是 `user1` 声明为 `mut`。



## 4. 结构体中数据的所有权

在上面的实例代码中，我们使用了自身拥有所有权的 `String` 类型而不是 `&str` 字符串 slice 类型。这是一个有意而为之的选择，因为我们想要这个结构体拥有它所有的数据，为此只要整个结构体是有效的，其数据就也是有效的。

可以使结构体存储被其他对象拥有的数据的引用，不过这么做的话需要用上**生命周期(lifetimes)**。生命周期确保结构体引用的数据有效性跟结构体本身保持一致。如果你尝试在结构体中存储一个引用而不指定生命周期将是无效的，比如这样：

文件名: src/main.rs

```rust
struct User {
    username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```

我们会在编译器的报错信息中发现关于提示需要生命周期标识符的内容：

```text
error[E0106]: missing lifetime specifier
 --> src/main.rs:2:15
  |
2 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 | struct User<'a> {
2 |     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:3:12
  |
3 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 | struct User<'a> {
2 |     username: &str,
3 |     email: &'a str,
  |

For more information about this error, try `rustc --explain E0106`.
```

现在，我们先使用像 `String` 这类拥有所有权的类型来替代 `&str` 这样的引用以修正这个错误。

在这篇文章中我们会详细介绍生命周期相关概念，以及如何修复这个问题以便在结构体中存储引用，





## 5. 其他结构体

### 5.1. 元组结构体

元组结构体只有结构体名字，成员变量类型，而没有成员变量类型。

例如

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

上面的代码就定义了分别名为 `Color` 和 `Point` 的结构体，然后各实例化了一个对象。

要注意 `black` 和 `origin` 类型是不同，不可以进行互相赋值、函数传参什么的操作，虽然成员类型都一样。

获取元组结构体的值，使用 `.` 符号后街索引，索引从 0 开始，例如 `black.0`，`black.2`。修改也一样，前提是对象声明为了 `mut`。



### 5.2. 类单元结构体

Rust 中也可以定义一个没有任何字段的结构体。这种结构体被称为**类单元结构体(unit-like structs)**，因为它们类似于 `()`，即 unit 类型。类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。







## 6. 一个使用了结构体的示例程序

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

上面的代码中，将一个 `Rectangle` 类型的不可变引用传给了函数 `area`，然后函数计算并返回这个矩形的面积。  
输出如下：

```
The area of the rectangle is 1500 square pixels.
```



## 7. 如何打印结构体信息？



我们先试着直接打印结构体

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

报错信息如下

```
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
 --> src/main.rs:9:29
  |
9 |     println!("rect1 is {}", rect1);
  |                             ^^^^^ `Rectangle` cannot be formatted with the default formatter
  |
  = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
  = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
  = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
```

这里涉及到 trait 概念，在这篇文章中有详细解释。

暂时先不管 trait 具体含义，简单理解下，报错信息中可以看到，编译器提示我们没有给 `Rectangle` 类型实现一个名为 `std::fmt::Display` 的 trait。这里我们只需要知道，只有实现了这个 `std::fmt::Display` 的类型，才可以直接使用宏 `println!` 打印，那些比较简单的类型，Rust 已经帮我们实现好了，但是对于我们自己的结构体，Rust 并不知道怎么做，需要我们自己实现，告诉 Rust 编译器需要打印哪些内容。



我们注意到这里有一句提示，

```
note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```

这句话的意思是，也许（may be）你可以格式化打印你的结构体，使用 `{:?}` 或者 `{:#?}` 而不是我们一直使用的 `{}`。

为什么是也许呢？因为这两种方式是 Rust 中用来调试的，可以给你打印出你结构体对象中的内容，但是这不应该是一个合格的打印内容，只是用来调试的。如果需要正确的打印我们的类型，还是应该去实现 trait `std::fmt::Display`。

要想使用 `{:?}` 或者 `{:#?}` ，首先我们要在结构体上添加注解 `#[derive(Debug)]`。看下面的代码：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
    println!("rect1 is {:#?}", rect1);
}
```

输出如下

```
rect1 is Rectangle { width: 30, height: 50 }
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

可以看到  `{:?}` 和 `{:#?}`  的区别了，前者是打印成了一行，后者格式化了一下。

-

注意，如果你不加注解 `#[derive(Debug)]`，则会编译报错如下

```
error[E0277]: `Rectangle` doesn't implement `Debug`
 --> src/main.rs:9:31
  |
9 |     println!("rect1 is {:?}", rect1);
  |                               ^^^^^ `Rectangle` cannot be formatted using `{:?}`
  |
  = help: the trait `Debug` is not implemented for `Rectangle`
  = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
  = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

error[E0277]: `Rectangle` doesn't implement `Debug`
  --> src/main.rs:10:32
   |
10 |     println!("rect1 is {:#?}", rect1);
   |                                ^^^^^ `Rectangle` cannot be formatted using `{:?}`
   |
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
```

简单的说就是你要先告诉编译器，你现在在调试代码，编译器才会让你打印。



在这篇文章中有写如何实现 std::fmt::Display。



## 8. 结构体中的方法



### 8.1. 定义方法



结构体中除了各个变量，方法也是不可缺少的。下面说下 Rust 中怎么定义方法。

定义方法和定义函数差不多，关键字都是 `fn`，  
区别在于定义方法的时候你要指定是给哪个结构体类型定义的方法。

指定结构体类型的关键字是 `impl`。

下面看个示例代码：

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

输出

```
The area of the rectangle is 1500 square pixels.
```



我们可以看到，Rust 中的结构体类型定义（包含其中成员变量的定义）和它的方法定义是分开的。  
方法定义被语句 `impl Rectangle` 的大括号 `{}` 包起来了，然后就和定义函数差不多了，都是 `fn` 关键字。  
但是注意方法的第一个参数一定是 `self` 或者 `&self`，用来指代当前对象，然后用 `self` 来访问当前对象，就像代码中的 `self.width` 这种。**调用方法的时候不需要传 `self` 或 `&self` 参数。**

这里选择 `&self` 的理由跟在函数版本中使用 `&Rectangle` 是相同的：我们并不想获取所有权，只希望能够读取结构体中的数据，但不修改。如果想要在方法中改变调用方法的实例，需要将第一个参数改为 `&mut self`。

通过仅仅使用 `self` 作为第一个参数来使方法获取实例的所有权是很少见的，这种技术通常用在当方法将 `self` 转换成别的实例的时候使用，这时我们需要防止调用者在转换之后使用原始的实例。



**方法中如果要添加更多参数的话，直接像函数一样写就行了，但是注意保留 `self` 参数，其他都和函数一样。**

另外你可以写很多很多个 `impl`，都是有效的，写在结构体声明前后都行。





### 8.2. 关联函数



`impl` 的另一个常用功能是，允许在 `impl` 块中定义**不**以 `self` 作为参数的函数，这种函数被称为**关联函数(associated functions)**。叫**关联**是因为这些函数与结构体相关联，叫**函数**而不叫方法是因为这种函数的使用不依赖一个已存在的实例。我们之前用过很多次的 `String::from()` 就是一个关联函数。

Rust 中的关联函数与一些编程语言中的静态(Static)成员函数很像（说很像而不是说完全一样，其实是因为我还不清楚具体区别在哪，但其实至少目前我看起来就是一样的 =。=），关联函数属于这个类型，而不属于某个具体的实例。

调用这种关联函数的方法是使用双 `::` 符号（也和其他语言差不多）。

下面看看代码：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let sq = Rectangle::square(20);
    println!("{:?}", sq);
}
```

输出

```
Rectangle { width: 20, height: 20 }
```



注意上面的 `impl` 块中定义的函数没有 `self` 参数哦！

注意我们通过类名调用上面的关联函数 `square`，而没有通过实例化的对象。

注意不可以通过实例来调用关联函数。编译器会提示你这是一个关联函数，而不是方法，要用调用关联函数的方法来调用这个函数。这里就不写代码举例了。