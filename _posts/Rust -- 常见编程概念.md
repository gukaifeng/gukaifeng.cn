---
title: Rust -- 常见编程概念
date: 2021-09-11 23:13:48
categories: [technology]
tags: [Rust]
---



很多编程语言的核心概念都是共通的，Rust 也一样。  
这篇文章简单说说 Rust 中的一些常见编程概念，其中很多都与其他编程语言相同或相似。



### 1. 变量与可变性



#### 1.1. Rust 中的变量

Rust 中的变量默认是不可改变的(immutable)，不过可以加修饰符使其可以修改。

Rust 由编译器保证不可变的变量一定不会被修改。

**注意，这里的不可变，与其他语言中的 `const` 不一样，Rust 自己也有 `const` 修饰符，区别在 1.3 节说。**

<!--more-->

下面举一个例子：

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

执行 `cargo run` 编译并运行，会得到以下的报错信息

```
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables`

To learn more, run the command again with --verbose
```

报错信息显示，在我们上面代码的第 4 行，对一个不可变的 `x` 变量赋值。

当然可变的变量还是非常有用的，在 Rust 中，想把一个变量声明为可变的，需要加 `mut` 关键字，下面试试。

修改上述代码，在 `x`  变量声明的时候加上 `mut`，

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

再次编译运行，得到以下信息

```
The value of x is: 5
The value of x is: 6
```



#### 1.2. Rust 中的常量 `const`

Rust 中声明常量使用 `const` 而不是 `let`。  
Rust 中的 `const` 修饰的常量，必须在声明时说明类型，并赋值（可以是值或常量表达式）。  
Rust 中约定，常量的命名中英文字母全部大写，单词之间用下划线 `_` 隔开。

下面看个例子

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

这里声明了一个常量 `THREE_HOURS_IN_SECONDS`，类型为 `u32`，值为常量表达式 `60 * 60 * 3` 的结果。







#### 1.3. Rust 中默认的不可变变量与 `const` 修饰常量的区别



|  -   |  -   |                           默认变量                           |                    `const` 常量                     |
| :--: | :--: | :----------------------------------------------------------: | :-------------------------------------------------: |
|  1   | 命名 |                        普通的命名要求                        | （约定）英文字母全部大写，单词之间用下划线 `_` 隔开 |
|  2   | 类型 |                         无需声明类型                         |                    必须声明类型                     |
|  3   | 赋值 | 只可赋值一次，不可二次赋值。第一次赋值不一定要在声明时完成，可以先声明但不初始化，以后再赋值。 |                 声明时必须完成赋值                  |



#### 1.4. 隐藏(Shadowing)

Rust 中有一个隐藏概念，意思是你重复定义一个同名的变量，后定义的会将前面定义的隐藏（这在大多其他编程语言中，会在编译时报错重复定义变量）。下面要注意区分隐藏变量和 `mut` 修饰的可变变量的区别。

我们下面看个代码

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

输出

```
The value of x in the inner scope is: 12
The value of x is: 6
```



这里第 2 行定义了一个**不可变变量量** `x = 5`，在第 4 行又使用了关键字 `let` 定义了一个 `x` 不可变变量，注意这里前后的两个 `x` 都没加 `mut` 修饰，都是不可修改的！所以第 4 行定义的 `x` 和第 2 行定义的 x 已经不是同一个 `x` 了，第 4 行的 `x` 把第 2 行的 `x` 隐藏了！但是赋值表达式计算的时候，第 4 行声明的 `x` 还没有产生，所以计算 `x + 1` 时的 `x` 值为 5，第 4 行声明的 `x` 值则为 6。

再看第 7 行，这里是一个新的作用域了，参考其他编程语言的作用域相关知识，这里的隐藏概念就比较好理解了，新定义的 `x` 是一个局部变量，会隐藏外部作用域的 `x`，离开局部作用域以后，这个 `x` 就会消失。 

现在在看上面代码的输出结果，应该就可以理解了。



#### 1.5. 隐藏概念和 `mut` 修饰的可变变量的区别

|  -   |  -   |                             隐藏                             |               `mut`                |
| :--: | :--: | :----------------------------------------------------------: | :--------------------------------: |
|  1   | 赋值 | 必须使用 `let` 关键字，新变量是一个与被隐藏变量同名的新变量。 |  直接赋值就可以，还是同一个变量。  |
|  2   | 类型 | 新的变量类型可以和被隐藏的变量类型**不同**，即可以改变变量的类型 | 只能赋值给变量与原来相同类型的值。 |

第 1 点比较好理解，这里主要说一下第 2 点，关于类型的问题。

看下面的代码

```rust
fn main() {
    let spaces = "   ";
    println!("the value of spaces is {}", spaces);
    let spaces = spaces.len();
    println!("the value of spaces is {}", spaces);
}
```

输出如下

```
the value of spaces is    
the value of spaces is 3
```

我们可以看到，刚开始的 `spaces` 是一个字符串，后面变成了一个数值。

再看下面的代码

```rust
fn main() {
    let mut spaces = "   ";
    println!("the value of spaces is {}", spaces);
    spaces = spaces.len();
    println!("the value of spaces is {}", spaces);
}
```

这段代码与之前代码的区别是在第一个 `spaces` 声明时加了 `mut` 修饰，在第二个 `spaces` 赋值的时候去掉了 `let` 修饰（如果不去掉的话，还是会隐藏第一个 `spaces`，但会触发一个 `warning`，编译器会建议你去掉第一个 `mut`）。

编译会得到报错信息如下

```
error[E0308]: mismatched types
 --> src/main.rs:4:14
  |
