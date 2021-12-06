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

-

关于如何在函数定义中使用泛型，上面的代码是没有问题的。

但是实际上这段代码无法通过编译，我们看看编译错误信息：

```rust
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
```

错误信息显示，无法在类型 `T` 上使用二元运算符 `>`。

help 信息显示，建议限制类型 `T`。

简单的说，我们在第 5 行进行了一个比较大小的操作，但是 Rust 不知道如何比较 `T` 类型。

Rust 会在编译前确定泛型的具体的类型，然后检查能否进行刚刚的比较操作。检查能否进行比较的依据是类型 `T` 是否实现了 `std::cmp::PartialOrd`，这是一个 trait，我们会在后面说什么是 trait，这里先简单了解。

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

另外，结构体方法中使用的泛型，和结构体定义时使用的泛型，可以不同。

看下面的代码：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

输出：

```
p3.x = 5, p3.y = c
```



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
    Err (E),
}
```

同样地，在结构体声明处，结构体名字的右侧，用 `<>` 包含泛型类型。

同样要注意，使用同一个泛型的成员，类型必须相同。



### 1.4. 泛型代码的性能问题

先说结论：**Rust 中使用泛型类型相比使用具体类型并没有任何速度上的损失。**

Rust 通过在编译时进行泛型代码的**单态化**（*monomorphization*）来保证效率。

单态化是一个通过填充编译时使用的具体类型，将泛型代码转换为特定代码的过程。

简单的说，在编译时，Rust 就会推断出代码中泛型类型的具体类型，然后使用具体类型来编译，所以最后的程序，在性能上和直接使用具体类型不会有任何区别。





## 2. trait

在 Rust 中，trait 用于定义共享的行为。

你可以通过 trait 以一种抽象的方式定义共享行为。

你可以使用 **trait bounds** 指定一个泛型是任何拥有特定行为的类型。

> trait 类似其他语言中的中接口的功能，虽然有些不一样。



### 2.1. 定义 trait

一个类型的行为由其可供调用的方法构成。

如果可以对不同的类型调用相同的方法，这些类型就可以共享相同的行为。

trait 定义是一种将方法与声明组合起来的方法，目的是定义一个实现某些目的所必须的行为的集合。

下面定义一个名为 Summary 的 trait:

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

定义 trait 的关键字是 `trait`。

前面加 `pub` 关键字是为了让其他 crate 可以实现这个 trait。

上面的 trait 的定义中，里面有一个名为 summarize 的方法，这个方法只有声明，没有实现，具体实现将由实现了这个 trait 的类型来完成。

Rust 编译器会保证，所有实现了 Summary 这个 trait 的类型中，都有一个名为 summarize 的可用的方法（有具体实现）。



### 2.2. 为类型实现 trait

上面定义了 Summary trait，这里来演示如何在一个类型上实现此 trait。

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

这段代码中定义了两个 struct，分别是 NewsArticle 和 Tweet。这两个类型都实现了 Summary 这个 triat，并且分别实现了其中的 summarize() 方法。

现在，类型 NewsArticle 和 Tweet 中都有方法 summarize() 了，并且做的事情是不同的。

例如下面的代码：

```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};
```

将输出：

```
1 new tweet: horse_ebooks: of course, as you probably already know, people
```



### 2.3. trait 的默认实现

我们 2.1 中的 Summray 定以，里面的方法没有写实现，其实现将由具体的实现此 trait 的类型来完成。

有时我们需要很多实现了此 trait 的类型做同样的事，这样每个都写一遍就很麻烦，所以我们可以为其添加一个默认实现。

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

现在，Summary 这个 trait 中的 summarize() 方法就有了默认实现。

如果一个类型实现 Summary trait，但没有实现 summarize() 这个方法，那么其将使用上面的默认实现。

当然，这里也可以重载，也就是说，如果一个实现此 trait 的类型实现了方法 summarize()，那么默认方法将失效，调用 summarize() 将执行其自己实现的那个代码。（注意：重载一个方法后，默认方法将无法再调用。）



### 2.4. 调用同一个 trait 中的其他方法

trait 中的一个方法可以调用同一个 trait 中的另一个方法，就像结构体的方法那样。

例如：

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

方法 summarize() 是可以调用方法 summarize_author() 的。

实现此 trait 的类型，只有方法 summarize_author() 是必须实现的，而 summarize() 可以使用默认实现（当然也可以重载）。



### 2.5. 将 trait 作为参数

#### 2.5.1. `impl [trait]` 语法

直接看示例代码，注意看语法格式：

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

函数 notify()，其参数 item 的类型是一个实现了 Summary trait 的类型。

如果给 notify() 传递一个没有实现 Summary trait 的方法，则会编译错误。



#### 2.5.2. Trait Bound 语法

上面的语法更为直观，但是在一些复杂情况会很冗长。

我们看看 Trait Bound 语法，这是一个 Rust 中 `impl [trait]` 语法的语法糖，注意看语法格式：

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

这段代码与上面 2.5.1 中的代码效果完全一样。

这段代码中尖括号 `<>` 类似泛型，这里的 `<T: Summary>` 表示，`T` 是一个实现了 Summary trait 的类型。

假如 notify() 函数有两个这样 的参数，就更能体验出两种语法的差别。

下面先看 2.5.1 中的 `impl [trait]` 语法：

```rust
pub fn notify(item1: impl Summary, item2: impl Summary) {
```

这个声明。item1 和 item2 可以是不同的类型，只要他们都实现了 Summary trait 就可以。

要想限制 item1 和 item2 是相同类型，就只能用 trait bound 语法了：

```rust
pub fn notify<T: Summary>(item1: T, item2: T) {
```



#### 2.5.3. 通过 `+` 符号为参数类型指定多个 trait

假设我们希望参数 item 的类型是同时实现了 Summary 和 Display 两个 trait 的类型，那么两种语法的写法分别如下：

```rust
// impl [trait]
pub fn notify(item: impl Summary + Display) {
```

```rust
// trait bound
pub fn notify<T: Summary + Display>(item: T) {
```



#### 2.5.4. 通过 `where` 关键字简化 trait bound

假设函数参数有多个，每个参数类型要求实现的 tarit 还不一样，那么像上面那样都写在函数声明里就显得太冗长了，就像下面这样：

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

T 是实现了 Display 和 Clone 的类型，U 是实现了 Clone 和 Debug 的类型。

让我们看下使用 `where` 关键字的简化版写法，注意写法格式：

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

我们一开始就像普通泛型一样定义函数，然后在函数的实现之前，也就是 `{}` 之前，像上面的代码那样写，就可以啦！

这样看着清清爽爽！



### 2.6. 将 trait 作为返回值

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

上面的代码使用了 `impl [trait]` 语法，函数的返回值只要是一个实现了 Summary 的类型就可以。

不过下面还要给出一个**无法编译**的代码：

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from("Penguins win the Stanley Cup Championship!"),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from("The Pittsburgh Penguins once again are the best
            hockey team in the NHL."),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        }
    }
}
```

由于 `impl [trait` 工作方式的限制，我们不能在不同的分支返回不同的类型，只能是相同的类型。



### 2.7. 使用 trait bound 有条件地实现方法

我们在 1.2.2 中说过，可以为特定的泛型类型定义方法。

与之类似，我们可以为实现了某些 trait 的特定的泛型类型定义方法。

看下面的代码，注意写法格式：

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

这段代码中，所有 Pair 类型都有 new() 方法，不论 T 是何类型。

不过，只有当 T 是实现了 Display 和 PartialOrd 这两个 trait 的类型是，才有 cmp_display() 这个方法。





### 2.8. 实现 trait 的限制

实现 trait 时需要注意的一个限制是，只有当 trait 或者要实现 trait 的类型位于 crate 的本地作用域时（即 trait 定义或者要实现 trait 的类型的定义，至少有一个要在本地作用域），才能为该类型实现 trait。

例如，可以为 `aggregator` crate 的自定义类型 `Tweet` 实现如标准库中的 `Display` trait，这是因为 `Tweet` 类型位于 `aggregator` crate 本地的作用域中。

类似地，也可以在 `aggregator` crate 中为 `Vec<T>` 实现 `Summary`，这是因为 `Summary` trait 位于 `aggregator` crate 本地作用域中。

但是不能为**外部**类型实现**外部** trait。例如，不能在 `aggregator` crate 中为 `Vec<T>` 实现 `Display` trait。这是因为 `Display` 和 `Vec<T>` 都定义于标准库中，它们并不位于 `aggregator` crate 本地作用域中。

这个限制是被称为 **相干性(coherence)** 的程序属性的一部分，或者更具体的说是 **孤儿规则(orphan rule)**，其得名于不存在父类型。这条规则确保了其他人编写的代码不会破坏你代码，反之亦然。

没有这条规则的话，两个 crate 可以分别对相同类型实现相同的 trait，而 Rust 将无从得知应该使用哪一个实现。





### 2.9. 修复 1.1 代码中的错误

回顾一下 1.1 中的代码：

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

编译错误信息如下：

```rust
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
```

看完了上面介绍的 trait 相关内容，相信大家已经可以看懂这个错误信息了。

简单的说，我们应该显式地限制 T 为实现了 std::cmp::PartialOrd 这个 trait 的类型（即便我们调用时传入的 T 类型为 `i32` 或 `char` 这两个已经实现了 std::cmp::PartialOrd 的类型）。

我们按照错误信息，修改代码如下：

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
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

要说一下，std::cmp::PartialOrd 在 Rust 的 prelude 中，所以直接用就行，不用像错误信息那样写的那么完整。

我们编译一下，还是出错，不过错误信息已经不是上次那个了，说明关于 trait 的问题我们已经解决了（我们调用时传入的 T 类型为 `i32` 或 `char` 这两个是 Rust 已经实现好了 std::cmp::PartialOrd 的类型）。

错误信息如下：

```
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src/main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       move occurs because `list[_]` has type `T`, which does not implement the `Copy` trait
  |                       help: consider borrowing here: `&list[0]`

error[E0507]: cannot move out of a shared reference
 --> src/main.rs:4:18
  |
4 |     for &item in list.iter() {
  |         -----    ^^^^^^^^^^^
  |         ||
  |         |data moved here
  |         |move occurs because `item` has type `T`, which does not implement the `Copy` trait
  |         help: consider removing the `&`: `item`

Some errors have detailed explanations: E0507, E0508.
For more information about an error, try `rustc --explain E0507`.
```

这是一个关于所有权问题的错误。

我们知道，Rust 中在编译时具有已知大小的类型完全存储在栈上，且 `i32`、`char` 这样的类型已经实现好了 Copy 这个 trait，所以可以实现拷贝，而不影响原来变量，也就是创建一个拷贝。关于这里不了解的同学可以看 [Rust -- 所有权(Ownership)](https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/#1-3-%E5%86%85%E5%AD%98%E4%B8%8E%E5%88%86%E9%85%8D)。

但是 Rust 不知道 T 类型是实现了 Copy trait 的，也就有了上面的错误信息，我们现在要做的，就是告诉 Rust，我们传入的参数 T 的类型，是实现了 Copy trait 的。

修改代码如下：

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {  // 添加了 Cpoy trait
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

现在，这个代码就可以正确运行了！

输出内容如下：

```rust
The largest number is 100
The largest char is y
```

-

当然，我们是学习过所有权的，如果不想为 T 指定实现了 Copy，我们也可以不进行拷贝操作，而是直接使用引用，修改代码如下，看注释部分：

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {  // 返回值加了 &
    let mut largest = &list[0];  // list[0] 这里前面加了 &

    for item in list.iter() {  // item 前面删除了 &
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

这样的代码和之前加了 Copy 的输出是一样的，只是执行过程不同了（没有创建拷贝，使用的一直都是引用）。

## 3. 生命周期(lifetime)

