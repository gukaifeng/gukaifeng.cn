---
title: 使用 mmap 函数的用户级内存映射
date: 2022-07-31 23:22:00
updated: 2022-07-31 23:22:00
categories: [技术杂谈]
tags: [Linux,CentOS,mmap]
toc: true
---





## 1. mmap




Linux 进程可以使用 mmap 函数来创建新的虚拟内存区域，并将对象映射到这些区域中。

```c
#include <unistd.h>
#include <sys/mman.h>


void *mmap(void *start, size_t length, int port, int flags,
           int fd, off_t offset);

// 返回：若成功时则为指向映射区域的指针，若出错则为 MAP_FAILED(-1)。
```

## 



## 2. munmap


