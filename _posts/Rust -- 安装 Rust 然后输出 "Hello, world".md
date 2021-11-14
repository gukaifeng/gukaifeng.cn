---
title: Rust -- 安装 Rust 然后输出 "Hello, world"
date: 2021-09-09 22:06:39
categories: [technology]
tags: [Rust]
---



如果你已经阅读过 "[快速配置 Rust 开发环境并编写一个小应用](https://gukaifeng.cn/archives/30)" 这篇文章，可以直接从 2 开始。



### 1. 安装 Rust

官方推荐的做法是通过 `rustup` 来安装 Rust。  
`rustup` 是一个管理 Rust 版本和相关工具的命令行工具。下载时需要联网。

#### 1.1. 在 Linux 上安装 rustup

```
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

这会下载一个脚本，安装 `rustup`，然后会自动安装最新稳定版的 Rust，成功会提示下面的内容

```
Rust is installed now. Great!
```

> 另外，你可能需要一个链接器(Linker)，你大概率已经有了，如果没有的话，自己尝试安装一个 C 编译器，它通常包括一个链接器。C 编译器也很有用，因为一些常见的 Rust 包依赖于 C 代码。

<!--more-->

#### 1.2. 更新或卸载 Rust

更新

```
$ rustup update
```

卸载

```
$ rustup self uninstall
```



#### 1.3. 检查是否正确安装了 Rust

```
$ rustc --version
```

你应该会输出类似下面的信息

```
rustc 1.54.0 (a178d0322 2021-07-26)
```

如果没有的话，接着看看 1.4。

#### 1.4. 关于环境变量

这里说下 `$HOME/.cargo/bin`，也就是 `~/cargo/bin` 这个目录。
在 Rust 开发环境中，所有工具都安装在 `~/.cargo/bin` 目录中，你可以在这里找到包括 `rustc`、`cargo` 和 `rustup` 在内的 Rust 工具链，所以 Rust 开发者通常会将该目录加入 `PATH` 环境变量（**上面的安装程序已经替我们做好了这件事**）。但由于不同平台、命令行 Shell 之间存在差异，`rustup` 中也可能存在 Bug，因此在终端重启或用户重新登录之前，`rustup` 对 `PATH` 的修改可能不会生效，甚至完全无效。



### 2. 输出 "Hello, world"

#### 2.1. 新建一个 Rust 项目

在一个你喜欢的位置，给你的新 Rust 项目一个家~~（创建个文件夹）~~。

然后新建一个 `.rs` 的文件。按照 Rust 的惯例，如果源码文件名有多个单词，用 `_` 隔开，所以我这里新建了一个 `hello_world.rs` 而不是 `helloworld.rs`。

#### 2.2. 编辑 `hello_world.rs`

在 `hello_world.rs` 里写入下面的内容

```rust
fn main() {
    println!("Hello, world!");
}
```



#### 2.3. 编译并运行

在我们源码文件的目录，终端输入

```
rustc hello_world.rs
```

然后在同级目录，会生成一个编译好的二进制文件 `hello_world`，运行它！

```
./hello_world
```

会输出

```
Hello, world!
```



#### 2.4 分析

如果你学过 C/C++，可能会发现 Rust 编写、编译、执行的过程和 C/C++ 基本相同，编译和执行是彼此独立的步骤。

`main()` 函数在 Rust 中有着和 C/C++ 中一样的地位与作用。

注意这里 `println!` 不是函数！这是 Rust 中的一个宏(macro)，在 Rust 中，末尾带 `!` 的是宏，不带的才是函数！

-

另外，仅仅使用 `rustc` 编译简单程序是没问题的，不过随着项目的增长，你可能需要管理你项目的方方面面，并让代码易于分享。在后面的文章里，我会介绍一个叫做 Cargo 的工具，它会帮助你编写真实世界中的 Rust 程序。