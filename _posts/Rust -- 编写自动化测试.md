---
title: Rust -- 编写自动化测试
date: 2021-12-09 18:11:11
updated: 2021-12-09 18:11:11
categories: [编程语言基础]
tags: [Rust]
toc: true
---

Rust 是一个相当注重正确性的编程语言。

Rust 的类型系统在此问题上下了很大的功夫，不过这仍然不可能捕获所有种类的错误。

为此，Rust 也在语言本身包含了编写软件测试的支持。

例如，我们可以编写一个叫做 `add_two()` 的函数，将传递给其的值加 2。其声明有一个整型参数并返回一个整型值。当实现和编译这个函数时，Rust 会进行所有目前我们已经见过的类型检查和借用检查，例如，这些检查会确保我们不会传递 `String` 或无效的引用给这个函数。不过，Rust 无法检查这个函数是否会准确地完成我们期望的工作：返回参数加 2 后的值。这也就是测试出场的地方。

我们可以编写测试断言，比如说，当传递 `3` 给 `add_two()` 函数时，返回值是 `5`。无论何时对代码进行修改，都可以运行测试来确保任何现存的正确行为没有被改变。

这篇文章我们来使用 Rust 语言本身提供的测试功能！

<!--more-->

## 1. 如何编写测试

测试函数体通常执行如下三种操作：

1. 设置任何所需的数据或状态；
2. 运行需要测试的代码；
3. 断言其结果是我们所期望的。

让我们看看 Rust 提供的专门用来编写测试的功能：`test` 属性、一些宏和 `should_panic` 属性。



### 1.1. 简单的测试

我们先新建一个库 crate，然后在 `src/lib.rs` 中写入下面的代码：

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn successful_test() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn failed_test() {
        panic!("Make this test fail");
    }
}
```

执行 `cargo test`，会得到以下信息：

```
running 2 tests
test tests::failed_test ... FAILED
test tests::successful_test ... ok

failures:

---- tests::failed_test stdout ----
thread 'tests::failed_test' panicked at 'Make this test fail', src/lib.rs:10:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::failed_test

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```



信息第 1 行显示执行了两个测试；

第 2、3 行分别显示了两个测试的名称和测试结果，测试的名称即我们函数的名称，结果为 ok 表示测试通过，结果为 FAILED 表示测试未通过。

然后接下来是测试未通过的失败信息，第 7-9 行和第 12-13 行显示了我们 failed_test() 函数的测试失败信息。

最后 15 行是整个测试的摘要：总的最终结果是 FAILED（所有测试只要有一个没通过，最终的结果就是 FAILED），1 个通过，1 个失败，0 个忽略(ignored)，0 个性能测试(measured)，0 个被过滤掉(filtered out)，和测试用时。

ignored 和 filtered out 我们会在后面介绍。

measured 还不完善，属于 Rust 开发版(Nightly Rust)的功能，这里就先不说了。



**Rust 测试判断通过或者不通过其实很简单，只要程序正确执行了，就是通过的，如果出错了，就是不通过。**

成功执行很好理解，出错的可能有很多，例如我们使用 `assert!()` 来判断某些结果是否是我们想要的，如果结果正确， `assert!()` 什么都不会做，程序就正确运行，如果结果不正确，`assert!()` 中就会执行 `panic!`。

我们把上面的 assert!() 修改一下，看看结果；

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn successful_test() {
        assert_eq!(2 + 2, 5);
    }

    #[test]
    fn failed_test() {
        panic!("Make this test fail");
    }
}
```

然后看看关于这个部分的输出，

```
---- tests::successful_test stdout ----
thread 'tests::successful_test' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:5:9
```

还是比较清楚的。

注意不论是 `assert_eq!()` 还是 `assert_ne!()`，第一个参数是 `left`，第二个参数是 `right`，参数的次序不影响结果。



### 1.2. 自定义断言的错误信息

不论是 `assert!()` 还是 `assert_eq!()` 还是 `assert_ne!()`，在主要的断言比较参数之后，所有的参数都会传递给 `format!()`。

我们看下面的测试：

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn failed_test() {
        let num1 = 2;
        let num2 = 2;
        let wrong_result = 5;
        assert_eq!(num1 + num2, wrong_result, "test failed, {} + {} != {}", num1, num2, wrong_result);
    }
}
```

执行 `cargo test`， 输出如下：

```
running 1 test
test tests::failed_test ... FAILED

failures:

