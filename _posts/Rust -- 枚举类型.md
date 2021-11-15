---
title: Rust -- 枚举类型
date: 2021-10-04 13:46:32
updated: 2021-10-04 13:46:32
categories: [技术杂谈]
tags: [Rust]
toc: true
---



Rust 中的枚举类型和其他语言的含义大致是一样的，只是定义和使用方法有区别。

这篇文章就不详细介绍什么是枚举类型了。



Rust 中声明枚举类型的关键字为 `enum`。

<!--more-->

## 1. 普通的枚举值

这里的代码以 IP 地址的类型举例，IP 地址目前有 `v4` 和 `v6` 两个版本。

下面看代码

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

这就是一个最简单枚举类型的定义了。

上面的代码定义了一个名为 `IpAddKind` 的枚举类型，其有两个成员，`V4` 和 `V6`。

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

上面的代码说明了如何使用枚举类型。枚举的成员位于其标识符的命名空间中，并使用两个冒号 `::` 分开。

也可以把枚举类型作为函数定义的参数，像下面这样写，和之前的函数定义方法没什么区别。

```rust
fn route(ip_type: IpAddrKind) { }
```

调用函数的时候，记得写全命名空间，像下面这样：

```rust
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```





## 2. 有关联类型的枚举值

上面的枚举类型，里面的 `V4` 和 `V6` 都是没有关联类型的。

大概意思就是，这个枚举类型只能表示其值是 `V4` 或 `V6` 这两个。  
 `V4` 和 `V6` 是我们自己设定的字面，在 Rust 看来并没有什么别的含义。

在 Rust 中，我们可以枚举值关联一个类型，也就是说，除了 `V4` 和 `V6` 这两个字面之外，还可以给它们关联一个类型，就像下面这样，注意看下定义和使用的写法。

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),  // 关联元组类型
    V6(String),  // 关联 String 类型
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

这样，就可以给枚举类型中的值再附加一个类型的值了。

-

枚举中的每个成员可以关联不同的类型，像下面这样：

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

这个枚举有四个含有不同类型的成员：

- `Quit` 没有关联任何数据。
- `Move` 包含一个匿名结构体。
- `Write` 包含单独一个 `String`。
- `ChangeColor` 包含三个 `i32`。





## 3. 枚举中定义方法



枚举和结构体类似，可以定义方法，关键字同为 `impl`，写法也完全一样。

例如下面这样：

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // 在这里定义方法体
        println!("{:#?}", self);
    }
}

fn main() {
    let mut m = Message::Write(String::from("hello"));
    m.call();
    m = Message::Move{ x: 1, y: 2};
    m.call();
}
```

输出如下：

```
Write(
    "hello",
)
Move {
    x: 1,
    y: 2,
}
```

上面的方法除了说明枚举类型定义方法的写法外，你也可以看到变量 `m` 所绑定的值是可以被改变为不同类型的（前提是 `m` 声明为 `mut`）。



## 4. `Option` 枚举

**为了安全，Rust 中没有类似其他编程语言中的空值(null)概念。**

但是空值往往也是有用的，空值可以代表没有值，或者值暂时缺省等等。

Rust 的标准库提供了一个枚举类型 `Option` 可以用来专门表示没有值或者值缺省。除此之外，Rust 中任何地方都不存在空值，换句话说，在 Rust 中，只要你在使用一个非 `Option` 类型的值，它就一定不是空值！

这里只简单介绍一下 `Option`，详细文档看 [Option in std::option - Rust](https://doc.rust-lang.org/std/option/enum.Option.html)。

Option 类型的定义如下。

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

* `None`: 表示没有值；
* `Some(T)`: 表示有类型为 `T` 的值。

注意下这里的 `<T>` 表示泛型，`T` 是类型名，可以指代任何类型。如果你有学习过其他编程语言，对泛型的概念应该会有所了解。

`Option<T>` 枚举很常用，其被包含在了 prelude 之中，你不需要将其显式引入作用域。另外，它的成员也是如此，可以不需要 `Option::` 前缀来直接使用 `Some` 和 `None`。注意，即便如此， `Option<T>` 也仍是常规的枚举，`Some(T)` 和 `None` 仍是 `Option<T>` 的成员。

下面从代码看看其简单的用法：

```rusr
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

前两行我们直接使用 `Some` 时并没有告诉编译器其中的值是什么类型的，和很多类型一样，编译器可以自己推断这里应该是什么类型。