4 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`.
error: could not compile `variables`

To learn more, run the command again with --verbose.
```

通过编译器报错信息，我们可以了解到，Rust 中不能改变 `mut` 变量的类型。



### 2. 数据类型

Rust 是 **静态类型**（*statically typed*）语言。  
Rust **在编译时就必须知道所有变量的类型**。

我们上面的代码，比如

```
let x = 5;
```

这样写没有声明 `x` 的类型，编译器会在编译前推断我们想用的类型。  
**在一些情况下，Rust 编译器如果无法推断出 `x` 可能的类型，或者有多种可能的类型，则会编译报错。**



Rust z中有两类数据类型子集，分别是**标量(scalar)**和**复合(compound)**。



#### 2.1. 标量类型

标量(scalar)类型代表一个单独的值。  
Rust 有四种基本的标量类型：整型、浮点型、布尔类型和字符类型（跟其他编程语言里的没啥太大区别）。

##### 2.1.1 整型

Rust 中的整型，有符号的以 `i` 开头，无符号的以 `u` 开头。  
表示范围依据长度，和其他编程语言一样。还有个整型溢出的问题，也和其他编程语言没区别，这里不展开说了。

|  长度   | 有符号  | 无符号  |                             备注                             |
| :-----: | :-----: | :-----: | :----------------------------------------------------------: |
|  8-bit  |  `i8`   |  `u8`   |                                                              |
| 16-bit  |  `i16`  |  `u16`  |                                                              |
| 32-bit  |  `i32`  |  `u32`  |                 Rust 中的默认整型是 `i32`。                  |
| 64-bit  |  `i64`  |  `u64`  |                                                              |
| 128-bit | `i128`  | `u128`  |                                                              |
|  arch   | `isize` | `usize` | 依赖运行程序的计算机架构。如在64 位架构上它们是 64 位的， 在 32 位架构上它们是 32 位的。主要作为某些集合的索引。 |

下面说一下 Rust 中的整型字面值

| 数值字面值                    | 样例                                                       | 备注                               |
| ----------------------------- | ---------------------------------------------------------- | ---------------------------------- |
| Decimal (十进制)              | `971030`, `97_1030`, `_971030`, `971030_`                  |                                    |
| Hex (十六进制)                | `0xff`, `0x_ff`, `0xf_f`, `0xff_`                          | 加前缀 `0x`                        |
| Octal (八进制)                | `0o77`, `0o_77`,`0o7_7`, `0o77_`                           | 加前缀 `0o`                        |
| Binary (二进制)               | `0b11110000`, `0b1111_0000`, `0b_1111_0000`, `0b11110000_` | 加前缀 `0b`                        |
| Byte (单字节字符)(仅限于`u8`) | `b'A'`, `b'#'`                                             | 加前缀 `b`，且字符需要写在单引号内 |

