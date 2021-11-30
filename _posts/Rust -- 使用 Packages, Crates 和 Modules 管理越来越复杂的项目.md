---
title: Rust -- 使用 Packages, Crates 和 Modules 管理越来越复杂的项目
date: 2021-11-18 22:58:18
updated: 2021-11-30 09:49:35
categories: [编程语言基础]
tags: [Rust]
toc: true
---



Rust 具有许多特性，这些特性允许你管理代码的组织，包括哪些细节是公开的，哪些细节是私有的，以及程序中每个作用域中的名称。这些特性，有时统称为模块系统，包括:



Rust 中的一些特性可以使得你更好的管理、组织代码，比如哪些部分是公开的，哪些细节是私有的，再比如程序中每个作用域的名字。

这些特性，在 Rust 中一般统称为**模块系统(module system)**。

模块系统包含以下几个部分：

* Packages(包): 一个可以让你构建、测试和分享 crates 的 Cargo 特性；
* Crates: 一个可以生成库或可执行文件的模块树；
* Modules(模块) 和 `use`: 允许你控制路径的组织、作用域和私有性；
* Paths(路径) : 一种命名项目的方法。例如结构体，函数或模块。

<!--more-->

这里这个 Paths(路径) 我觉得有必要解释一下。Rust 中的路径指的不是文件系统的路径，但与之类似，指的是一些项的路径（比如函数、模块、常量等等）。例如 `mod1::mod2::func()` 就是函数 `func()` 的路径，Rust 会按照 `mod1` -> `mod2` -> `func()` 这个顺序找到 `func()` 这个函数。

## 1. Packages(包) 和 Crates



一个 Crate 是一个二进制文 件或库。

Crate root 是一个源文件，一般是 `src/main.rs` 或 `src/lib.rs`。Rust 编译器以 Crate root 为起点，构建你的 crate 的根模块。

一个 package 就是用来提供一系列功能的一个或多个 crates。

一个 package 包含一个 `Cargo.toml` 文件，这个文件描述了如何构建其中的那些 crates。

-

关于 package，Rust 中有一些规定：

1. 一个 package 最多包含一个库 crate；
2. 一个 package 可以包含任意多个二进制文件 crate；
3. 一个 package 至少包含一个 crate（库或二进制文件 crate 都可以）。

-

当我们用 `cargo new` 命令创建一个新的 rust 项目时，我们看看提示信息是什么。

```shell
cargo new hello-rust
```

提示信息如下：

```
Created binary (application) `hello-rust` package
```

可以从提示信息看到，我们新建的一个 Rust 项目，就是一个 package。	

我们查看 `Cargo.toml` 文件：

