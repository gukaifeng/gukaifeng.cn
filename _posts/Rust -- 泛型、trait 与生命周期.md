---
title: Rust -- 泛型、trait 与生命周期
date: 2021-12-02 18:15:09
updated: 2021-12-08 09:36:20
categories: [编程语言基础]
tags: [Rust]
toc: true
---



简单来说，**泛型**就是一个减少重复的编程技术。泛型概念在大部分编程语言中都有，学习 Rust 的人一般都有其他语言的基础，这里就不详细介绍什么是泛型了。

**trait** 是 Rust 的一个特性，这使你可以以一个通用的方式定义行为。trait 可以与泛型结合，将泛型限制为拥有特定行为的类型，而不是任意类型。

**生命周期(lifetimes)**也是 Rust 中的一个特性，生命周期其实就是一个泛型，这个泛型为编译器提供引用之间如何相互关联的信息。Rust 的生命周期允许我们在许多情况下借用值，同时仍然允许编译器检查引用是否有效。

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

我们知道，Rust 中 `i32`、`char` 这样的类型已经实现好了 Copy 这个 trait，所以可以实现拷贝，而不影响原来变量。关于这里不了解的同学可以看 [Rust -- 所有权(Ownership)](https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/#1-3-%E5%86%85%E5%AD%98%E4%B8%8E%E5%88%86%E9%85%8D)。

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

当然，还有一些其他的改法也可以使程序达到期望中的效果，这里就不多说了。



## 3. 生命周期(lifetime)



### 3.1. 什么是生命周期

我之前在 [Rust -- 所有权(Ownership)](https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/) 这篇文章中有写过关于 Rust 中的所有权、引用与借用的相关概念。不过，还有一个与之相关的重要概念没有说，也就是这节要说的生命周期。

Rust 中每一个引用都有其生命周期，也就是引用保持有效的作用域。

大部分时候生命周期是隐含的，并且是很容易推断出来的，就像大部分时候类型也是可以推断出来的一样。

类型推断中，如果有多种可能性，我们就必须指明具体的类型。同样的，各个引用的生命周期之间，也存在以一些不同的方式相关联的情况，Rust 无法推断其生命周期（即作用域范围），这时候，我们就必须具体指明这些引用的生命周期，以保证运行时实际使用的引用绝对是有效的。

生命周期是 Rust 语言最与众不同的功能。这篇文章不可能涉及生命周期的全部内容，但是会说说那些最常用的功能。



### 3.2. 借用检查器

我们先看一段错误的代码：

```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x;
    }
    println!("r: {}", r);
}
```

注：第 2 行的 `r` 是没有初值的，在给其赋予初值前使用将导致编译错误，Rust 不允许空值。

这段代码中，`x` 在离开作用域后，其使用的内存空间被释放，而此时 `r` 还在其作用域内，成了一个悬垂引用，在第 7 行使用一个悬垂引用，将导致编译错误：

```
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 | 
9 |     println!("r: {}", r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
```

**生命周期的主要目标是避免悬垂引用，因为悬垂引用会导致程序引用了非预期引用的数据。**

那么，Rust 是如何检查 `r` 是一个悬垂引用的呢？更宽泛的说，Rust 是如何检查引用是否有效呢？这得益于**借用检查器(borrow checker)**。

还是那段代码，下面的这个在注释中标记了 `r` 和 `x` 的生命周期，即作用域范围，`r` 的生命周期是 `'a`，`x` 的生命周期是 `'b`。

```rust
fn main() {
    let r;                // ---------+-- 'a
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
    println!("r: {}", r); //          |
}                         // ---------+
```

同样的功能，我们看一段正确的代码，同样使用注释标注 `r` 和 `x` 的生命周期：

```rust
fn main() {
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

`r` 的生命周期是 `'a` ，`x` 的生命周期是 `'b`。

我们可以看到，`'b` 比 `'a` 大，这就意味着 `r` 可以引用 `x`，Rust 知道在 `r` 有效时，`x` 一定是有效的。



### 3.3. 函数中的泛型生命周期

假设我们要实现一个函数，这个函数比较两个字符串哪个更长，并将更长的那个以返回值返回。假定函数名为 `longest()`，我们看下面的代码：

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

那么我们该如何实现 `longest()` 这个函数呢？

首先，我们应该使这个函数接收的参数为字符串 slice 的引用，因为我们不希望这个函数获得参数的所有权。

在上面的调用方法前提下，我们实现一下 `longest()` 方法：

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

看起来没什么问题，但事实上这段代码是无法编译通过的。

我们尝试编译，看看错误信息：

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
1 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
```

错误信息的大意是，这个函数返回了一个借来的值，但是返回值类型信息中没有指名到底是借的 `x` 还是借的 `y`。

事实上，不仅 Rust 不知道，我们自己也不知道，因为我们也不知道函数中是 if 代码块被执行，还是 else 代码块被执行。

至于为什么要纠结这个呢，简单来说，对于一个函数参数而言，`x` 和 `y` 可能具有不同的生命周期，Rust 不知道返回值借的是 `x` 还是 `y`，也就无法准确判断返回值的生命周期，所以编译报错，让我们指定返回值的生命周期。

为了修复这个错误，我们将增加泛型生命周期参数来定义引用间的关系，以便借用检查器可以进行分析。

### 3.4. 生命周期注解语法

**生命周期注解并不改变任何引用的生命周期的长短。**

与在函数声明中指定了泛型类型参数后，就可以接受任何类型参数一样，当指定了泛型生命周期后，函数也能接受任何生命周期的引用。

生命周期注解描述了多个引用的生命周期相互的关系，而不影响他们生命周期。

生命周期注解有着一个不太常见的语法：生命周期参数名称必须以单引号 `'` 开头，其名称通常全是小写，类似于泛型其名称非常短。`'a` 是大多数人默认使用的名称。

生命周期参数注解位于引用的 `&` 之后，并有一个空格来将引用类型与生命周期注解分隔开。

这里有一些例子：我们有一个没有生命周期参数的 `i32` 的引用，一个有叫做 `'a` 的生命周期参数的 `i32` 的引用，和一个生命周期也是 `'a` 的 `i32` 的可变引用：

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

单个的生命周期注解本身没有多少意义，因为生命周期注解告诉 Rust 多个引用的泛型生命周期参数如何相互联系的。

例如如果函数有一个生命周期 `'a` 的 `i32` 的引用的参数 `first`，还有另一个同样是生命周期 `'a` 的 `i32` 的引用的参数 `second`。这两个生命周期注解意味着引用 `first` 和 `second` 必须与这泛型生命周期存在得一样久。





### 3.5. 函数声明中的生命周期注解

我们现在给之前的 `longest()` 函数添加声明周期注解：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

注意，与普通的泛型不同，生命周期泛型不需要在调用时再传一次。例如第 13 行调用 `longest()` 时没有再写 ``<'a>`

现在这个代码就可以正确编译以及正确执行了！

这段代码可能不是很好理解，这是 Rust 中比较难的一个地方，我们这里详细解释一下。

我们看函数声明：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
```

在函数名 longest后面的这个 `<'a>` 中的 `'a` 是一个泛型，就和我们之前常用的泛型 `T` 类似，这里的 `'a` 表示的是生命周期泛型。

然后我们在参数和返回值里，都加上了 `'a`，这样做的含义是，**`x` 引用、`y` 引用和返回值引用具有相同的生命周期！**

**再次注意，生命周期注解，不会改变任何引用的生命周期，这只是告诉 Rust 引用的生命周期之间的关系！**

也就是说，在上面 `longest()` 函数中，`x` 和 `y` 必须同时有效，生命周期 `'a` 可以理解为使得 `x` 和 `y` 同时有效的生命周期，也就是 `x` 和 `y` 生命周期较小的那个，也就是 `x` 和 `y` 生命周期的交集部分！

返回值引用其实就是借走了 `x`（if 代码块）或者借走了 `y`（else 代码块），对返回值引用添加生命周期注解 `‘a`，就保证了这个返回值引用的生命周期是参数 `x` 和 `y` 的生命周期交集部分。

-

无论函数最终执行的是 if 代码块还是 else 代码块，返回值的生命周期都是 `'a`。

假设 `x` 和 `y` 的生命周期不同，`x` 的生命周期大于 `y` 的生命周期，即 `'a` 与 `y` 的生命周期等长。此时，即便执行的是 if 代码块，返回的是 `x` 的借用，当离开 `’a` 的范围时，`x` 可能还是有效的，但是返回值那个引用，已经失效了！

我们修改 main() 函数代码，演示一下上刚刚说的结论：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

这段代码可以正确运行，因为使用 `result` 时，`string1` 和 `string2` 都仍然有效。

现在看一段不能通过编译的代码：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

这段代码无法编译通过，这是因为，我们在使用 `result` 时，`string2` 已经失效了，**即便 `result` 是 `strings1` 的引用！**这就是我们之前 `longest()` 函数限制的返回值生命周期的作用。



### 3.6. 深入理解生命周期



指定生命周期参数的正确方式依赖函数实现的具体功能。

例如，如果将 `longest()` 函数的实现修改为总是返回第一个参数而不是最长的字符串 slice，就不需要为参数 `y` 指定一个生命周期。如下代码将能够编译：

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}

