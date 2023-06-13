
程序中的错误是不可避免的。

大多编程语言都有异常这个概念，并通过异常机制来应对错误。

Rust 中要求你承认出错的可能性，并在编译代码之前就采取行动。

Rust 中没有异常概念，而是将错误分成了两个主要类别：

1. 可恢复错误 `Result<T, E>`：通常代表向用户报告错误和重试操作是合理的情况，比如未找到文件。
2. 不可恢复错误 `panic!`：通常是 bug 的同义词，比如尝试访问超过数组结尾的位置。



下面先介绍不可恢复错误 `panic!`，然后再说如何返回 `Result<T, E>`。然后还会探讨尝试从错误中恢复还是停止执行时的注意事项。



<!--more-->

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



上面信息中还有一个 note，告诉我们在运行程序是设置环境变量 `RUST_BACKTRACE=1` （其实只要等于一个非 0 值都可以∫）可以显示回溯，也就是调用栈信息。

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

另外 `RUST_BACKTRACE=1` 这种环境变量，主要是在 debug 模式中使用的，在 release 模式下，可能得不到什么有效信息。

虽然我们自己编写的代码中没有调用 `panic!`，但是在上面的调用栈信息中还是可以看到触发了 panic，这也就是说明 panic 不一定发生在我们编写的代码中。

上面的调用栈信息中，第 1 条，也就是栈顶的那一条，是实际真正触发了 `panic!` 的位置，当然这个 `panic!` 不是我们写的。

和大多数语言一样，阅读调用栈信息的关键是从头开始读直到发现你编写的文件，这就是问题的发源地。这一行往上是你的代码所调用的代码；往下则是调用你的代码的代码。这些行可能包含核心 Rust 代码，标准库代码或用到的 crate 代码。上面的调用栈信息中第 6 条，就是我们编写的代码位置，是问题的发源地。



## 2. `Result<T, E>` 与可恢复的错误

当 panic 发生时，程序一定会停止，但并不是所有的错误发生时都需要程序完全停止。

例如，我们要打开一个文件，但是文件不存在，这个错误发生后，也许我们想要的不是终止程序，而是创建这个文件。

对于这个场景，我们就要用到 `Result<T, E>`。



### 2.1. 什么是 `Result<T, E>`

`Result<T, E>` 这是一个枚举类型，其定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 都是泛型参数。T 代表成功时返回的 Ok 成员中的数据的类型，而 `E` 代表失败时返回的 `Err` 成员中的错误的类型。

`Result<T, E>` 通常作为一个可能出错的方法的返回值类型。如果方法执行成功了，会把成功的结果放在 `T` 中，返回 `Ok(T)`，如果失败了，会把错误信息放在 `E` 中，返回 `Err(E)`。

这里已知标准库中的打开文件函数 `File::open()` 返回值类型是 `Result<T, E>`。  
这个函数如果执行成功了，`T` 的类型就是 `std::fs::File`，这是一个文件句柄；  
如果失败了，`E` 的类型是 `std::io::Error`。

然后看一段示例代码，这段代码使用 `match` 表达式处理 `Result<T, E>`：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

要注意的是，与 `Option` 枚举一样，`Result` 枚举和其成员也被导入到了 prelude 中，所以就不需要在 `match` 分支中的 `Ok` 和 `Err` 之前指定 `Result::`。

上面的代码中可以看到，如果函数的返回结果是 `OK` 时，把 `OK` 中的句柄赋值给 f；如果函数的返回结果是 `Err`，就执行 `panic!`。

由于我们现在没有 "hello.txt" 文件，所以执行上面的代码，理所应当的报错如下：

```
thread 'main' panicked at 'Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:9:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
The terminal process "cargo 'run'" terminated with exit code: 101.
```

错误信息中明确指出文件没找到。



### 2.2. 匹配不同的错误



#### 2.2.1. 使用 `match` 为不同的结果执行不同的操作

上面打开文件的代码中，只要打开文件失败就会执行 `panic!`。

但是我们并不是什么遇到什么错误都想执行 `panic!`。

就以上面打开文件这个代码来说，我们可能希望，如果错误是文件不存在，那我们就创建一个文件，如果是其他错误或者创建文件仍然失败，再执行 `panic!`。

我们在代码中再增加一个 `match` 来实现这个效果：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

现在这段代码中，就可以实现我们想要的效果了：