上面的例子可以看到，Rust 允许使用 `_` 做为分隔符以方便读数，例如 `1_000`，它的值与的 `1000` 相同，分隔符可以加载前缀后的任意位置，甚至写很多个也是没得问题的，比如 `10____00` 还是合法的，值还是等于 `1000`。

另外，**Rust 还支持使用类型后缀来指定某个字面值的类型**，下面举几个例子  
`1997u32` : 指 `u32` 类型的字面值 `1997`；  
`0xffi64`: 指 `i64` 类型的字面值 `0xff`。  
下面是代码示例  

```rust
fn main() {
    let x = 0xffu32;	//此时 x 的类型将不再是默认的 i32，而是我们指定的 u32
    println!("the value of x is {}", x);
}
```

输出

```
the value of x is 255
```



##### 2.1.2. 浮点型

Rust 有两个原生的**浮点数**（*floating-point numbers*）类型，`f32` 和 `f64`，分别占 32 位和 64 位。  
Rust 中浮点数采用 IEEE-754 标准表示。`f32` 是单精度浮点数，`f64` 是双精度浮点数。  
Rust 中浮点数的默认类型是 `f64`，因为在现代 CPU 中 `f64` 与 `f32` 的运算速度几乎一样且精度更高。

下面是一个展示浮点数的示例

文件名: src/main.rs

```rust
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
    let z = 3.0f32; // f32，使用类型后缀来指定字面值的类型
}
```



##### 2.1.3. 布尔类型

和其他大部分编程语言一样，  
Rust 中的布尔类型 `bool ` 有两个可能的值：`true` 和 `false`。

```rust
fn main() {
    let t = true;

    let f: bool = false; // 显式指定类型注解
}
```

##### 2.1.4. 字符类型

 `char` 类型是 Rust 语言中最原生的字符类型，由单引号指定，区别与使用双引号指定的字符串。

Rust 的 `char` 类型大小为 **4 个字节(four bytes)**，并代表了一个 **Unicode 标量值（Unicode Scalar Value）**。  
Unicode 标量值包含从 `U+0000` 到 `U+D7FF` 和 `U+E000` 到 `U+10FFFF` 在内的值。  
Unicode 意味着 Rust 中的 `char` 可以表示比 ASCII 码中更多的字符（大部分语言中的 `char` 只能表示 ASCII 表中的 128 个符号）。

在 Rust 中，拼音字母（Accented letters），中文、日文、韩文等字符，emoji（绘文字）以及零长度的空白字符都是有效的 `char` 值。

下面通过代码来看

```rust
fn main() {
    let lowercase_z = 'z';
    println!("this is a lowercase 'z': {}", lowercase_z);
    
    let hi = '嗨';
    let hi_code = '\u{55e8}';
    println!("this is a Chinese character '嗨': {}", hi);
    println!("this is a Chinese character '嗨' (using Unicode): {}", hi_code);
    
    let bear_face = '🐻';
    let bear_face_code = '\u{1f43b}';
    println!("this is an emoji -- Bear Face: {}", bear_face);
    println!("this is an emoji -- Bear Face (using Unicode): {}", bear_face_code);
}
```

输出

```
this is a lowercase 'z': z
this is a Chinese character '嗨': 嗨
this is a Chinese character '嗨' (using Unicode): 嗨
this is an emoji -- Bear Face: 🐻
this is an emoji -- Bear Face (using Unicode): 🐻
```



##### 2.1.5. 数值运算

Rust 中的所有数字类型都支持基本数学运算（加法、减法、乘法、除法和取余），和其他编程语言没区别。  
注意，和大多编程语言一下，整数除法当不能整除的时候，会有**截断**（向下取整）。

下面的代码简单展示了一下 Rust 中的基本数学运算。

```rust
fn main() {
    // 加法
    let sum = 5 + 10;

    // 减法
    let difference = 95.5 - 4.3;

    // 乘法
    let product = 4 * 30;

    // 除法
    let quotient = 56.7 / 32.2;
    let floored = 4 / 3; // 结果截断为 1

    // 取余
    let remainder = 43 % 5;
}
```



#### 2.2. 复合类型