---- tests::failed_test stdout ----
thread 'tests::failed_test' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`: test failed, 2 + 2 != 5', src/lib.rs:8:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::failed_test

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

可以看到打印出了我自定义的错误信息："test failed, 2 + 2 != 5"。

有了自定义的错误信息，我们找测试失败的位置就很容易了。



### 1.3. 使用 `should_panic` 检查 panic

我们上面说过，判断一个测试是否通过，只要看程序是否正确执行就可以了，如果没出错就是通过了，出错了就是没通过。

简单来说我，我们的之前测试程序，通过才是我们想要的结果。

我们还可以再给测试代码加一个 `#[should_panic]`，位于 `#[test]` 之后，这样，我们的测试程序，失败了才是我们想要的结果。当程序 panic 时，Rust 会将其判定为通过！

我们看一下示例代码

```rust
#[cfg(test)]
mod tests {
    #[test]
    #[should_panic]
    fn should_panic_test() {
        panic!("this is a panic");
    }
}
```

```
running 1 test
test tests::should_panic_test - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests testdemo

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

我们的程序触发了 panic，但是测试是通过的！

> 我们先忽略下第 6 行以后的内容，这部分是文档测试的结果，文档测试不是本文要说的，这里先不管。



我们再看看程序如果没有出发 panic 会如何：



```rust
#[cfg(test)]
mod tests {
    #[test]
    #[should_panic]
    fn should_panic_test() { }  // 这里程序正确执行完成了
}
```

执行测试，输出：

```
running 1 test
test tests::should_panic_test - should panic ... FAILED

failures:

---- tests::should_panic_test stdout ----
note: test did not panic as expected

failures:
    tests::should_panic_test

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

当我们的程序没有 panic 时，测试不通过！

-

但是！

上面的 `should_panic` 的测试通过的信息中，仅仅只告诉我们，程序确实发生了 panic，就再也没有更多有用的信息了，但是引发 panic 的可能有很多种，我们该如何确保这是一个我们想要的 panic 呢？

`should_panic` 可以添加一个参数 `expected`，指定你期望的 panic 信息，如果程序发生了 panic，但是 panic 信息和你期望的不一样，测试一样不会通过！

我们看下面的代码，注意 参数`expected` 的写法（在 `should_panic`  后面的括号里）：

```rust
#[cfg(test)]
mod tests {
    #[test]
    #[should_panic(expected = "this is a panic message which is our expected")]
    fn should_panic_test() {
        panic!("this is a panic message which is not our expected");
    }
}
```

这段程序，确实 panic 了，但是我们看看测试结果：

```
running 1 test
test tests::should_panic_test - should panic ... FAILED

failures:

---- tests::should_panic_test stdout ----
thread 'tests::should_panic_test' panicked at 'this is a panic message which is not our expected', src/lib.rs:6:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
note: panic did not contain expected string
      panic message: `"this is a panic message which is not our expected"`,
 expected substring: `"this is a panic message which is our expected"`

failures:
    tests::should_panic_test

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

测试确实没有通过，并且第 10 行指出了我们实际的 panic 信息，第 11 指出了我们期望中的 panic 信息，两个信息不匹配，所以测试失败。

现在我们把 panic 信息改成一样的再试试：

```rust
#[cfg(test)]
mod tests {
    #[test]
    #[should_panic(expected = "this is a panic message which is our expected")]
    fn should_panic_test() {
        panic!("this is a panic message which is our expected");
    }
}
```

运行测试：

```
running 1 test
test tests::should_panic_test - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

通过！





### 1.4. 将 `Result<T, E>` 用于测试

我们之前的测试程序，都是在失败的时候 panic。

还有一种测试方法，就是使用 `Result<T, E>`。

我们看一段代码：

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

测试输出：

```
running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

`it_works` 函数的返回值类型为 `Result<(), String>`。

这个测试方法不同于调用 `assert_eq!` 宏，当测试通过时返回 `Ok(())`，当测试失败时返回一个包含 String 的 `Err`。

这样编写测试来返回 `Result<T, E>` 就可以在函数体中使用 `?` 运算符，这是编写测试的一种方便方法，如果测试中的任何操作返回 `Err` 变量，则测试将不通过。

注意，你不能对这些使用 `Result<T, E>` 的测试使用 `#[should_panic]` 注解，相反，你只应该在测试失败时直接返回 `Err` 值。



## 2. 控制测试的运行

与使用 `cargo run` 会编译代码，然后运行生成的二进制文件一样，`cargo test` 在测试模式下编译代码并运行生成的测试二进制文件。

默认情况下，Rust 中使用 `cargo test`，会

1. **并行**运行所有的测试；
2. 截获测试运行中产生的标准输出，以阻止这些输出显示出来，使我们查看相关测试结果更容易。

我们可以给 `cargo test` 传递参数，改变其默认行为。

