---
title: Rust -- 错误处理
date: 2021-11-30 22:52:13
updated: 2021-12-01 00:42:45
categories: [编程语言基础]
tags: [Rust]
toc: true
---

程序中的错误是不可避免的。

大多编程语言都有异常这个概念，并通过异常机制来应对错误。

Rust 中要求你承认出错的可能性，并在编译代码之前就采取行动。

Rust 中没有异常概念，而是将错误分成了两个主要类别：

1. 可恢复错误 `Result<T, E>`：通常代表向用户报告错误和重试操作是合理的情况，比如未找到文件。
2. 不可恢复错误 `panic!`：通常是 bug 的同义词，比如尝试访问超过数组结尾的位置。



下面先介绍不可恢复错误 `panic!`，然后再说如何返回 `Result<T, E>`。然后还会探讨尝试从错误中恢复还是停止执行时的注意事项。

## 1. `panic!` 宏与不可恢复的错误

下面的内容要求你了解调用栈，如果你不知道什么是调用栈，可能需要先补一下。

Rust 中有一个宏 `panic!`，当这个宏被执行时，程序会打印出一条错误信息，展开、清理栈数据，然后退出。`panic!` 宏被执行的场景通常是程序检测到某些 bug，而且程序员不清楚如何处理（因为这个宏是程序员手动触发执行的）。

当程序出现 panic 时，程序有两种选择，一种是展开，另一种是终止。

展开是程序默认的选择，当程序出现 panic 时，Rust 会回溯栈并清理其遇到的每一个函数的数据，这个回溯并清理的过程要做很多事，也就会花一些时间。

如果你不想程序出现 panic 时展开，可以直接终止程序，这不会清理数据，程序使用中未释放的内存、数据等将交给操作系统处理。

如果你希望项目最终的二进制文件越小越好，可以在 `Cargo.toml` 中的 `[profile.release]` 字段增加 `panic = ‘abort’`，那么当程序以 release 模式编译后，运行时遇到 panic 将直接终止。

```toml
[profile.release]
panic = 'abort'
```



下面我们以一个最简单的方式试一试 `panic!` 宏：

```rust
fn main() {
    panic!("crash and burn");
}
```

`cargo run` 结果如下：

```
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `
` environment variable to display a backtrace
The terminal process "cargo 'run'" failed to launch (exit code: 101).
```

我们可以看到错误信息中有我们在代码中 `panic!` 里指定的文本内容 "crash and burn"，并且指出了在除法 panic 的代码在 `src/main.rs` 第 2 行第 5 个字符（这个索引是从 1 开始的）。



上面信息中还有一个 note，告诉我们在运行程序是设置环境变量 `RUST_BACKTRACE=1` （其实只要等于一个非 0 值都可以∫）可以显示回溯，也就是调用栈信息。

我们上面的示例太简单了，我们换一个稍微复杂点的示例：

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

显然，这个示例程序有数组越界的问题。

`cargo run` 结果如下：

```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
The terminal process "cargo 'run'" terminated with exit code: 101.
```

信息中可以看到我们在 main 中触发了 panic，是数组越界的问题，现在我们再试试设置环境变量，`RUST_BACKTRACE=1 cargo run`，结果如下：

```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/std/src/panicking.rs:517:5
   1: core::panicking::panic_fmt
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:101:14
   2: core::panicking::panic_bounds_check
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:77:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:184:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:15:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/alloc/src/vec/mod.rs:2465:9
   6: hello_rust::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/ops/function.rs:227:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

此信息最后一行有说，使用环境变量 `RUST_BACKTRACE=full` 可以显示更多的（冗长的）调用栈信息，感兴趣的小伙伴可以自己试试，这里就不做演示了，`RUST_BACKTRACE=1` 足矣。

另外 `RUST_BACKTRACE=1` 这种环境变量，主要是在 debug 模式中使用的，在 release 模式下，可能得不到什么有效信息。

虽然我们自己编写的代码中没有调用 `panic!`，但是在上面的调用栈信息中还是可以看到触发了 panic，这也就是说明 panic 不一定发生在我们编写的代码中。

上面的调用栈信息中，第 1 条，也就是栈顶的那一条，是实际真正触发了 `panic!` 的位置，当然这个 `panic!` 不是我们写的。

和大多数语言一样，阅读调用栈信息的关键是从头开始读直到发现你编写的文件，这就是问题的发源地。这一行往上是你的代码所调用的代码；往下则是调用你的代码的代码。这些行可能包含核心 Rust 代码，标准库代码或用到的 crate 代码。上面的调用栈信息中第 6 条，就是我们编写的代码位置，是问题的发源地。



## 2. `Result<T, E>` 与可恢复的错误

当 panic 发生时，程序一定会停止，但并不是所有的错误发生时都需要程序完全停止。

例如，我们要打开一个文件，但是文件不存在，这个错误发生后，也许我们想要的不是终止程序，而是创建这个文件。

对于这个场景，我们就要用到 `Result<T, E>`。

`Result<T, E>` 这是一个枚举类型，其定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 都是泛型参数。T 代表成功时返回的 Ok 成员中的数据的类型，而 `E` 代表失败时返回的 `Err` 成员中的错误的类型。