但我们在使用 `None` 的时候，我们就必须显式的告诉编译器这里应该是什么类型，因为编译器无法推断出来。

我们再看下面的代码：

```rust
fn main() {
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
}
```

编译会报错如下：

```
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`

For more information about this error, try `rustc --explain E0277`.
```

大意就是 `i8` 类型无法与 `Option<i8>` 类型相加。

在 Rust 中，如果你想对 `Option<T>`  进行运算，必须先将其转化成 `T`。通常这能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况。因为 Rust 中除了 `Option<T>` 以外的类型不可能为空，编译器会确保这一点！

在 `Option<T>` 中取出 `T`，标准库中提供了多种方法，但是这些方法的具体实现方法不同，有些也可能有危险，所以先不在这里介绍了。

这里就先只介绍上面的内容，更多关于 `Option<T>` 的详细的内容推荐看文档  [Option in std::option - Rust](https://doc.rust-lang.org/std/option/enum.Option.html)。



## 5. 扩展：`IpAddr` 枚举

在这篇文章前面的内容中，使用了 IP 地址的两个版本 `V4` 和 `V6` 作为说明 Rust 中枚举类型用法的例子。

但这仅仅是例子罢了，由于 IP 地址这种实在是太常用了，所以 Rust 的标准库中提供了一个开箱即用的 IP 地址定义 `IpAddr`。

当然，这种有完整详细官方文档的内容，我是不打算写在这里的。

这节扩展就简单说一下有 `IpAddr` 这个东西，具体怎样使用，看文档更好 [IpAddr in std::net - Rust](https://doc.rust-lang.org/std/net/enum.IpAddr.html)。





## 6. 控制流运算符 `match`



  

Rust 中的 `match` 类似 C/C++ 语言中 `switch`，用于匹配多个分支，但是要注意也有区别，Rust 中 `match` 的分支是可以有返回值的。

`match` 会与一些列模式相比较，模式可以由字面值、变量、通配符和许多其他内容构成。

Rust 中 `match` 的力量来源于模式的表现力以及编译器检查，它确保了所有可能的情况都得到处理。



### 6.1. 简单的模式匹配

 下面看例子，注意语法格式：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

上面的代码展示了**一个枚举**和**一个以枚举成员作为模式的 `match` 表达式**。

`match` 表达式中，**每行一个模式，然后一个符号 `=>` 接一个表达式，表达式的结果值将作为整个 `match` 表达式的返回值**。注意这里是表达式（虽然上述代码只是数字字面值，单句话省略 `{}` 这是 Rust 中常见的做法，后面会有示例使用 `{}`）。 每一个分支以逗号隔开，最后一个逗号可写可不写。

当 `match` 表达式执行时，它将结果值按顺序与每一个分支的模式相比较。如果模式匹配了这个值，这个模式相关联的代码将被执行，如果模式并不匹配这个值，将继续执行下一个分支。Rust 中不限制 `match` 中分支的数量。

下面看一下分支代码使用了大括号 `{}` 的示例：

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

上面的代码，其中的模式 `Coin::Penny` 对应的代码块，会输出 "Lucky penny!"，表达式依然返回 `1`。



### 6.2. 有绑定值的模式



匹配分支也可以绑定匹配模式的部分值。这个特征也是从枚举成员中提取值的方式之一。

作为一个例子，我们修改 6.1 例子中 `Coin` 枚举中的一个成员来存放数据。

>1999 年到 2008 年间，美国在 25 美分（即 Quarter）的硬币的一侧为 50 个州的每一个都印刷了不同的设计。其他的硬币都没有这种区分州的设计，所以只有这些 25 美分硬币有特殊的价值。

修改后的 `Coin` 枚举代码如下：

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),  // Coin 枚举中还有个 Quarter 枚举
}
```

使用了 `match` 的代码如下：

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

