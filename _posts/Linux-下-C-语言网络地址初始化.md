---
title: Linux 下 C 语言网络地址初始化
mathjax: false
date: 2021-01-01 19:34:11
updated: 2021-01-01 19:34:11
tags: [Socket, TCP/IP, 网络编程]
categories: [网络编程]
toc: true
---


这里介绍套接字创建过程中常见的网络地址信息初始化方法。

```c
struct sockaddr_in addr;
char* serv_ip = "211.217.168.13";   // 声明 ID 地址字符串
char* serv_port = "9190";           // 声明端口号字符串
memset(&addr, 0, sizeof(addr));     // 结构体变量 addr 的所有成员初始化为 0
addr.sin_family = AF_INET;          // 指定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip);  // 基于字符串的 IP 地址初始化
addr.sin_port = htons(atoi(serv_port));     // 基于字符串的端口号初始化
```

```c
struct sockaddr_in addr;
char* serv_port = "9190";           // 声明端口号字符串
memset(&addr, 0, sizeof(addr));     // 结构体变量 addr 的所有成员初始化为 0
addr.sin_family = AF_INET;          // 指定地址族
addr.sin_addr.s_addr = inet_addr(INADDR_ANY);  // 基于字符串的 IP 地址，INADDR_ANY 会自动获得服务器地址
addr.sin_port = htons(atoi(serv_port));     // 基于字符串的端口号初始化
```


## 向套接字分配网络地址

bind 函数负责把初始化的地址信息分配给套接字。
```c
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr* myaddr, socklen_t addrlen);
// 成功时返回 0，失败时返回 -1。
```

* sockfd: 要分配地址信息（IP 地址和端口号）的套接字文件描述符
* myaddr: 存地址信息的结构体变量地址值。
* addrlen: 第二个结构体变量的长度。


如果此函数调用成功，则将第二个参数指定的地址信息分配给第一个参数中的相应个套接字。

下面给出服务器端常见套接字初始化过程：
```c
int serv_sock;
struct sockaddr_in serv_addr;
char* serv_port = "9190";

/* 创建服务器端套接字（监听套接字） */
serv_sock = socket(PF_INET, SOCK_STREAM, 0);

/* 地址信息初始化 */
memset(&serv_addr, 0. sizeof(serv_addr);
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
serv_addr.sin_port = htons(atoi(serv_port));

/* 分配地址信息 */
bind(serv_sock, (struct sockaddr*) &serv_addr, sizeof(serv_addr));

......
```

服务器端代码结构默认如上，当然还有未显示的异常处理代码。