在 cargo test 后面的参数，除了我们刚刚说的改变测试默认行为的参数外，还可能有测试程序需要的参数，这两种参数用 `--` 隔开，即：

```
cargo test [测试二进制文件参数] [...] -- [cargo test 的参数] [...]
```

两种参数都可以有多个。





### 2.1. 并行或串行地运行测试



默认情况下，Rust 使用多线程运行测试，这效率更高，但是如果你的不同测试文件中相互依赖、相互影响，多线程运行就会导致一些意外的结果。

我们可以给 `cargo test` 传递参数 `--test-threads` 指定运行线程数，将其设置为 1，即为单线程串行地运行测试。即

```shell
cargo test -- --test-threads=1
```

注意我们没有给二进制文件传参数，所以只有 `--` 后面有参数，前面没有。

单线程测试更慢，但是更安全，测试之间相互不会有干扰。



### 2.2. 显示测试的标准输出

默认情况下，如果测试通过，Rust 会截获测试中的标准输出。比如我们的函数中有调用 `println!()` 宏，如果我们的测试通过了，那么我们将看不到其打印到标准输出的内容。我们只能看到的失败了的测试里面执行到了的 `println!()` 打印的内容。

看下面的示例代码，这里有一个会通过的测试和一个会失败的测试：

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

我们运行测试，输出如下：

```rust
running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

通过的测试什么也没有输出，失败的测试则打印出了全部的标准输出。

第 7 行 `---- tests::this_test_will_fail stdout ----` 写明了下面的内容是失败的测试的标准输出内容。

如果你希望也能看到通过的测试的输出内容，可以使用 `--nocapture` 参数。

我们用 `cargo test -- --nocapture` 再试一次，输出如下：

```
running 2 tests
I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rsI got the value 4
:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

在第 5 行后半段，"I got the value 4" 就是我们通过测试的输出！

你可能会奇怪为什么这个输出出现在这里，我们再试一次，输出如下：

```
running 2 tests
I got the value 8
I got the value 4
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

"I got the value 4" 跑到了第三行的位置！

我们之前说过，Rust 中的多个测试，默认是多线程并发执行的，所以通过的测试的输出内容，和失败的测试的输出内容混在了一起。

当禁用捕获以后，测试的输出可能会很乱，这时候还是建议加上我们之前说过的 `` 参数，单线程执行比较好。

```shell
cargo test -- --test-threads=1 --nocapture
```



我们试一试，输出如下：

```
running 2 tests
test tests::this_test_will_fail ... I got the value 8
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
FAILED
test tests::this_test_will_pass ... I got the value 4
ok

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

这次，起码各种信息都在它应在的位置。。。



### 2.3. 运行指定的测试



当我们有很多段测试代码，可能不想全都测试一遍。

Rust 也提供了比较简单的方法，我们可以只测试一段我们指定的代码。

```shell
cargo test [测试代码关键字]
```

注意！这里是关键字，关键字就是要执行代码的函数名字、模块名字的部分或全部！

-

我们举一个例子，看下面的代码：

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod keyword_tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

我们执行 `cargo test add`，会测试 `add_two_and_two()` 和 `add_three_and_two()` 两个函数，因为这两个函数的名字中含有 "add"。输出如下：

```
running 2 tests
test keyword_tests::add_three_and_two ... ok
test keyword_tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

甚至，我们执行 `cargo test and` 也会是和 `cargo test add` 一样的结果，因为只有 `add_two_and_two()` 和 `add_three_and_two()` 这两个函数中有 "and"。

如果我们想单独测试 `one_hundred()` 方法，可以执行 `cargo test one`，因为上述代码里只有这一个函数名字中有 `one`。输出如下：

```
running 1 test
test keyword_tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

当然你也可以考虑写完整的函数名，例如 `cargo test one_hundred`，这结果是一样的。

-

我们之前说的是测试代码名称中的关键字，可没有说是函数名字，我们上面的示例都是用的函数名字中的部分。我们同样可以使用模块名字中的部分，这可以运行这个模块下所有的测试。

我们执行 `cargo test keyword` 试一试，"keyword" 是上述代码中模块名字的一部分，输出如下：

```
running 3 tests
test keyword_tests::add_three_and_two ... ok
test keyword_tests::add_two_and_two ... ok
test keyword_tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

-

使用部分名称，有些时候可能会有些模糊，但是非常方便！

如果你想要精确指定测试一个代码段，那你可以使用代码段完整的名字，这样肯定是最合适的。

另外，不能给 `cargo test` 传递多个关键字，只能一个。



### 2.4. 过滤测试



### 2.5. 忽略测试







## 3. 测试的组织结构