// fn main() { ... }
```

在这个例子中，我们为参数 `x` 和返回值指定了生命周期参数 `'a`，不过没有为参数 `y` 指定，因为 `y` 的生命周期与参数 `x` 和返回值的生命周期没有任何关系。

当从函数返回一个引用，返回值的生命周期参数需要与一个参数的生命周期参数相匹配。如果返回的引用**没有**指向任何一个参数，那么唯一的可能就是它指向一个函数内部创建的值，返回值将会是一个悬垂引用，因为它将会在函数结束时离开作用域。尝试考虑这个并不能编译的 `longest` 函数实现：

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}

// fn main() { ... }
```

即便我们为返回值指定了生命周期参数 `'a`，这个实现却编译失败了，因为返回值的生命周期与参数完全没有关联。这里是会出现的错误信息：

```text
error[E0515]: cannot return value referencing local variable `result`
 --> src/main.rs:3:5
  |
3 |     result.as_str()
  |     ------^^^^^^^^^
  |     |
  |     returns a value referencing data owned by the current function
  |     `result` is borrowed here

For more information about this error, try `rustc --explain E0515`.
```

出现的问题是 `result` 在 `longest()` 函数的结尾将离开作用域，而我们尝试从函数返回一个 `result` 的引用。

我们无法指定生命周期参数来改变悬垂引用，而且 Rust 也不允许我们创建一个悬垂引用。