复合类型(Compound types) 指的是将多个值组合成一个类型。  
Rust 有两个原生的复合类型：**元组(tuple)**和**数组(array)**。

##### 2.2.1. 元组类型

元组是一个将多个其他类型的值组合进一个复合类型的主要方式。  
元组长度固定，一旦声明，其长度不会增大或缩小。

使用 `()` 来创建一个元组，多个值使用 `,` 逗号隔开，其中每一个位置都有一个类型，并且这些类型可以不同。  

下面是例子

```rust
let tup0: (i32, f64, char) = (500, 6.4, 'c'); // 显式指定了元组 tup0 每个位置的类型
let tup1 = (500, 6.4, 'c');  // tup1 是一个元组，三个位置都由编译器推断类型
```

现在如何声明元组我们已经知道了，那怎么获取元组中的值呢？

------



获取元组中的值主要有两种方式，一个是**解构(destructuring)**，一个是通过索引 `.`。



先说解构，看代码

```rust
fn main() {
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);
}
```

输出

```
The value of y is: 6.4
```

可以看到，我们通过和 `tup` 一样的格式，将其中的内容以此绑定到 `x`, `y`, `z` 三个变量上，然后就可以通过这三个标量得到元组 `tup `中的值。  
但这种方式下，并不能通过 `x`, `y`, `z` 三个变量来修改 `tup` 中的内容，看下面的代码

```rust
fn main() {
    let mut tup = (500, 6.4, 1);  // 使 tup 可变
    let (x, mut y, z) = tup;  // 使 y 可变
    y = 5.4;
    println!("The value of y is: {}", y);
    println!("The value of tup.1 is: {}", tup.1);
}
```

输出（先忽略下编译器警告。这里会警告一些内容，例如 `x`, `z` 未使用，第一次绑定 `tup.1` 给 `y` 后还没有使用 `y`，就对 `y` 重新绑定值 `5.4` 覆盖掉了之前的值 `6.4`）

```
The value of y is: 5.4
The value of tup.1 is: 6.4
```

可以看到，虽然 `tup` 和 `y` 都是可变的，但并不能通过修改 `y` 来间接修改 `tup.1`，当然反过来也不行，`y` 和 `tup.1` 已经相互独立。

-

下面看看通过索引 `.` 访问元组内容的示例代码  
跟大多数编程语言一样，Rust 元组的索引值从 0 开始。

```rust
fn main() {
    let mut tup: (i32, f64, char) = (500, 6.4, 'c');
    println!("{} {} {}", tup.0, tup.1, tup.2);
    let mut x = tup.0;
    tup.1 = 5.4;
    x = 300;
    println!("{} {} {}", tup.0, tup.1, tup.2);
}
```

输出

```
500 6.4 c
500 5.4 c
```

可以看出，通过 `tup.1 = 5.4` 使得 `tup.1` 被实际修改，但通过 `x = 300` 并不会修改 `tup.0`。  
因为第 4 行中，只是将 `tup.0` 的值绑定给 `x`，之后 `x` 和 `tup.0` 已经相互独立。



---

在 Rust 中，没有任何值的元组 `()` 是一种特殊的类型。  
这种特殊的类型只有一个值，也写成 `()` ，这个类型被称为**单元类型(unit type)**，这个值被称为**单元值(unit value)**。  
如果一个表达式不返回任何其他值，则会隐式返回单元值。

##### 2.2.2. 数组类型

Rust 中，与元组不同，数组中的每个元素的类型必须相同。  
Rust 中的数组和 C/C++ 中的数组类似，和一些高级语言不同，其长度固定，一旦声明，数组的长度不能增长或缩小。  

这种数组类型长度不可变，存在**栈**中。后面会在其他文章中说怎样在堆中声明可变长度的动态数组。

Rust 中，数组中的值位于中括号内的逗号分隔的列表中。

下面通过代码举例说明 Rust 中数组的声明方式

```rust
fn main() {
    let arr0 = [1, 2, 3, 4, 5];  // 最朴素的声明方式
    let arr1: [i32; 5] = [1, 2, 3, 4, 5];  // 显式声明数组中存储数据的类型，以及数组的长度
    let arr2 = [3; 5];  // 变量名为 arr2 的数组长度为 5，这些元素的值都将被设置为 3
    let months = [  // 最朴素的声明方式
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
    ];  
}
```



