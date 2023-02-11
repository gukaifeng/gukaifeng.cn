---
title: "C/C++ 程序编译过程简述"
date: 2023-02-11 13:48:00
updated: 2023-02-11 13:48:00
categories:
- 编程语言基础
- Cpp
tags: [C,Cpp]
---



C/C++ 程序最简单的 "Hello World" 几乎是每个程序员闭着眼睛就能写出的，编译运行过程一气呵成，基本成了程序入门和开发环境测试的默认标准。

例如，一个最基本的 "Hello World" 的 C 语言程序像下面这样：

```c
#include <stdio.h>

int main() {
    printf("Hello World\n");
    return 0;
}
```

在 Linux 中，当我们使用 GCC 来编译此程序时，只需要使用最简单的命令（假设源文件名为 "hello.c"）：

```shell
$ gcc hello.c
$ ./a.out
Hello World
```

