

此篇博客将在 Linux 下实现完整的 TCP 服务器端，在此过程中大家将理解套接字使用方法及数据传输方法。
<!--more-->
## TCP 服务器端

### TCP 服务器端的默认函数调用顺序

下图给出了 TCP 服务器端默认的函数调用顺序，绝大部分 TCP 服务器端都按照该顺序调用。
![](https://gukaifeng.cn/posts/shi-xian-ji-yu-tcp-de-fu-wu-qi-duan-ke-hu-duan/%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E-TCP-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF-%E5%AE%A2%E6%88%B7%E7%AB%AF_1.png)

调用 socket 函数创建套接字，声明并初始化地址信息结构体变量，调用 bind 函数向套接字分配地址。这 2 个阶段之前都已讨论过（参见[Linux 下 C 语言网络地址初始化](https://gukaifeng.me/2021/01/01/Linux-%E4%B8%8B-C-%E8%AF%AD%E8%A8%80%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E5%88%9D%E5%A7%8B%E5%8C%96/)），下面讲解之后的几个过程。


### 进入等待连接请求状态

我们已调用 bind 函数给套接字分配了地址，接下来就要通过调用 listen 函数进入等待连接请求状态。只有调用了 listen 函数，客户端才能进入可发出链接请求的状态。换言之，这时客户端才能调用 connect 函数（若提前调用将发生错误）。

```c
#include <sys/socket.h>

int listen(int sock, int backlog);
// 成功时返回 0，失败时返回 -1。
```

* sock: 希望进入等待连接请求状态的套接字文件描述符，传递的描述符套接字参数成为服务器端套接字（监听套接字）。
* backlog: 连接请求等待队列（Queue）的长度，若为 5，则队列长度为 5，表示最多使 5 个连接请求进入队列。

先解释一下等待连接请求状态的含义和连接请求等待队列。“服务器端处于等待连接请求状态”是指，客户端请求连接时，受理连接前一直使请求处于等待状态。下图给出了这个过程。
![](https://gukaifeng.cn/posts/shi-xian-ji-yu-tcp-de-fu-wu-qi-duan-ke-hu-duan/%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E-TCP-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF-%E5%AE%A2%E6%88%B7%E7%AB%AF_2.png)

由上图可知作为 listen 函数的第一个参数传递的文件描述符套接字的用途。客户端请求连接本身也是从网络中接收到的一种数据，而想要接收就需要套接字。此任务就由服务器端套接字完成。服务器端套接字是请求接收连接请求的一名门卫或一扇门。

客户端如果向服务器端询问：“请问我是否可以发起连接？” 服务器端套接字就会亲切应答：“您好！当然可以，但系统正忙，请到等候室排号等待，准备好后会立即受理您的连接。” 同时将连接请求请到等候室。调用 listen 函数即可生成这种门卫（服务器端套接字），listen 函数的第二个参数决定了等候室的大小。等候室称为连接请求等待队列，准备好服务器端套接字和连接请求等待队列后，这种可接收连接请求的状态称为等待连接请求状态。

listen 函数的第二个参数值与服务器端的特性有关，像频繁接受请求的 Web 服务器端至少应为 15。另外，连接请求队列的大小始终根据实验结果而定。

### 受理客户端连接请求

调用 listen 函数后，若有新的连接请求，则应按序受理。受理请求意味着进入可接收数据状态。也许大家已经猜到进入这种状态所需部件——当然是套接字！大家可能认为可以使用服务器端套接字，但服务器端套接字是做门卫的。如果在与客户端的数据交换中使用门卫，那谁来守门呢？因此需要另外一个套接字，但没必要亲自创建。下面这个函数将自动创建套接字，并连接到发起请求的客户端。

```c
#include <sys/socket.h>

int accept(int sock, struct sockaddr* addr, socklen_t* addrlen);
// 成功时返回创建的套接字文件描述符，失败时返回 -1。
```

* sock: 服务器套接字的文件描述符。
* addr: 保存发起连接请求的客户端地址信息的变量地址值，调用函数后向传递来的地址变量参数填充客户端地址信息。
* addrlen: 第二个参数 addr 结构体的长度，但是是存有长度的变量地址。函数调用完成后，该变量即被填入客户端地址长度。

accept 函数受理连接请求等待队列中待处理的客户端连接请求。函数调用成功时，accept 函数内部将产生用于数据 I/O 的套接字，并返回其文件描述符。需要强调的是，套接字是自动创建的，并自动与发起连接请求的客户端建立连接。下图展示了 accept 函数调用过程。

![](https://gukaifeng.cn/posts/shi-xian-ji-yu-tcp-de-fu-wu-qi-duan-ke-hu-duan/%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E-TCP-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF-%E5%AE%A2%E6%88%B7%E7%AB%AF_3.png)

上图展示了“从等待队列中取出 1 个链接请求，创建套接字并完成连接请求”的过程。服务器单独创建的套接字与客户端建立连接后进行数据交换。

### TCP 服务器端的代码实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

void error_handling(char *message);

int main(int argc, char *argv[])
{
   int serv_sock;
   int clnt_sock;

   struct sockaddr_in serv_addr;
   struct sockaddr_in clnt_addr;
   socklen_t clnt_addr_size;

   char message[] = "Hello World";

   if (argc != 2)
   {
      printf("Usag : %s <port>\n", argv[0]);
      exit(1);
   }

   serv_sock = socket(PF_INET, SOCK_STREAM, 0);
   if (serv_sock == -1)
   {
      error_handling("socket() error");
   }

   memset(&serv_addr, 0, sizeof(serv_addr));
   serv_addr.sin_family = AF_INET;
   serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
   serv_addr.sin_port = htons(atoi(argv[1]));

   if (bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)
   {
      error_handling("bind() error");
   }

   if (listen(serv_sock, 5) == -1)
   {
      error_handling("listen() error");
   }

   clnt_addr_size = sizeof(clnt_addr);
   clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_addr, &clnt_addr_size);

   if (clnt_sock == -1)
   {
      error_handling("accpt() error");
   }

   write(clnt_sock, message, sizeof(message));
   close(clnt_sock);
   close(serv_sock);
   return 0;
}

void error_handling(char *message)
{
   fputs(message, stderr);
   fputc('\n', stderr);
   exit(1);
}
```


## TCP 客户端

### TCP 客户端的默认函数调用顺序

接下来讲解客户端的实现顺序。这要比服务器端简单许多。因为创建套接字的请求连接就是客户端的全部内容。
TCP 客户端函数调用顺序如下图。
![](https://gukaifeng.cn/posts/shi-xian-ji-yu-tcp-de-fu-wu-qi-duan-ke-hu-duan/%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E-TCP-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF-%E5%AE%A2%E6%88%B7%E7%AB%AF_4.png)

与服务器端相比，区别就在于“请求连接”，它是创建客户端套接字后向服务器端发起的连接请求。服务器端调用 listen 函数后创建连接请求等待队列，之后客户端即可请求连接。那如何发起连接请求呢？通过调用如下函数完成。

```c
#include <sys/socket.h>

int connect(int sock, struct sockaddr* servaddr, socklen_t addrlen);
// 成功时返回 0，失败时返回 -1。
```
* sock: 客户端套接字文件描述符。
* servaddr: 保存目标服务器端地址信息的变量地址值。
* addrlen: 以字节为单位传递已传递给第二个结构体参数 servaddr 的地址变量参数。

客户端调用 connect 函数后，发生以下情况之一才会返回（完成函数调用）。
1. 服务器端接收连接请求。
2. 发生断网等异常情况而中断连接请求。

需要注意，所谓的“接收连接”并不意味着服务器端调用 accept 函数，其实是服务器端把连接请求信息记录到等待队列。因此 connect 函数返回后并不立即进行数据交换。

--

* Tip: 客户端套接字的地址信息在哪？
实现服务器端必经过程之一就是给套接字分配 IP 和端口号。但客户端实现过程中并未出现套接字地址分配，而是创建套接字后立即调用 connect 函数。难道客户端套接字无需分配 IP 和端口？当然不是，网络数据交换必须分配 IP 和端口。既然如此，那客户端套接字何时、何地、如何分配地址呢？

    1. 何时？调用 connect 函数时。
    2. 何地？操作系统，更准确地说是在内核中。
    3. 如何？IP 用计算机（主机）的 IP，端口随机。


客户端的 IP 地址和端口在调用 connect 函数时自动分配，无需调用标记的 bind 函数进行分配。

### TCP 客户端的代码实现
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

void error_handling(char *message);

int main(int argc, char *argv[])
{
   int sock;
   struct sockaddr_in serv_addr;
   char message[30];
   int str_len;
   if (argc != 3)
   {
      printf("Usag : %s <IP> <port>\n", argv[0]);
      exit(1);
   }

   sock = socket(PF_INET, SOCK_STREAM, 0);
   if (sock == -1)
   {
      error_handling("socket() error");
   }

   memset(&serv_addr, 0, sizeof(serv_addr));
   serv_addr.sin_family = AF_INET;
   serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
   serv_addr.sin_port = htons(atoi(argv[2]));

   if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) == -1)
   {
      error_handling("connect() error");
   }

   str_len = read(sock, message, sizeof(message) - 1);
   if (str_len == -1)
   {
      error_handling("read() error");
   }

   printf("Message from server : %s \n", message);
   close(sock);
   return 0;
}

void error_handling(char *message)
{
   fputs(message, stderr);
   fputc('\n', stderr);
   exit(1);
}
```

## 基于 TCP 服务器端/客户端函数调用关系

前面讲解了 TCP 服务器端/客户端的实现顺序，实际上二者并非相互独立，大家应该可以勾勒出他们之间的交互过程，如下图所示。
![](https://gukaifeng.cn/posts/shi-xian-ji-yu-tcp-de-fu-wu-qi-duan-ke-hu-duan/%E5%AE%9E%E7%8E%B0%E5%9F%BA%E4%BA%8E-TCP-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF-%E5%AE%A2%E6%88%B7%E7%AB%AF_5.png)

上图的总结流程如下：服务器端创建套接字后连续调用 bind、listen 函数进入等待状态，客户端通过调用 connect 函数发起连接请求。需要注意的是，客户端只能等到服务器调用 listen 函数后才能调用 connect 函数。同时要清楚，客户端调用 connect 函数前，服务端有可能率先调用 accept 函数。当然，此时服务器端在调用 accept 函数是进入阻塞（blocking）状态，直到客户端调用 connect 函数为止。