关于数组元素的访问，就和其他编程语言一模一样了~

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];

    let first = arr[0];  // 只是把 arr[0] 的值绑定给 first，之后两者独立
    let second = arr[1];
    
    println!("1st is {}, 2nd is {}, 3rd is {}", first, second, arr[2]);
}
```

输出

```
1st is 1, 2nd is 2, 3rd is 3
```

同样的，你可以通过修改 `arr[2]` 来修改 `arr` 数组第 3 个元素的值，而不能通过 `first` 变量来修改 `arr` 数组的第 1 个值。

另外，和大多编程语言一样，当越界访问数组的时候，Rust 编译器也会报错，这里先不演示了。



### 3. 函数

和很多编程语言一样，Rust 程序的入口也是 `main()` 函数，上面的示例代码都有用到过，这里就不多说了。

Rust 定义函数的关键字是 `fn`。  
Rust 中的函数和变量名都使用 `snake case` 规范风格，所有字母都是小写，单词之间使用下划线分隔。

下面通过代码看看定义函数的示例

```rust
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

输出

```
Hello, world!
Another function.
```

函数的定义和调用后面要加 `()`，里面可以是空的，也可以是一些参数，带参的函数定义等下说。

注意下示例代码中我们自定义的函数定义在了 `main()` 函数下面，  
Rust 中函数定义可以在任意位置，而不像部分其他编程语言一样定义（或声明）必须在调用之前。

---



上面的代码示例中的函数定义是没有参数的，下面看看带参数的函数定义（调用）代码示例

```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

输出

```
The measurement is: 5h
```

上面的代码中我们给函数 `print_labeled_measurement` 定义了两个参数

* `value`: 一个 `i32` 类型的形参；
* `unit_label`: 一个 `char` 类型的形参。



这里是比较好理解的，就不多说了，下面说 Rust 中带返回值的函数，稍微特殊一些。

---



在看关于 Rust 中带返回值函数定义的问题之前，我们得先了解一个比较重要的概念，**Rust 是一门基于表达式(expression-based)的语言**，这个概念是大部分编程语言都不具有的。

Rust 是一门基于表达式(expression-based)的语言，除了形如 `1 + 2` 这样的是表达式以外，函数调用是一个表达式，宏调用是一个表达式，**我们用来创建新作用域的大括号(代码块) `{}`，也是一个表达式**，看看下面的示例代码。

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

输出（忽略 `warning`）

```
The value of y is: 4
```

上面的代码中，有一个大括号(代码块)表达式

```rust
{
    let x = 3;
    x + 1
}
```

注意其中的第二个语句，是没有以 `;` 结尾的。  
在 Rust 中，以 `;` 结尾的是不返回值的，返回值的语句结尾不加 `;`。

这个代码块的值是 `4`，这个值作为 `let` 语句的一部分被绑定到 `y` 上。

---

现在看返回值的函数定义方式。

Rust 中的函数定义返回值，要在函数定义的 `()` 后添加 `->` 再接类型名。看看代码示例

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    println!("The value of x is: {}", x);
}
```

输出

```
The value of x is: 5
```



上面的代码中，函数 `five()` 的返回值为 `i32` 类型，函数中返回了一个值为 `5`。

再看看另一个代码示例，这里定义一个既有参数又有返回值的函数

```rust
fn main() {
    let x = plus_one(5);
    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

输出

```
The value of x is: 6
```

上述代码还是好理解的，函数的含义是返回 `x + 1` 的结果，不多说。

---

另外 Rust 中也是有 `return` 关键字的，其可以提前结束函数并返回值。  
**注意 `return` 关键字只可以用在函数体中，但不可以用在普通大括号表达式中。**  
**因为 `return` 之后结束的是整个函数，而不是只是大括号代码块，用在 `main()` 里面大括号表达式结束的是 `main()` 函数。**

下面用代码来演示这一特性

```rust
fn main() {
    let y = plus_or_sub_x(3);
    println!("The value of y is: {}", y);
}

