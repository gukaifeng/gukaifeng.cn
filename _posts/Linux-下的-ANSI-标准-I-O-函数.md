---
title: Linux 下的 ANSI 标准 I/O 函数
mathjax: false
date: 2021-01-01 08:37:39
updated: 2021-01-01 08:37:39
tags: [Linux, C, I/O]
categories: [技术杂谈]
toc: true
---


## 打开文件 fopen()
用于文件和终端的输入输出，类似于系统调用 open。
```c
#include <stdio.h>

FILE * fopen(const char *filename, const char *mode);
// 成功时返回一个 FILE * 指针，失败时返回 NULL。
```
* filename: 要打开的文件。
* mode: 打开文件的方式，有一些取值。

关于 FILE 的结构，如下：
```c
typedef struct _iobuf {
    int cnt;      // 剩余字符数
    char *ptr;     // 下一个字符的位置
    char *base;    // 缓冲区的位置
    int flag;     // 文件访问模式
    int fd;       // 文件描述符
} FILE;
```

关于 `mode` 参数的取值如下：
|值（字符串）|含义|
|-|-|
|"r", "rb"|以只读方式打开文件。|
|"w", "wb"|以写方式打开文件，并把文件长度截断为0。|
|"a", "ab"|以写方式打开文件，新内容追加在文件尾。|
|"r+", "rb+", "r+b"|以更新方式打开文件（读和写）。|
|"w+", "wb+", "w+b"|以更新方式打开文件，并把文件长度截断为 0。|
|"a+", "ab+", "a+b"|以更新方式打开文件，新内容追加在文件尾。|

其中，字母 `b` 表示文件是一个二进制文件而不是文本文件。另外，需要注意的是，`mode` 参数是一个字符串而不是一个字符，因此总是需要使用双引号 `" "` 括起来。

## 读取文件 fread()
fread 函数用于从一个文件流里读取数据。调用 fread 成功时，返回成功读到缓冲区里的记录个数（不是字节数）。
```c
#include <stdio.h>

size_t fread(void *ptr,size_t size,size_t nitems,FILE *stream);
```
* ptr: 缓存区起始地址，将文件流的数据读到这个缓存区。
* size: 每个待读取记录的长度。
* nitems: 待读取记录的个数。
* stream: 待读取的文件流。

举个栗子：
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int buffer[10];
    int i;
    int counter;
    counter = fread(buffer, 1, 10, stdin);
    printf("读取到缓冲区的记录个数为：%d\n", counter);
    fwrite(buffer, sizeof(int), 1, stdout);

    return 0;
}
```
* 这段代码从标准输入读取 10 个 字符到缓冲区中，然后调用 fwrite 函数将缓冲区中的前 4 个字符写到标准输出。



## 写入文件 fwrite()

fwrite 函数从指定的数据缓冲区里取出数据记录，并把它们写到输出流中，返回成功写入的记录个数，其参数与 fread 函数的参数类似。
```C
#include <stdio.h>

siize_t fwrite(const void *ptr,size_t size,size_t nitems,FILE *stream);
```

* ptr: 缓存区起始地址，将缓存区的数据写到文件流。
* size: 每个待写入记录的长度。
* nitems: 待读取写入的个数。
* stream: 待写入的文件流。


需要注意的是，用 fwrite 写的文件在不同的计算机体系结构之间可能不具备可移植性，因此，不推荐把 fread 和 fwrite 用于结构化数据。演示代码如下：

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    char buffer[] = "hello,my name is gukaifeng!\n";
    fwrite(buffer, 1, sizeof(buffer), stdout);

    return 0;
}

/* 输出结果： 
hello,my name is gukaifeng!
*/
```
这段代码将 buffer 中的内容写入标准输出。

## 关闭文件 fclose()
fclose 函数关闭指定的文件流 stream，使所有尚未写出的数据都写出。因为 stdio 库会对数据进行缓冲，所以使用 fclose 是很重要的。如果程序需要确保数据已全部写出，则应该调用 fclose 函数。
```c
#include <stdio.h>

int fclose(FILE *stream);
```

## 写出文件流中的数据 fflush()
fflush 函数的作用是把文件流里所有未写出的数据立刻写出。
```c
#include <stdio.h>

int fflush(FILE *stream);
```
在 调用 fclose 函数时，隐含执行了一次 flush 操作，因此不需要再调用 fclose 之前调用 fflush。

## 修改读写指针位置 fseek()
fseek 函数是与 lseek 系统调用对应的文件流函数。它在文件流里为下一次读写操作指定位置。```c
#include <stdio.h>

int fseek(FILE *stream,long int offset,int whence);
// 成功时返回 0，失败时返回 -1 并设置 errno 指出错误。
```



## 在 Linux 下使用 ANSI标准 I/O 编写文件复制程序