如果调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`，`coin` 将是 `Coin::Quarter(UsState::Alaska)`。

当将值与每个分支相比较时，没有分支会匹配，直到遇到 `Coin::Quarter(state)`。这时，`state` 绑定的将会是值 `UsState::Alaska`。接着就可以在 `println!` 表达式中使用这个绑定了，像这样就可以获取 `Coin` 枚举中 `Quarter` 成员内部州的值。





### 6.3. 匹配 `Option<T>`

我们在之前的部分中使用 `Option<T>` 时，想要从 `Some` 中取出其内部的 `T` 值。

我们还可以像处理 `Coin` 枚举那样使用 `match` 处理 `Option<T>`！只不过这回比较的不再是硬币，而是 `Option<T>` 的成员，但 `match` 表达式的工作方式保持不变。

比如我们想要编写一个函数，它获取一个 `Option<i32>` ，如果其中含有一个值，将其加一。如果其中没有值，函数应该返回 `None` 值，而不尝试执行任何操作。下面看代码。

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,  // 匹配 None
        Some(i) => Some(i + 1),  // 匹配 Some(T)
    }
}

let five = Some(5);
let six = plus_one(five);  // six = Some(6)
let none = plus_one(None);  // none = None
```









### 6.4. 默认的匹配模式

我们看下下面的代码：

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}


fn main() {

}
```

这段代码会有一个编译错误：

```
error[E0004]: non-exhaustive patterns: `None` not covered
   --> src/main.rs:5:11
    |
5   |     match x {
    |           ^ pattern `None` not covered
    |
    = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
    = note: the matched value is of type `Option<i32>`

For more information about this error, try `rustc --explain E0004`.
```

Rust 编译器告诉我们，`x` 的值还有一种情况 `None` 没有被列出。

**在 Rust 中 `match` 模式匹配的情况（分支）应该是穷尽的，必须列出全部可能的分支。**

但是有些时候，我们只需要特殊处理几个分支，而对剩下的分支（可能很多）做同样的处理，这个场景下，每个分支都单独列出就显得很繁琐而且并没有必要。

Rust 中提供了一个通配符 `_` （这是个下划线）。这个通配符可以代表所有未被列出的情况，下面看代码。

```rust
fn main() {
    let u8_val_3 = 3u8;
    let u8_val_9 = 9u8;
    my_match(&u8_val_3);
    my_match(&u8_val_9);
}

fn my_match(x: &u8) {
    match x {
        1 => println!("one"),
        3 => println!("three"),
        5 => println!("five"),
        7 => println!("seven"),
        _ => (),
    }
}
```

输出

```
three
```

上面的代码中，第一次调用 `my_match(&u8_val_3);` 输出了 "three"；  
第二次调用 `my_match(&u8_val_9);` 没有任何输出，因为没有 `x` 为 `9` 的分支，代码执行到了通配符 `_` 匹配这里，而 **`()` 就是 unit 值，所以 `_` 的情况什么也不会发生**。

**注意：`_` 应当放在最后，因为 Rust 中的 match 是按分支顺序以此向下匹配的，把 `_` 放在不是最后的位置，会导致其后面的分支永远都不会被匹配（虽然并没有编译错误）。**



## 7. 控制流运算符 `if let`

前面说的 `match` 适合有多个分支的场景，如果我们只需要一个分支的话（忽略其他模式），`if let` 更合适些。

下面通过代码来看：

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

请注意上面代码中 `if let` 的使用方法。第一行中 `if let` 后紧接的是模式 `Some(3)`，然后是等号 `=`，最后是待匹配模式的变量 `some_u8_value`。

如果用 `match` 实现一样的逻辑，代码是下面这样的：

```rust
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

我们可以发现，`if let` 相比 `match` 少了一个穷尽性的检查。

也可以在 `if let` 中添加一个 `else`，其意义就等价于上面 `match` 代码中的 `_` 匹配。

我们先用 `match` 举个例子，然后再用 `if let` 重写它。

下面是使用了 `match` 的代码，注意我在这里又补了一遍上了上面的 `Coin` 枚举代码：

```rust
#![allow(unused)]

#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
}

```

上面的代码 `match` 的逻辑是如果这枚硬币是 `Quarter`，就打印一句话，否则给变量 `count` 加一。

下面用 `if let` 重写一段等价于上面 `match` 部分的代码，如下：

```rust
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count +=1;
}
```


> 使用 `match` 还是 `if let` ？
>
> 当我们有很多个分支的时候（分支数大于等于 3），我们只能选择 `match`；  
> 当我们有两个分支，并且两个分支都要匹配某个模式时，还是只能用 `match`；  
> 当我们只需要匹配一个分支时，使用 `if let` 更合适些；  
> 在使用 `if let else` 的场景下（只匹配一个确定的模式，其他所有的都在 `else` 里），我个人更喜欢使用 `match` 配上 `_`，因为代码看起来更美观。当然实际开发中你用哪个都是可以的。























































