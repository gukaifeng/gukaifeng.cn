---
title: 基于 TCP/IP 的服务器/客户端实现数值运算
mathjax: false
date: 2021-01-02 15:32:51
updated: 2021-01-02 15:32:51
tags: [Socket, TCP/IP, 网络编程]
categories: [网络编程]
toc: true
---


下面编写程序以体验应用层协议的定义过程。该程序中，服务器端从客户端获得多个数字和运算符信息。服务器端收到数字后对其进行加减乘运算，然后把结果传回客户端。例如，向服务器端传递3、5、9的同时请求加法运算，则客户端收到3+5+9的运算结果；若请求做乘法运算，则客户端收到3x5x9的运算结果。而如果向服务器端传递4、3、2的同时要求做减法，则客户端将收到4-3-2的运算结果，即第一个参数成为被减数。

下面的代码与前面的服务器/客户端编程代码总体是一样的。**区别在于对发送、接收数据的处理。**

## 服务器端代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/socket.h>
#include <string.h>
#include <arpa/inet.h>

#define BUF_SIZE 1024
#define NUM_SIZE 4

void error_handling(char *message);
int cal(int nun_cnt, int nums[], char op);

int main(int argc, char *argv[])
{
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    socklen_t clnt_adr_len;
    char message[BUF_SIZE];

    if (argc != 2)
    {
        printf("Usage: %s <port>\n", argv[0]);
        exit(1);
    }

    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    if (serv_sock == -1)
    {
        error_handling("scoket() error!");
    }

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
    {
        error_handling("bind() error!");
    }

    if (listen(serv_sock, 5) == -1)
    {
        error_handling("listen() error!");
    }

    clnt_adr_len = sizeof(clnt_adr);
    clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, &clnt_adr_len);
    if (clnt_sock == -1)
    {
        error_handling("accept() error!");
    }

    // 读取b套接字中的数据
    if (read(clnt_sock, &message, NUM_SIZE) == -1)
    {
        error_handling("read() error!");
    }

    int num_cnt = *((int *)&message);
    int str_len = (1 + num_cnt) * NUM_SIZE + 1;
    int recv_len = 0, recv_cnt;

    while (recv_len + NUM_SIZE < str_len)
    {
        recv_cnt = read(clnt_sock, &message[NUM_SIZE * 1], BUF_SIZE - NUM_SIZE);
        if (recv_cnt == -1)
        {
            error_handling("read() error!");
        }
        recv_len += recv_cnt;
    }
    int result = cal(num_cnt, (int *)&message[1 * NUM_SIZE], *(char *)&message[(num_cnt + 1) * NUM_SIZE]);

    // 写入数据
    if (write(clnt_sock, &result, sizeof(result)) == -1)
    {
        error_handling("write() error!");
    }

    close(serv_sock);
    close(clnt_sock);

    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}

int cal(int num_cnt, int nums[], char op)
{
    int result = nums[0];
    int i;

    for (i = 1; i < num_cnt; ++i)
    {
        switch (op)
        {
        case '+':
            result += nums[i];
            break;

        case '-':
            result -= nums[i];
            break;

        case '*':
            result *= nums[i];
            break;

        case '/':
            result /= nums[i];
            break;
        }
    }
    return result;
}
```

## 客户端代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/socket.h>
#include <string.h>
#include <arpa/inet.h>

#define BUF_SIZE 1024
#define NUM_SIZE 4

void error_handling(char *message);

int main(int argc, char *argv[])
{
    int clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    char message[BUF_SIZE];
    int num_count;
    int i, result;

    if (argc != 3)
    {
        printf("Usage: %s <ip> <port>\n", argv[0]);
        exit(1);
    }

    clnt_sock = socket(PF_INET, SOCK_STREAM, 0);

    memset(&serv_adr, 0, sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htons(atoi(argv[2]));

    if (connect(clnt_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
    {
        error_handling("connect() error!");
    }

    // 往套接字写数据
    fputs("请输入操作数的数量：", stdout);
    scanf("%d", &message);
    for (i = 0; i < *(int *)&message; ++i)
    {
        printf("请输入第 %d 个操作数：", i + 1);
        scanf("%d", &message[(i + 1) * NUM_SIZE]);
    }
    fputs("请输入操作符：", stdout);
    getchar();
    scanf("%c", &message[((*(int *)&message) + 1) * NUM_SIZE]);

    if (write(clnt_sock, message, sizeof(message)) == -1)
    {
        error_handling("write() error!");
    }

    // 读取数据
    if (read(clnt_sock, &result, NUM_SIZE) == -1)
    {
        error_handling("read() error!");
    }

    close(clnt_sock);

    printf("the result from server is: %d.\n", result);

    return 0;
}

void error_handling(char *message)
{
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

## 示例输入输出

* 服务器代码运行后如下：

```shell
[root@VM-8-13-centos c]# ./op_server 1233

```
光标会停留在第二个行，处于监听中。

* 客户端代码运行后如下：

```shell
[root@VM-8-13-centos c]# ./op_client 127.0.0.1 1233
请输入操作数的数量：3
请输入第 1 个操作数：11
请输入第 2 个操作数：22
请输入第 3 个操作数：33
请输入操作符：*
the result from server is: 7986.
```