* 如果文件存在且正确打开了，就返回文件的句柄；
* 如果是除了文件不存在的其他错误，执行 `panic!`；
* 如果是文件不存在的错误，就新建一个文件。
    * 如果新建文件成功了，返回新建文件的句柄；
    * 如果新建文件遇到错误，执行 `panic!`。



#### 2.2.2. 使用闭包简化代码

不过我们使用 `match` 分支实现上述的效果是很繁琐的，如果你了解 Rust 中的闭包，则下面这个不使用 `match` 的写法会更简单更清晰：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

如果你不了解 Rust 中的闭包，可以看 [Rust -- 函数式语言功能]() 中的闭包部分，或查阅相关文档。

不过使用闭包简化这段代码并不是最好的选择，这里只是提供一个思路，也许以后用得到。

要简化上述代码，下面一节的方法更实用一些。

#### 2.2.3. 失败时 panic 的简写: `unwarp()` 和 `expect()`

`Result<T, E>` 类型定义了很多辅助方法来处理各种情况，`unwrap()` 和 `expect()` 便是其中之二。

`unwrap()` 和 `expect()` 用起来就和使用了 `match` 的类似。

* 如果 `Result<T, E>` 值是 `Ok`，那么 `unwarp()` 和 `expect()` 就会返回 `T`；
* 如果 `Result<T, E>` 值是 `Err`，那么 `unwarp()` 和 `expect()` 就会执行 `panic!`。

`unwrap()` 和 `expect()` 的区别是，`expect()` 可以自己指定 `panic!` 中的信息，而 `unwrap()` 不可以。

下面看一段 `unwrap()` 的示例代码：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

假定要打开的文件不存在，运行会输出错误信息如下：

```
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
The terminal process "cargo 'run'" terminated with exit code: 101.
```

下面看一段 `expect()` 的示例代码：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

假定要打开的文件不存在，运行会输出错误信息如下：

```
thread 'main' panicked at 'Failed to open hello.txt: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:4:37
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
The terminal process "cargo 'run'" terminated with exit code: 101.
```

看出区别了没？

使用了 `expect()` 的错误信息以我们指定的文本 "Failed to open hello.txt" 开始，我们将会更容易找到代码中的错误信息来自何处。如果在多处使用 `unwrap`，则需要花更多的时间来分析到底是哪一个 `unwrap` 造成了 panic，因为所有的 `unwrap` 调用都打印相同的信息。

### 2.3. 传播错误



#### 2.3.1. 如何传播错误

我们上面的代码中，都是在出错的位置执行 `panic!` 的。

但是在一些场景下，在出错位置可能无法判断是应该解决错误，还是 `panic!`。

例如，有其他人调用你写的代码，你的代码出错了（前提是可恢复错误 `Result<T, E>`），但此时你无法预知调用你代码的人想要做什么，也就无法判断是应该解决错误，还是 `panic!`。

这种情况下，我们可以在代码中将此错误向上传递，把错误的处理交给调用者。这个叫做**传播(Propagating)**错误。

我们看示例代码，这段代码从文件中读取信息：

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

上面的代码中，有两个 `match` 表达式， 第一个表达式是打开文件的分支，第二个表达式是读取内容的分支。

在第一个 `match` 表达式中，可以看到，如果打开文件成功了，就会返回句柄给 f，如果失败了，则显式调用 `return`，将错误返回给调用者，中止执行函数。

在第二个 `match` 表达式中也是类似的，如果读取内容成功了，就返回读取到的内容给调用者，否则返回错误给调用者。由于这个 `match` 是整个函数最后的表达式，所以无需显式调用 `return`。

> `?` 与 `match` 表达式做的事还是有些不同的：
>
> `?` 运算符所使用的错误值被传递给了 `from` 函数，它定义于标准库的 `From` trait 中，其用来将错误从一种类型转换为另一种类型。
>
> 当 `?` 运算符调用 `from` 函数时，收到的错误类型被转换为由当前函数返回类型所指定的错误类型。这在当函数返回单个错误类型来代表所有可能失败的方式时很有用，即使其可能会因很多种原因失败。只要每一个错误类型都实现了 `from` 函数来定义如何将自身转换为返回的错误类型，`?` 运算符会自动处理这些转换。

#### 2.3.2. 传播错误的简写：`?` 运算符

Rust 中提供了一个运算符 `?` 用以简化传播错误的实现。

下面的示例代码使用了 `？` 运算符，其功能与上面的完全相同。

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