```toml
[package]
name = "hello-rust"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

我们知道 `Cargo.toml` 中记录了包之间的依赖关系，我们的项目 `hello-rust` 本身就是个包(Package)。

`Cargo.toml` 中没有提到 `src/main.rs`，因为 Cargo 遵循一个约定：

* 如果包目录中包含 `src/main.rs`，则这个包中包含一个与包同名（即与项目同名）的**二进制 crate**，`src/main.rs` 就是这个包的 crate root，Rust 编译器以此为起点；
* 如果包目录中包含 `src/lib.rs`，则这个包中包含一个与包同名（即与项目同名）**库 crate**，`src/lib.rs` 就是这个包的 crate root，Rust 编译器以此为起点。



Cargo 会把 crate root 传递给 rustc 用以实际构建库或二进制项目。

我们上面的示例中的包，只包含一个 `src/main.rs`，也就说说这个包只含有一个名为 `hello-rust` 的二进制 crate。

如果一个包同时有 `src/main.rs` 和 `src/lib.rs`，则这个包有两个 crate，一个是库 crate 和一个 二进制 crate，且名字都与包相同。

你可以把多个文件放在 `src/bin/` 目录下，每个 `src/bin/` 下的文件都会被编译成一个独立的二进制 crate，这样一个包就有了多个二进制 crate。 

一个 crate 会将一个作用域内的相关功能放在一起，这样可以方便地在多个项目中共享这些功能。例如，`rand` crate 提供了生成随机数的功能，通过将 `rand` crate 加入到我们项目的作用域中，我们就可以在自己的项目中使用该功能。`rand` crate 提供的所有功能，都可以通过其 crate 的名字 `rand` 来访问。这里这个概念类似其他语言中的命名空间(namespace)，也可以解决命名冲突的问题，如通过 `::` 访问指定 crate 中的内容。





## 2. Modules(模块) 和 Paths(路径)

Rust 中的模块和 C++ 中的 namespace 概念有些像，都是把一些内容集中放在一起。

Rust 中的路径的使用，也和 C++ 的 namespace 的使用方法差不多，例如 `use std::thread::Thread;`。

当然，在 Rust 和 C++ 中，上面说的这些概念的区别还是很大的，这里提到 C++ 的 namespace 只是为了便于理解，区别看这里 [Rust "use" vs. C++ "using namespace"](https://stackoverflow.com/questions/29013617/rust-use-vs-c-using-namespace)，这个不是本文重点，这里就不多说了。

-

通常来说，我们编写模块的场景，一般不是产生二进制文件，而是库文件。也就是说，我们需要创建一个库 crate，而不是二进制文件 crate。但是我们之前使用 `cargo new` 命令创建的都是二进制文件 crate，下面我们加个参数 `--lib` 来创建库 crate。

```shell
cargo new --lib restaurant
```

和之前的不加 `--lib` 的时候不同，这次生成了 `src/lib.rs` 而不是 `src/main.rs`。

下面，我们开始介绍关于 Rust 中模块和路径的具体内容。

### 2.1. 定义模块

下面的示例来自 [《The Rust Programming Language》](https://doc.rust-lang.org/book/#the-rust-programming-language)。

在 `src/lib.rs` 写入下面的示例代码：

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }
    mod serving {
        fn take_order() {}
        fn server_order() {}
        fn take_payment() {}
    }
}
```

模块的关键字是 `mod`。

上面的代码还是比较简单的，就是一个模式里有两个子模式，两个子模式里又分别定义了几个函数。

模块里面还可以定义一些其他项，比如结构体、枚举、常量、trait 或者函数。

我们之前说过，`src/main.rs` 和 `src/lib.rs` 叫做 crate 根(crate root)。之所以这样称呼，是因为这两个文件的内容分别在 crate 模块结构的根组成了一个名为 `crate` 的模块，这个结构叫做**模块树(module tree)**。

下面是上面示例代码的模块树结构：

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

上面的模块树结构中，模块之间是存在一些关系的。例如：

* 模块 `hosting` 和 `serving` 是模块 `front_of_house` 的**子(child)**模块；
* 模块 `front_of_house` 是模块 `hosting` 和 `serving` **父(parent)**模块；
* 模块 `hosting` 和 `serving` 是**兄弟(siblings)**模块；
* 模块 `front_of_house` 是根模块 `crate` 的子模块。

这里要说一下根模块 `crate`（即 crate root），这个模块是隐式的，`src/main.rs` 或 `src/lib.rs` 中的内容构成了这个模块的内容。

这个模块树可能会令你想起电脑上文件系统的目录树，这是一个非常恰当的比喻。就像文件系统的目录，你可以使用模块来组织你的代码。并且，就像目录中的文件，我们需要一种方法来找到模块，也就是路径。





### 2.2. 通过路径引用模块



#### 2.2.1. 引用路径方法



我们上面定义了几个模块，现在来说如何在代码中引用这些模块（与模块中的项）。

引用模块要通过路径，类似文件系统的路径，这里的路径也可以分为**绝对路径(absolute path)**和**相对路径(relative path)**。

* 绝对路径：从 crate 名字或根模块 `crate` 开始。之前说过，crate root 其实就是源文件 `src/main.rs` 或 `src/lib.rs`，其中的内容构成了根模块 `crate`。
* 相对路径：从当前模块开始，以 `self` 或 `super` 或当前模块的标识符开头。

