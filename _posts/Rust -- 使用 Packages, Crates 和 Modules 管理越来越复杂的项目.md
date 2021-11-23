---
title: Rust -- 使用 Packages, Crates 和 Modules 管理越来越复杂的项目
date: 2021-11-18 22:58:18
updated: 2021-11-24 02:19:35
categories: [编程语言基础]
tags: [Rust]
toc: true
---



Rust 具有许多特性，这些特性允许您管理代码的组织，包括哪些细节是公开的，哪些细节是私有的，以及程序中每个作用域中的名称。这些特性，有时统称为模块系统，包括:



Rust 中的一些特性可以使得你更好的管理、组织代码，比如哪些部分是公开的，哪些细节是私有的，再比如程序中每个作用域的名字。

这些特性，在 Rust 中一般统称为**模块系统(module system)**。

模块系统包含以下几个部分：

* Packages(包): 一个可以让你构建、测试和分享 crates 的 Cargo 特性；
* Crates: 一个可以生成库或可执行文件的模块树；
* Modules(模块) 和 `use`: 允许你控制路径的组织、作用域和私有性；
* Paths(路径) : 一种命名项目的方法。例如结构体，函数或模块。

<!--more-->



## 1. Packages 和 Crates



一个 Crate 是一个二进制文 件或库。

Crate root 是一个源文件，Rust 编译器以 Crate root 为起点，构建你的 crate 的根模块。

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

如果一个包同时有 `src/main.rs` 和 `src/lib.rs`，则这个包有两个 crate，一个是库 crate 和一个 二进制 crate，且名字都与包相同。

你可以把多个文件放在 `src/bin/` 目录下，每个 `src/bin/` 下的文件都会被编译成一个独立的二进制 crate，这样一个包就有了多个二进制 crate。 

一个 crate 会将一个作用域内的相关功能放在一起，这样可以方便地在多个项目中共享这些功能。例如，`rand` crate 提供了生成随机数的功能，通过将 `rand` crate 加入到我们项目的作用域中，我们就可以在自己的项目中使用该功能。`rand` crate 提供的所有功能，都可以通过其 crate 的名字 `rand` 来访问。这里这个概念类似其他语言中的命名空间(namespace)，也可以解决命名冲突的问题，如通过 `::` 访问指定 crate 中的内容。



## 2. 定义 Modules 来控制作用域和私有性





## 3. 在模块树中引用项目的路径





## 4. 使用 `use` 关键字将名称引入作用域





## 5. 将模块划分为不同的文件

