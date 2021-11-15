---
title: Rust -- Cargo - Rust 的构建系统和包管理器
date: 2021-09-09 22:56:34
updated: 2021-09-09 22:56:34
categories: [编程语言基础]
tags: [Rust]
toc: true
---

如果你已经阅读过 "[快速配置 Rust 开发环境并编写一个小应用](https://gukaifeng.cn/archives/30)" 这篇文章，关于 Cargo 的内容可能会有少量重复。



-----

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean 们使用 Cargo 来管理他们的 Rust 项目，因为 Cargo 可以帮你做很多事，比如构建代码、下载并编译依赖库。

最简单的 Rust 程序，比如 "[安装 Rust 然后输出 "Hello, world!" ](https://gukaifeng.cn/archives/31)" 这篇文章编写的输出 "Hello, World!"，没有任何依赖。所以如果使用 Cargo 来构建 “Hello, world!” 项目，就只会用到 Cargo 构建代码的那部分功能。

但在编写更复杂的 Rust 程序时，你就会添加一些依赖项，使用 Cargo 可以让这个事情变得简单。

一般从 `rustup` 安装的 Rust 自带了 Cargo。  
如果通过其他方式安装的话，可以在终端输入如下命令检查是否安装了 Cargo：

```
cargo --version
```

如果你看到了版本号，说明已安装！  
如果看到类似 `command not found` 的错误，你应该查看相应安装文档以确定如何单独安装 Cargo。

<!--more-->

### 1. 使用 Cargo 创建项目

```
cargo new hello_cargo
cd hello_cargo
```

上面的命令使用 Cargo 创建了一个名为 ”hello_cargo“ 的项目，这个过程会在同级目录创建一个名字同为 "hello_cargo" 的文件夹，进入这个文件夹。

目录结构如下

```
hello-cargo
|- Cargo.toml
|- src
  |- main.rs
```

终端输入

```
ls -a
```

输出

```
.  ..  Cargo.toml  .git  .gitignore  src
```

可以发现，Cargo 在新建项目的同时，还在这里初始化了一个 git 仓库。

如果在一个现有的 git 仓库执行 `cargo new`，就不会生成这些 git 文件，或者你可以使用 `cargo new --vcs=git`，这会强行初始化一个 git 仓库，会覆盖掉目录下原有的 git 信息。

另外，如果你不想为新项目新建一个目录，而是就用当前目录，那直接在这个目录下执行 `Cargo init`。





### 2. 关于 `Cargo.toml` 与 `src/main.rs`

我们看看 Cargo.toml 这个文件里有些什么

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

这个文件使用 [*TOML*](https://toml.io/) (*Tom's Obvious, Minimal Language*) 格式，这是 Cargo 配置文件的格式。

第一行，`[package]`，是一个片段(section)标题，表明下面的语句用来配置一个包。  
随着我们在这个文件增加更多的信息，还将增加其他片段(section)。

接下来的三行设置了 Cargo 编译程序所需的配置：

* `name`: 项目的名称；
* `version`: 项目的版本；
* `edition`: 要使用的 Rust 版本。[附录 E](https://kaisery.github.io/trpl-zh-cn/appendix-05-editions.html) 会介绍 `edition` 的值。

在本文编写时，Rust 有两个版本：Rust 2015 和 Rust 2018。本书基于 Rust 2018 edition 编写。

Cargo.toml 中的 `edition` 字段表明代码应该使用哪个版本编译。如果该字段不存在，其默认为 `2015` 以提供后向兼容性。

最后一行，`[dependencies]`，是罗列项目依赖的片段的开始。在 Rust 中，代码包被称为 `crates`。现在这个项目并不需要其他的 `crate`，所以这里啥也没有，不过在以后的项目中会用得上这个片段。



-

我们看看 `src/main.rs` 这个 Rust 源码文件里有些什么

```rust
fn main() {
    println!("Hello, world!");
}
```

这就是一个最简单的 Rust 代码，输出 "Hello, world!"。注意 `println!` 是宏，不是函数。

-

Cargo 项目与之前 rustc 项目的区别是 Cargo 将代码放在 src 目录下，同时项目根目录包含一个 Cargo.toml 配置文件。Cargo 期望源文件存放在 src 目录中。项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件。使用 Cargo 可以帮助你保持项目干净整洁。

另外，如果你有一个没有使用 Cargo 开始的项目，现在想用 Cargo，可以把代码放入 src 目录，并手动创建一个合适的 Cargo.html 文件。





### 3. 构建并运行 Cargo 项目

这里给出几个 Cargo 构建、运行项目的几个常用操作

* `cargo build`: 编译项目；
* `cargo run`: 编译并运行项目（注意哦，是编译 + 运行都有）；
* `cargo check`: 检查项目是否可以编译通过，省略了生成可执行文件的步骤（好东西）。

**如果没有改变项目，但重复执行 `cargo build`，cargo 什么都不会做；**  
**如果没有改变项目，但重复执行 `cargo run`**，cargo 不会重复编译，只会直接执行之前编译好的二进制文件。

-

我们先试试 `cargo build`

输出如下

```
$ cargo build
   Compiling hello_cargo v0.1.0 (/home/gukaifeng/rust/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.77s
```

可以从输出信息中看到，  
编译了项目 "hello_cargo"，项目版本是 v0.1.0，项目目录是 /home/gukaifeng/rust/hello_cargo 。  
编译结束，用时 0.77s，有两个标签 `unoptimized` 表示未做编译优化，`debuginfo` 表示启用了调试信息。  
这说明我们编译的项目默认是 debug 版的。

此时如果再重复执行 `cargo build`，只会输出类似第二行的内容而没有编译过程了。

现在知道我们编译出来的是 debug 版了，Cargo 会把编译好的二进制文件放在 `target/debug/` 目录下，  
运行我们编译好的可执行文件

```
./target/debug/hello_cargo
```

输出

```
Hello, world!
```



### 4. Release 构建

上面说了 Cargo 默认是以 Debug 方式编译的，方便调试。

当项目准备好发布了，就要使用 Release 方式构建，并且编译好的文件会放在 `target/release` 目录下。

使用 Release 构建方式只需要在 `cargo build` 命令后添加参数 `--release`，即 `cargo build --release`。

关于 Debug 和 Release 编译方式的区别这里就不作介绍了~

 