路径之间由双冒号 `::` 分隔。

下面看一段示例代码：

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径，两者等价
    front_of_house::hosting::add_to_waitlist();
}
```

上面的示例中绝对路径以 `crate` 开始，这是因为模块 `front_of_house` 和函数 `eat_at_restaurant` 在同一个 crate 中，所以绝对路径可以从根模块 `crate` 开始。如果不在一个 crate 中，那绝对路径应该从 crate 名字开始，例如 `std::collections::LinkedList` 就是一个以标准库 crate 名 `std` 开头的绝对路径。

相对路径以 `front_of_house` 开始，这是因为模块 `front_of_house` 和函数 `eat_at_restaurant` 在同一个层级中。

选择使用绝对路径还是相对路径，要根据具体项目决定，取决于你是更倾向于将项的定义代码与使用该项的代码分开来移动，还是一起移动。

举一个例子，如果我们要将 `front_of_house` 模块和 `eat_at_restaurant` 函数一起移动到一个名为 `customer_experience` 的模块中，我们需要更新 `add_to_waitlist` 的绝对路径，但是相对路径还是可用的。

然而，如果我们要将 `eat_at_restaurant` 函数单独移到一个名为 `dining` 的模块中，还是可以使用原本的绝对路径来调用 `add_to_waitlist`，但是相对路径必须要更新。

我们更倾向于使用绝对路径，因为把代码定义和项调用各自独立地移动是更常见的。

#### 2.2.2. 使用 `pub` 关键字暴露路径

我们尝试编译一下上面的代码，会无法通过编译，错误信息如下：

```
   Compiling restaurant v0.1.0 (/home/gukaifeng/rust/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

简单的说，就是模块 `hosting` 是私有的，无法在函数 `eat_at_restaurant()` 中访问。

Rust 中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。

父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用他们父模块中的项。这是因为子模块封装并隐藏了他们的实现详情，但是子模块可以看到他们定义的上下文（这也意味着同级的项互相都是暴露的）。

在 Rust 中，如果你希望子模块的内部部分暴露给父级模块，需要使用 `pub` 关键字。

我们给模块 `hosting` 加上 `pub` 关键字后再试一次：

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

依然报错，报错信息如下：

```
   Compiling restaurant v0.1.0 (/home/gukaifeng/rust/restaurant)
error[E0603]: function `add_to_waitlist` is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^ private function
  |
note: the function `add_to_waitlist` is defined here
 --> src/lib.rs:3:9
  |
3 |         fn add_to_waitlist() {}
  |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
3  |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

简单的说，就是函数 `add_to_waitlist()` 是私有的，无法在函数 `eat_at_restaurant()` 中访问。

这是因为，Rust 中，有 `pub` 关键字的项只向父级暴露其内容，父级也无法查看没有 `pub` 关键字项的内容。`hosting` 模块虽然向父级模块 `front_of_house` 暴露了，但是由于 `hosting` 没有查看函数 `add_to_waitlist()` 的权限，所以父级 `front_of_house` 模块也没有这个权限。

要想上面的代码编译通过，我们需要给函数 `add_to_waitlist()` 也添加 `pub` 关键字。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

现在，就可以通过编译了。

> 我们之前一直没说为什么在函数 `eat_at_restaurant()` 的定义前也加了 `pub` 关键字。不过我们说完上面的内容以后，你应该已经明白了，这个是为了使外部的代码可以调用这个函数的。



#### 2.2.3. 以 `self` 或 `super` 为起点的相对路径

我们先看以 `self` 的相对路径，以 `self` 为起点，意思就是以当前位置为起点，这个写不写其实区别不大，我们修改下上面的代码：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径，两者等价
    front_of_house::hosting::add_to_waitlist();
    self::front_of_house::hosting::add_to_waitlist();
}
```



我们再看以 `super` 为起点的相对路径，即以父级为起点：

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

`serve_order()` 与函数 `fix_incorrect_order()` 的父级模块 `back_of_house` 同级，所以要想在 `fix_incorrect_order()` 中调用 `serve_order()`，就要使用 `super::serve_order();`。

#### 2.2.4. 定义公有的结构体和枚举

和上面定义函数 `pub fn eat_at_restaurant() {...}` 的方法类似，我们可以通过 `pub` 关键字定义结构体和枚举等等。

但是这里也有一些特殊的地方：

* 结构体：如果我们在一个结构体前加上 `pub` 关键字，那么这个结构体就会变成公有的，但是其中的字段，还是私有的，要想某个字段也是公有的，也需要给其加上 `pub` 关键字。
* 枚举类型：与结构体相反，如果一个枚举前加了 `pub` 关键字，那么这个枚举的所有成员都是公有的。



### 2.3. `use`, `as` 与 `pub use`



#### 2.3.1. 使用 `use` 关键字将名称引入作用域

对于路径级别比较多的项，每次都写那么长一段路径是很麻烦的，也会使代码变得冗长、重复。

我们现在使用 use 关键字，简化一下之前的代码：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;  // 使用绝对路径 use
// use front_of_house::hosting;  // 与上面等价，使用相对路径 use
// use self::front_of_house::hosting;  // 与上面等价，使用以 "self" 开头的相对路径

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

上面的几个 `use` 语句是等价的。

和之前一样，通过 `use` 引用的路径也会检查私有性。

我们上面的 `use` 引用语句引入的路径是到 `hosting` 的，这是我们调用函数 `add_to_waitlist()` 的上一级，引用到父级模块是常用做法。

当然你也可以直接引用到 `add_to_waitlist()`，例如：

```rust
use crate::front_of_house::hosting::add_to_waitlist();
```

这样在调用的时候就不用加前缀 `hosting::` 了，不过这不是惯用做法。

引用到父级模块是惯用做法，例如 `use std::collections::HashMap;`。这样可以清晰的看出某个项是外部引入的，而不是在本地定义的，同时也避免了不同父级模块下可能存在的重名项的问题。这个做法没有硬性要求，就是惯例而已。



#### 2.3.2. 使用 `as` 关键字给 `use` 引入的项起别名

看一段代码：

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
```

上面的代码中，`std::fmt` 和 `std::io` 下都有一个名为 `Result` 的。

除了可以引入到父级模块 `std::fmt` 和 `std::io` 然后通过 `fmt::Result` 和 `io::Result` 来区分以外，还可以像上面代码那样通过 `as` 关键字起别名来区分。

当然，你也可以给引入的任意内容起个别名，你开心就好！



#### 2.3.3. 使用 `pub use` 重导出(re-exporting)名称

当我们使用 `use` 引入名称以后，例如 `use crate::front_of_house::hosting;`，我们就可以在我们的作用域中直接使用 `hosting` 。

但是，这个 `hosting` 只有在我们的作用域中是可以直接用的，是私有的。

###### 如果你想让调用我们编写的代码的代码可以和我们一样直接使用 `hosting`，则要结合 `pub` 和 `use` 关键字，这个技术叫做**重导出(re-exporting)**。

例如：

```rust
pub use crate::front_of_house::hosting;
```

这样调用我们代码的代码，也可以直接使用 `hosting` 了。

这样做可以使我们的库很好地组织起来，以便程序员使用库和调用库。





#### 2.3.4. 使用外部包

假设我们现在的项目要使用一个外部包 `rand` 来生成随机数。

为了使用这个外部包，我们在 `Cargo.toml` 中的 `dependencies` 字段加入如下行：

```toml
[dependencies]
rand = "0.5.5"
```

`=` 左边是包名，右边是要使用的版本号。

在 Cargo.toml 中加入 `rand` 依赖告诉了 Cargo 要从 [crates.io](https://crates.io/) 下载 `rand` 包和其依赖，并使其可在项目代码中使用。

接着，为了将 `rand` 定义引入项目包的作用域，我们加入一行 `use`，它以 `rand` 包名开头并列出了需要引入作用域的项。

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1, 101);
}
```

为了测试我们的代码，我们这里使用的应该是一个二进制文件 crate，在 `src/main.rs` 中加入上面的代码。

当你尝试编译上面的代码时会发现，如果 `rand` 还没有下载到本地，那么编译器会先将其下载后，再进行编译。

注意标准库（`std`）对于你的包来说也是外部 crate。因为标准库随 Rust 语言一同分发，你无需修改 *Cargo.toml* 来引入 `std`，不过你仍让需要通过 `use` 将标准库中定义的项引入项目包的作用域中来引用它们，比如我们使用的 `HashMap`：

```rust
use std::collections::HashMap;
```

这是一个以标准库 crate 名 `std` 开头的绝对路径。





#### 2.3.5. 使用嵌套路径来消除大量的 use 行

当我们要引入同一个块的多个项时，每一项都写一个 `use` 行会占用很多空间。

像这样：

```rust
use std::collections::HashMap;
use std::collections::HashSet;
use std::collections::LinkedList;
```

Rust 中提供了更简单的写法，像下面这样：

```rust
use std::collections::{HashMap, HashSet, LinkedList};
```

再举几个例子，你就可以明白怎么用了：

```rust
// 使用两行 use
use std::cmp::Ordering;
use std::io;

// 嵌套整合 use
use std::{cmp::Ordering, io};
```

```rust
// 通过两行 use 语句引入两个路径，其中一个是另一个的子路径
use std::io;
use std::io::Write;

// 是用嵌套路径，由于第一个就引入到 std::io，所以在大括号使用 self 表示其本身
use std::io::{self, Write};
```







#### 2.3.6. 使用 glob(`*`) 运算符将所有的公有项引入作用域

如果希望将一个路径下所有公有项引入作用域，可以指定路径后跟 `*`，glob 运算符：

```rust
use std::collections::*;
```

这个 `use` 语句将 `std::collections` 中定义的所有公有项引入当前作用域。

使用 glob 运算符时请多加小心，glob 会使得我们难以推导作用域中有什么名称和它们是在何处定义的。



### 2.4. 将模块划分为不同的文件

到目前为止，本文所有的例子都是在一个文件（这里都是 `src/lib.rs`）中定义多个模块。

当模块变得更大时，你可能想要将它们的定义移动到单独的文件中，从而使代码更容易阅读。

但是不管如何移动，库 crate 的根（crate root）仍然是 `src/lib.rs`。

现在，我们将 `front_of_house` 模块移动到属于它自己的文 `src/front_of_house.rs` 中：

```rust
// src/front_of_house.rs

pub mod hosting {
    pub fn add_to_waitlist() {}
}
```



然后改变 crate 根文件（`src/lib.rs`），如下：

```rust
// src/lib.rs

mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

`src/lib.rs` 中，在 `mod front_of_house` 后使用分号，而不是代码块，这将告诉 Rust 在另一个**与模块同名的文件**中加载模块的内容。

继续重构我们例子，将模块 `front_of_house` 中 的 `hosting` 模块也提取到其自己的文件中，仅对 `src/front_of_house.rs` 包含 `hosting` 模块的声明进行修改：

```rust
// src/front_of_house.rs

pub mod hosting;
```

然后编辑 `src/front_of_house/hosting.rs`，内容如下：

```rust
// src/front_of_house/hosting.rs

pub fn add_to_waitlist() {}
```



注意看路径哦！`hosting` 模块是 `front_of_house` 模块的子模块，所以其同名 rs 文件 `hosting.rs` 在 `src/front_of_house/` 下面。

一定要注意，模块划分进文件，文件名一定要与模块名字相同（包括路径相同）！



在上面例子中，crate 根文件是 `src/lib.rs`。不过模块划分到文件同样适用于以 `src/main.rs` 为 crate 根文件的二进制 crate 项。