fn plus_or_sub_x(x: i32) -> i32 {
    if x < 10 {
        println!("exec \"return x + 1;\"");
        return x + 1;
    }
    println!("exec \"x - 1\"");
    x - 1
}
```

输出

```
exec "return x + 1;"
The value of y is: 4
```

上述代码中定义了一个函数，可以看出函数在第 9 行提前返回值了，并没有继续执行到后面第 12 行的 `x - 1`。

---



有的小伙伴可能会问，按前面所将的，这段代码使用 `return` 是不是多余，因为不带 `;` 的 `x + 1`不已经是返回值了吗？

我们来试一下，删掉 `return` 和这个语句后面的分号 `;`，结果代码如下

```rust
fn main() {
    let y = plus_or_sub_x(3);
    println!("The value of y is: {}", y);
}

fn plus_or_sub_x(x: i32) -> i32 {
    if x < 10 {
        println!("exec \"return x + 1;\"");
        x + 1
    }
    println!("exec \"x - 1\"");
    x - 1
}
```

编译会提示报错信息

```
error[E0308]: mismatched types
  --> src/main.rs:21:9
   |
19 | /     if x < 10 {
20 | |         println!("exec \"return x + 1;\"");
21 | |         x + 1
   | |         ^^^^^ expected `()`, found `i32`
22 | |     }
   | |_____- expected this to be `()`
   |
help: consider using a semicolon here
   |
22 |     };
   |      ^
help: you might have meant to return this value
   |
21 |         return x + 1;
   |         ^^^^^^      ^

For more information about this error, try `rustc --explain E0308`.
```

意思是 `mismatched types`，不匹配的类型，在 `x + 1` 这里，期待返回一个单元类型的值 `()`。

这里涉及到下面即将说的控制流的知识，先简单解释一下。   
在 Rust 中，`if` `else` 应当是成对使用的，且如果有返回值，则 `if` 和 `else` 代码块中的返回值必须类型相同。  
Rust 中没有返回值等价于返回一个单元类型的值 `()`。

现在问题出在哪应该就知道了。  
我们上面的代码中没有写 `else`，但不代表 Rust 编译器认为没有，由于之前的 `if` 没有显式的指定返回值类型（只写 `x + 1` 是没有注明类型的，这里无关函数定义处的返回值类型，`if` `else` 代码块才是一个整体），所以编译器会认为这有一个没有返回值 `else` 空代码块。因为编译器认为 `else` 代码块返回一个单元类型，那 `if` 语句应当也返回一个单元类型（等价于不返回值），也就有了上面的报错信息。

解决方案有两种，第一种就是像我们之前的代码那样，在 `if` 中使用 `return` 关键字显式的指定返回值，编译器会推测出 `else` 语句（假如有的话）中应有的返回值类型，而不是以 `else` 的返回值类型单元类型来决定 `if` 语句中的返回值类型；第二种是显式地把后面的 `x - 1` 语句放在 `else` 中，如下代码。

```rust
fn main() {
    let y = plus_or_sub_x(3);
    println!("The value of y is: {}", y);
}

fn plus_or_sub_x(x: i32) -> i32 {
    if x < 10 {
        println!("exec \"return x + 1;\"");
        x + 1
    } else {
        println!("exec \"x - 1\"");
        x - 1
    }
}
```

---



`return` 关键字可以提前结束函数，**即便这个函数没有返回值**，我以前使用其他编程语言写递归函数的时候，常常用到这个特征。

这里举一个简单的例子~~（当然这个例子实现的内容有更好的写法，这里只是演示我们要看的特征）~~

下面的代码递减的输出从 `x` 到 `0` 的数，输出到 `0` 时结束整个 `no_return_val()` 函数。

```rust
fn main() {
    no_return_val(3);    
}