在第 6 行末尾，我们加了一个 `?`，这个的作用是，如果前面表达式（这个表达式返回值类型是 `Result<T, E>`）的值是 `OK`，`?` 就会返回其中的 `T`，如果前面的表达式的值是 `Err`，那么 `?` 就会将其返回给调用者，这个返回类似执行了 `return`，会中止函数，返回错误给调用者。

第 8 行同理，如果读取内容成功了，`?` 会返回 `Ok(_)`（`read_to_string()` 执行成功的话里面的 `T` 就是 `_`），如果出错了，就会返回错误给调用者，中止函数。我们这里没有用变量来接收 `Ok(_)`，因为这里什么也没有，读取的内容是存在 `s` 里面的，所以函数最后返回 `Ok(s)`。



上面的代码还可以进一步简化，功能完全一样，使用链式调用的方法：

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

这样写省去了一个变量 f，因为我们确实不需要它了。这是一个与众不同且更符合工程学(ergonomic)的写法。





#### 2.3.3. `?` 使用场景

我们知道，`main()` 函数的默认返回值是 `()`。

让我们看看在 `main()` 函数中使用 `?` 运算符会发生什么：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

会出错，错误信息如下：

```
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
   --> src/main.rs:5:36
    |
4   | / fn main() {
5   | |     let f = File::open("hello.txt")?;
    | |                                    ^ cannot use the `?` operator in a function that returns `()`
6   | | }
    | |_- this function should return `Result` or `Option` to accept `?`
    |
    = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`
note: required by `from_residual`
   --> /home/gukaifeng/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/try_trait.rs:339:5
    |
339 |     fn from_residual(residual: R) -> Self;
    |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0277`.
error: could not compile `hello-rust` due to previous error
The terminal process "cargo 'run'" terminated with exit code: 101.
```



错误信息的一开始就说了，`?` 运算符只能用于返回值类型为 `Result` 或 `Option` 或其他实现了 `FromResidual` 的函数。

我们的 `main()` 现在的返回值是 `()`，这不符合使用 `?` 的条件。

当你期望在不返回 `Result` 的函数中调用其他返回 `Result` 的函数时使用 `?` 的话，有两种方法解决这个问题：

1. 将函数返回值类型修改为 `Result<T, E>`，如果没有其它限制阻止你这么做的话；
2. 通过合适的方法（例如使用 `match` 或另一个 `Result`）来处理 `Result<T, E>`。



第 2 中方法在我们之前的一些示例中已经有所演示了，我们这里看一下第 1 种方法：

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```

我们这里修改了 `main()` 的返回值类型。

**`main()` 函数是比较特殊的，支持的返回值类型，除了 `()` 就只有 `Result<T, E>` 了。**

现在代码就可以正常编译通过了。

> `Box<dyn Error>` 被称为 “trait 对象”（“trait object”）。
>
> 这里不去讲解它，目前你可以把其简单地理解为，在 `main()` 中使用 `?`，`main()` 允许返回的任何类型的错误。
>
> 如果你不了解 trait，可以查看 [Rust -- 泛型、trait 与生命周期](https://gukaifeng.cn/posts/rust-fan-xing-trait-yu-sheng-ming-zhou-qi/) 这篇文章，或查阅相关文档。



#### 2.3.4. 扩展

其实上面的功能，Rust 中还有一个更简单的写法，只是这个写法会脱离我们学习的相关内容的初衷（因为这个写法都给实现好了）。不过这里还是给出这个写法：

```rust
use std::io;
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

这个写法和之前的区别就是，用一个方法 `fs::read_to_string()` 解决了所有问题。

因为打开文件，然后读取文件，这两个操作一起使用的频率实在是太高了，所以 Rust 标准库中提供了 `std::fs::read_to_string()` 这个方法（我们之前用的 `read_to_string()` 方法虽然和这个名字一样，但那个是文件句柄里的方法）。

`std::fs::read_to_string()` 就是把打开文件和读取文件两个操作打包在一起了，我们用着会更方便一些。

这个写法和之前的写法功能上完全相同！







## 3. 总结



Rust 的错误处理功能被设计为帮助你编写更加健壮的代码。

`panic!` 宏代表一个程序无法处理的状态，并停止执行而不是使用无效或不正确的值继续处理。

Rust 类型系统的 `Result` 枚举代表操作可能会在一种可以恢复的情况下失败。可以使用 `Result` 来告诉代码调用者他需要处理潜在的成功或失败。

在适当的场景使用 `panic!` 和 `Result` 将会使你的代码在面对不可避免的错误时显得更加可靠。
