---
title: Linux 下的底层文件 I/O 函数
mathjax: false
date: 2020-12-31 19:46:02
updated: 2020-12-31 19:46:02
tags: [Linux, C, I/O]
categories: [技术杂谈]
toc: true
---

## 打开文件

首先介绍打开文件以及读写数据的函数。调用此函数时需传递两个参数：第一个参数是打开的目标文件名及路径信息，第二个参数是文件打开模式（文件特性信息）。
<!--more-->
:spider:

```cpp
#include <sys/type.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char* path, int flag);
// 成功时返回文件描述符，失败时返回 -1。
```
* path: 文件名的字符串地址。
* flag: 文件打开模式信息。

此函数第二个参数 `flag` 可能的常量值如下：

|打开模式|含义|
|-|-|
|O_CREAT|必要时创建文件|
|O_TRUNC|删除全部现有数据|
|O_APPEND|维持现有数据，保存到其后面|
|O_RDONLY|只读打开|
|O_WRONLY|只写打开|
|O_RDWR|读写打开|

如需传递多个参数，则应通过位或运算（OR）符组合传递。

## 关闭文件

使用文件后必须关闭。下面介绍关闭文件时调用的函数。

```cpp
#include <unistd.h>

int close(int fd);
// 成功时返回 0，失败时返回 -1.
```
* fd: 需要关闭的文件或者套接字的文件描述符。

若调用此函数的同时传递文件描述符参数，则关闭（终止）相应文件。另外需要注意的是，此函数不仅可以关闭文件，还可以关闭套接。因为 Linux 系统是不区分文件和套接字的。

## 将数据写入文件

接下来介绍的 `write` 函数用于向文件输出（传输）数据。当然，Linux 中不区分文件与套接字，因此，通过套接字向其他计算机传递数据时也会用到该函数。

```cpp
#include <unistd.h>

ssize_t write(int fd, const void* buf, size_t nbytes);
// 成功时返回写入的字节数，失败时返回 -1。
```

* fd: 数据传输对象的文件描述符。
* buf: 保存要传输数据的缓冲地址值。
* nbytes: 要传输数据的字节数。

此函数定义中，`size_t` 是通过 `typedef` 声明的 `unsigned int` 类型。
对 `ssize_t` 来说，`size_t` 前面多加的 `s` 代表 `signed`，即 `ssize_t` 是通过 `typedef` 声明的 `signed int` 类型。

-
下面通过示例帮助大家更好地理解前面讨论过的函数。此程序将创建新文件并保存数据。
```c
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>

void error_handling(char* message);

int main(void)
{
   int fd;
   char buf[] = "Let's go!\n";
   
   fd=open("data.txt", O_CREAT | O_WRONLY | O_TRUNC);
   if(fd == -1)
   {
      error_handling("open() error!");
   }
   printf("file descriptor: %d \n", fd);

   if(write(fd, buf, sizeof(buf)) == -1)
   {
      error_handling("write() error!");
   }
   close(fd);
   return 0;
}

void error_handling(char* message)
{
   fputs(message, stderr);
   fputc('\n', stderr);
   exit(1);
}
```

运行示例后，利用 Linux 的 `cat` 命令输出 data.txt 文件内容，可以确认确实已向文件传输数据。
```shell
[root@VM-8-13-centos c]# ./low_open
file descriptor: 3 
[root@VM-8-13-centos c]# cat data.txt
Let's go!
```


## 读取文件中的数据

与之前的 write 函数相对应， read 函数用来输入（接受）数据。
```c
#include <unistd.h>

ssize_t read(int fd, void* buf, size_t nbytes);
// 成功时返回接受的字节数（但遇到文件结尾则返回 0），失败时则返回 -1。
```
* fd: 数据接收对象的文件描述符。
* buf: 要保存接受数据的缓冲地址值。
* nbytes: 要接收数据的最大字节数。

-
下列示例将通过 `read` 函数读取 data.txt 中保存的数据。
```c
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>
#define BUF_SIZE 100

void error_handling(char* message);

int main(void)
{
   int fd;
   char buf[BUF_SIZE];

   fd = open("data.txt", O_RDONLY);
   if(fd == -1)
   {
      error_handling("open() error!");
   }
   printf("file descriptor: %d \n", fd);

   if(read(fd, buf, sizeof(buf)) == -1)
   {
      error_handling("read() error!");
   }
   printf("file data: %s", buf);
   close(fd);
   return 0;
}

void error_handling(char* message)
{
   fputs(message, stderr);
   fputc('\n', stderr);
   exit(1);
}
```

```shell
[root@VM-8-13-centos c]# ./low_read
file descriptor: 3 
file data: Let's go!
```

基于文件描述符的 I/O 操作相关介绍到此结束。希望大家记住，该内容同样适用于套接字。

## 文件描述符与套接字
下面将同时创建文件和套接字，并用整数型态比较返回的文件描述符值。
```c
#include<stdio.h>
#include<sys/socket.h>
#include<fcntl.h>
#include<unistd.h>

int main(void)
{
   int fd1, fd2, fd3;

   fd1 = socket(PF_INET,SOCK_STREAM, 0);
   fd2 = open("test.dat", O_CREAT | O_WRONLY | O_TRUNC);
   fd3 = socket(PF_INET, SOCK_DGRAM, 0);
  
   printf("file descriptor 1: %d\n", fd1);
   printf("file descriptor 2: %d\n", fd2);
   printf("file descriptor 3: %d\n", fd3);
   
   close(fd1);
   close(fd2);
   close(fd3);
   
   return 0;
}
```

```shell
[root@VM-8-13-centos c]# ./fd_seri
file descriptor 1: 3
file descriptor 2: 4
file descriptor 3: 5
```

从输出的文件描述符整数值可以看出，描述符从 3 开始由小到大顺序编号（numbering），因为 0、1、2 是分配给标准 I/0 的描述符，如下表所示。

|文件描述符|对象|
|-|-|
|0|标准输入：Standard Input|
|1|标准输出：Standard Output|
|2|标准错误：Standard Error|


## 在 Linux 下使用底层 I/O 编写文件复制程序

现在，我们根据的示例代码，举一反三，编写一个程序，将上面新建的 data.txt 文件复制，并在同目录粘贴为 data2.txt。

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

#define BUF_MAX 100

void error_handling(char* message);

int main(void)
{
    int fd1; // 待复制文件描述符
    int fd2; // 待粘贴文件描述符

    char* source_file = "data.txt";
    char* defination_file = "data2.txt";

    char buf[BUF_MAX];

    if ((fd1 = open(source_file, O_RDONLY)) == -1)
    {
        error_handling("open() fd1 error!");
    }

    if (read(fd1, buf, sizeof(BUF_MAX)) == -1)
    {
        error_handling("read() error!");
    }

    if ((fd2 = open(defination_file, O_CREAT | O_WRONLY | O_TRUNC)) == -1)
    {
        error_handling("open() fd2 error");
    }

    if (write(fd2, buf, sizeof(buf)) == -1)
    {
        error_handling("write() error!");
    }

    close(fd1);
    close(fd2);

    printf("已成功将文件 %s 复制到 %s.\n", source_file, defination_file);

    return 0;
}

void error_handling(char* message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```
```shell
[root@VM-8-13-centos c]# ./duplicate
已成功将文件 data.txt 复制到 data2.txt.
```