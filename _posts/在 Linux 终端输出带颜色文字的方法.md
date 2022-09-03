---
title: 在 Linux 终端输出带颜色文字的方法
date: 2022-09-03 18:49:00
updated: 2022-09-03 18:49:00
categories: [技术杂谈]
tags: [Linux]
toc: true
---

我们经常会看到一些程序在 Linux 终端的输出是带有颜色的。

除了默认颜色外，最常见的应该是<font color=red>红色</font>、<font color=yellow>黄色</font>和<font color=green>绿色</font>，分别代表<font color=red>错误(Error)/失败(Failed)</font>、<font color=yellow>警告(Warning)</font>和<font color=green>成功(Successed)</font>。当然可以输出的不止这三个颜色，只是这三个最常用。

本文以这三个颜色为例，说明在 Linux 终端输出带颜色文字的方法。

**首先说明，在终端中给文字加颜色和使用的编程语言没有关系，不论是 shell，还是 C/C++、Java、Python 或是 Go 语言的控制台输出，等等，都是一样的。带颜色的文字，其实只是普通文字加了转义字符罢了，没有什么特殊之处。**

此外，在终端中，除了可以给文字加颜色，也可以给文字加背景色，背景色在某些场景下也十分有用，这个在后面也会说。

## 1. 示例演示

下面先看一个例子，使用 **C++** 语言输出：

```cpp
#include <iostream>

int main(int argc, char* argv[]) {
    std::cout << "C/C++ 在 Linux 终端输出 \e[31m红色\e[m" << std::endl;
    std::cout << "C/C++ 在 Linux 终端输出 \e[33m黄色\e[m" << std::endl;
    std::cout << "C/C++ 在 Linux 终端输出 \e[32m绿色\e[m" << std::endl;

    return 0;
}
```

输出截图如下（使用截图是因为代码块在这里无法正确显示颜色）：

![](https://gukaifeng.cn/posts/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa_1.png)

我们再使用 **Shell** 语句试一试：

![](https://gukaifeng.cn/posts/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa_2.png)

## 2. 标准语法

```
\e[Param {;Param;...}m
```

* `\e`：表示转义开始。与 `\033` 等价，可互换。

* `Param`：转义参数。为空时（即 `\e[m`）等价于 `\e[0m`，表示**重置**。多个参数用分号 `;` 隔开。

* `m`：表示转义结束。

> 请注意，在一般情况下，在希望有效果的文字之后，一定要加 `\e[m` 或 `\e[0m` 重置效果。不然的话，可能你整个终端都是这样的了！

## 3. 输出彩色字符

我们以这个语句为例：

```
\e[31m红色\e[m
```

输出颜色开始的转义为 `\e[31m`，其中 `31` 代表红色。

`\e[m` 代表重置，也就是说后面的文字就不再有红色了。

## 3. 给文字加上背景色

大部分 Linux 终端的默认背景色都是深色的（如黑色、深灰色等），所以我们最常用的红黄绿三个颜色的显示在大多数时候并没有什么问题。但如果有用户将终端背景色设置为浅色（如白色，浅灰色等），那像黄色这样的颜色恐怕就很不清晰了。

为了我们输出的颜色文字能在任何场景下都可以被识别，我们可以给文字加一个背景色。

![](https://gukaifeng.cn/posts/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa_3.png)

通过观察语句我们可以发现，**语法和上面修改文字颜色是一模一样的！**

**事实上，只是不同的颜色代码而已！比如，31 代表红色文字，41 代表红色背景！**

## 5. 转义对照表

颜色一共只有 8 个，分为文字颜色（30\~37）和背景颜色（40\~47），如下：

| 颜色  | 文字色代码 | 背景色代码 |
| --- | ----- | ----- |
| 黑色  | 30    | 40    |
| 红色  | 31    | 41    |
| 绿色  | 32    | 42    |
| 黄色  | 33    | 43    |
| 蓝色  | 34    | 44    |
| 紫色  | 35    | 45    |
| 青色  | 36    | 46    |
| 白色  | 37    | 47    |

还有一些其他比较常用的文字效果代码，这里也给出：

| 功能          | 代码     |
| ----------- | ------ |
| 重置默认        | 0      |
| 加粗 / 取消加粗   | 1 / 22 |
| 闪烁 / 取消闪烁   | 5 / 25 |
| 下划线 / 删除下划线 | 4 / 24 |
| 反显 / 取消反显   | 7 / 27 |

> 无转义参数时，默认有参数为 0。
> 
> 反显的含义是反转文字颜色和背景色。

例如白底、红色、加粗、下划线：

![](https://gukaifeng.cn/posts/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa/zai-linux-zhong-duan-shu-chu-dai-yan-se-wen-zi-de-fang-fa_4.png)