fn no_return_val(x: i32) {
    let mut i = x;
    loop {
        if i < 0 {
            return ();
            // return ; 两种写法均可
        }
        println!("{}", i);
        i = i - 1;
    }
}
```

输出

```
3
2
1
0
```





### 4. 控制流



#### 4.1. 条件分支 `if` `else if` `else`

一般情形下，Rust 中的 `if` `else if` `else` 和其他语言没什么不同，记得判断语句不加括号就好了。

下面通过简单示例说明，主要看看怎么用就好，不过多解释了。

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

输出

```
number is divisible by 3
```



---



上面说过 Rust 是一门基于表达式(expression-based)的语言，可以使用大括号(代码块)表达式。  
之前只是使用了普通的大括号表达式，不过在 Rust 中，大括号表达式可以搭配 `if` `else if` `else`  一起用。

下面通过代码来看，主要看怎么用即可

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

输出

```
The value of number is: 5
```

**要注意的是，前面也说过，Rust 中 `if`  `else if`  `else` 代码块如果有返回值，那么返回值类型必须相同。**  
下面的代码就是错误的

```rust
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

编译器报错，因为 `if` 和 `else` 代码块的返回值类型不一样。

```
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:44
  |
4 |     let number = if condition { 5 } else { "six" };
  |                                 -          ^^^^^ expected integer, found `&str`
  |                                 |
  |                                 expected because of this

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches`

To learn more, run the command again with --verbose.
```





#### 4.2. 循环

在说具体的循环关键字之前，先说两个干预循环的关键字 `continue` 和 `break`。

这两个关键字的作用在一般情形下，和大部分编程语言保持一致。

* `continue`: 跳过本次循环中的剩余部分，开始下一次循环；
* `break`: 直接结束整个循环。

这两个关键字所影响的循环，在其他编程语言中一般是**距离这两个关键字最近的外层循环**，  
在 Rust 中的一般情境下也是一样的，具体不一样的地方我们在后面具体说。

##### 4.2.1  `loop`

看个简单代码

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

这是一个死循环，会不断的输出 `again!`，可能需要强制结束程序才能中断输出。

刚说过，一般情景下，如果存在嵌套循环，`break` 和 `continue` 默认应用于此时最近的外层循环。  
不过在 Rust 中你可以选择在一个循环上指定一个**循环标签(loop label)**，然后将标签与 `break` 或 `continue` 一起使用，使这两个关键字应用于某个特定的循环，而不是默认最近的外层循环。

下面是一个包含两个嵌套循环的示例，**注意写法格式**

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

输出

```
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

外层循环有一个标签 `counting_ up`，它将从 0 数到 2。  
没有标签的内层循环从 10 向下数到 9。  
第一个没有指定标签的 `break` 将只退出最近的外层循环（即代码中两个嵌套循环中的内层循环），而 `break 'counting_up` 语句将退出外层循环。

---

`loop` 循环可以有返回值，如果将返回值加入你用来停止循环的 `break` 表达式，那这个值会被**被停止的循环**返回。

同样的，**注意写法格式**

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

输出

```
The result is 20
```





##### 4.2.2.  `while`

Rust 中的 `while` 循环和其他编程语言没什么两样，从下面的代码 中看看写法格式即可

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);
        number = number - 1;
    }

    println!("LIFTOFF!!!");
}

```

输出

```
3!
2!
1!
LIFTOFF!!!
```



##### 4.2.3.  `for`

学过` while` 循环后，我们就可以用 `while` 来遍历一个数组，例如下面的代码

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}
```

输出

```
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

-

但显然这不是一个好的方法，所以我们下面看看 `for` 循环，注意写法格式

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

输出和上面的代码一样，但更简洁清晰。

```
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```



-

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

这段代码用 `for` 从 `3` 倒序遍历到 `1`，注意 `(1..4)` 是**左闭右开**，表示从 `1` 到 `3`，后面的 `rev()` 函数表示倒序，  
即输出如下

```
3!
2!
1!
LIFTOFF!!!
```





### 5. 注释

Rust 中的普通注释有两种，和 C/C++ 一样，行注释 `//` 和 块注释 `/* */`。  
Rust 中还有一种是文档注释，这里先不说，这个文档注释会在后面的文章中来解释。



下面举例说一下两种普通注释~~（其实上面的代码都有用注释来着）~~

```rust
// hello, world

// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.

/*
So we’re doing something complicated here, long enough that we need
multiple lines of comments to do it! Whew! Hopefully, this comment will
explain what’s going on.
*/

fn main() {
    // I’m feeling lucky today
    let lucky_number = 7; // I’m feeling lucky today
}
```

~~太简单了，我不想多写了，哈哈 😄。。。。。。~~