在这种情况，最好的解决方案是返回一个有所有权的数据类型而不是一个引用，这样函数调用者就需要负责清理这个值了。

综上，生命周期语法是用于将函数的多个参数与其返回值的生命周期进行关联的。一旦他们形成了某种关联，Rust 就有了足够的信息来允许内存安全的操作，并阻止会产生悬垂指针亦或是违反内存安全的行为。

### 3.7. 结构体定义中的生命周期注解

我曾经在 [Rust -- 结构体](https://gukaifeng.cn/posts/rust-jie-gou-ti/) 这篇文章介绍过 Rust 中的结构体。

但是在那篇文章中，结构体中的成员都是具有所有权的，我们这里要定义包含引用的结构体。

定义包含引用的结构体，就需要为结构体中每一个引用添加生命周期注解。

我们看下面的示例：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

我们结合之前在函数声明中的生命周期注解的学习经验，应该不难理解，结构体中的两个 `'a` 注解意味着 `ImportantExcerpt` 的实例不能比其 `part` 字段中的引用存在的更久。

这里的 `main()` 函数创建了一个 `ImportantExcerpt` 的实例，它存放了变量 `novel` 所拥有的 `String` 的第一个句子的引用。`novel` 的数据在 `ImportantExcerpt` 实例创建之前就存在，直到 `ImportantExcerpt` 离开作用域之后 `novel` 都不会离开作用域，所以 `ImportantExcerpt` 实例中的引用是有效的。



### 3.8. 生命周期省略(Lifetime Elision)

我们看一段代码，这段代码曾在 [Rust -- 所有权(Ownership)](https://gukaifeng.cn/posts/rust-suo-you-quan-ownership/) 出现过。

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
```

按我们之前说的，这段代码应当有生命周期注解（因为其参数和返回值都是引用），否则会编译失败。

不过这段代码确实是可以编译通过的！

这段代码能编译通过，是由于一些历史原因。在早期的 Rust 版本中，这个代码确实是不能通过编译的，在那时，这个函数的声明必须是下面这样的：

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

后来，在编写了很多 Rust 代码之后，Rust 团队发现在一些特定的情况下，程序员总是在写一模一样的生命周期注解，这些场景是可预测的，并且遵循几个明确的模式。随后，Rust 团队就把这些模式编码进了 Rust 编译器中。如此一来，在这些情况下，即便不写生命周期注解，Rust 的借用检查器也能推断出生命周期。

回到上面的代码，只有一个参数，只有一个返回值，都是引用，那么这个返回值引用的生命周期，一定是与参数引用的声明周期一样的，没有其他可能。这是一个确定的模式，是可以推断出来的，所以这个函数的声明就可以不写明参数引用和返回值引用的生命周期了。

被编码进 Rust 引用分析的模式，被称为 **生命周期省略规则(lifetime elision rules)**。这不是程序员需要遵循的规则，而是一些特定场景。当代码符合这些特定场景时，Rust 就不要求程序员明确指定相关引用的生命周期。当代码不适用任何一条规则时，Rust 不会推断引用的生命周期，这时就强制要求程序员明确指定。

-

函数或方法的参数的生命周期被称为**输入生命周期(input lifetimes)**，而返回值的生命周期被称为**输出生命周期(output lifetimes)**。

Rust 编译器采用三条规则来判断引用何时不需要明确的注解。第一条规则适用于输入生命周期，后两条规则适用于输出生命周期。如果编译器检查完这三条规则后仍然存在没有计算出生命周期的引用，编译器将会停止并生成错误。这些规则适用于 `fn` 定义，以及 `impl` 块。

1. 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
2. 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
3. 如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明是个对象的方法(method)，那么所有输出生命周期参数被赋予 `self` 的生命周期。



可能上面的描述不是很容易理解，下面以一开始示例的函数声明，来演示这三条规则：

```rust
fn first_word(s: &str) -> &str {
```

此时这是一个没有任何声明周期注解的函数声明。

1. Rust 依据第一条规则（每一个是引用的参数都有它自己的生命周期参数）为这个方法中的每一个参数引用添加其自己的生命周期注解。之后，在 Rust 编译器的视角里，函数声明变成了像下面这样：

    ```rust
    fn first_word<'a>(s: &'a str) -> &str {
    ```

2. 随后，Rust 依据第二条规则（如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数）为函数返回值引用添加生命周期注解，然后如下：

    ```rust
    fn first_word<'a>(s: &'a str) -> &'a str {
    ```

3. 由于我们的函数参数中没有 `self`，所以不适用第三条。



现在，Rust 对最初的函数声明应用了三条规则以后，函数声明最终变成了下面这样：

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

现在，是不是生命周期注解就是很完整的了呢？也就是说，Rust 推断出了参数引用和返回值引用的生命周期，所以我们就不用手动写了。



我们再看看最初写过的 `longest()` 方法，其在没有添加生命周期注解时，函数声明如下：

```rust
fn longest(x: &str, y: &str) -> &str {
```

我们应用第一条规则后，函数声明如下：

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

显然，这就是最终的函数声明了，因为第二和第三条规则都不适用。

显然，Rust 没有推断出 `longest()` 这个函数返回值引用的生命周期，所以需要我们明确指定。

因为第三条规则真正能够适用的就只有方法声明，所以现在就让我们看看这种情况中的生命周期，并看看为什么这条规则使得我们经常不需要在方法声明中中标注生命周期。



### 3.9. 方法定义中的生命周期注解

为带有生命周期的结构体实现方法，语法类似为带有泛型类型的结构体实现方法（毕竟生命周期就是一个特殊的泛型）。

定义方法时，在哪里声明和使用生命周期参数取决于它们是否与结构字段或方法参数和返回值相关。

定义方法时，结构体字段的生命周期必须总是在 `impl` 关键字之后声明，并在结构体名称之后被使用，因为这些生命周期是结构体类型的一部分。

我们下面看一个应用了第三条规则的示例：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

有两个输入生命周期，因此 Rust 应用第一个生命周期省略规则，并赋予 `&self` 和 `announcement` 各自的生命周期。然后，因为其中一个参数是 `&self`，返回类型获得 `&self` 的生存期。到这里，所有引用的生存期都已经计算出来了！



### 3.10. 静态生命周期

这里有一种特殊的生命周期值得讨论：`'static`，其生命周期能够存活于整个程序期间。

例如，所有的字符串字面值都拥有 `'static` 生命周期，我们也可以选择像下面这样显式标注出来：

```rust
let s: &'static str = "I have a static lifetime.";
```

这句代码与下面不写 `'static` 的等价：

```rust
let s: &str = "I have a static lifetime.";
```



这个字符串的文本被直接储存在程序的二进制文件中，而这个文件总是可用的。因此所有的字符串字面值都是 `'static` 的。

下面看一个使用到了静态生命周期的示例代码：

```rust
fn rtn_str(s: &str) -> &str {
    s
}

fn main() {
    let s;
    {
        let s_static: &'static str = "I have a static lifetime.";
        s = rtn_str(s_static);
    }
    
    println!("{}", s);
}
```

代码输出：

```
I have a static lifetime.
```

当上面的代码执行到第 10 行时，`s_static` 变量就没了，但是字符串字面值还在，因为其具有静态生命周期，我们依然可以通过 `s` 来访问这个字符串字面值。



你可能在错误信息的帮助文本中见过使用 `'static` 生命周期的建议，不过将引用指定为 `'static` 之前，思考一下这个引用是否真的在整个程序的生命周期里都有效。你也许要考虑是否希望它存在得这么久。

大部分情况，代码中的问题是由于你尝试创建一个悬垂引用或者可用的生命周期不匹配，你应该解决这些问题而不是指定一个 `'static` 的生命周期。





## 4. 结合泛型类型参数、trait bounds 和生命周期

前面几节说完了泛型类型、trait 和生命周期。

这里，让我们试一下在同一函数中使用泛型类型参数、trait bounds 和生命周期语法！

看下面的代码：

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

这个一个返回两个字符串 slice 中较长者的 `longest()` 函数.

不过带有一个额外的参数 `ann`。`ann` 的类型是泛型 `T`，它可以被放入任何实现了 `where` 从句中指定的 `Display` trait 的类型。这个额外的参数会在函数比较字符串 slice 的长度之前被打印出来，这也就是为什么 `Display` trait bound 是必须的。

因为生命周期也是泛型，所以生命周期参数 `'a` 和泛型类型参数 `T` 都位于函数名后的同一尖括号列表中。
