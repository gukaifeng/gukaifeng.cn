---
title: "Linux 中查看网络连接信息命令 netstat 的常见用法"
date: 2022-12-08 01:02:00
updated: 2022-12-08 02:49:00
categories: [技术杂谈]
tags: [Linux]
---



## 1. 命令 `netstat` 的作用



`netstat` 命令用于打印网络连接、路由表、接口统计、伪装连接和多播成员信息。



因为我本人不是专业搞网络的，我最常用的就是用 `netstat` 打印网络信息，主要是看看 IP、端口什么的，验证一些程序的网络是否正常等。所以本文要给出的可能也是 `netstat` 最泛用的几个方法（选项），不管是不是专业搞网络的人应该都在某些场景里会用得上。



本文主要参考自 [netstat(8) — Linux manual page](https://man7.org/linux/man-pages/man8/netstat.8.html) 中的**查看网络连接部分**，其他部分感兴趣可以自己看看。







## 2. `netstat` 查看网络连接





`netstat` 查看网络连接信息的命令如下：

```shell
netstat [address_family_options]
```

其实就是命令 `netstat` 命令后接地址族选项。默认情况下，`netstat` 显示打开的套接字列表。如果我们不指定任何地址族，那么将打印所有已配置地址族的活动套接字。



与网络连接信息相关的地址族选项有：

* `-t` `--tcp`：打印使用 TCP 协议网络连接信息。
* `-u` `--udp`：打印使用 UDP 协议网络连接信息。
* `-U` `--udplite`：打印使用 UPD-Lite 协议网络连接信息。
* `-S` `--sctp`：打印使用 SCTP 协议的网络连接信息。
* `-w` `--raw`：打印使用 RAW 协议网络连接信息。
* `-2` `--l2cap`：打印活跃的蓝牙(Bluetooth)连接信息。
* `-f` `--rfcomm`：打印使用 RFCOMM 协议网络连接信息。
* `-l` `--listening`：仅显示正在监听的 sockets（默认情况下省略了这些）。
* `-a` `--all`：现在所有 sockets（正在监听的和没有监听的都算）。
* `-n` `--numeric`：用数字显示主机、端口和用户名信息。如果不加此选项的话，可能会有一个地址信息为 `iZ8vbf7xcuoq7ug1e7h:ssh`，`:` 前面是主机名，`:` 后面是协议。而如果你加了 `-n` 参数，则上述信息显示的可能是 `172.26.13.181:22`，`:` 前面是主机的 IP 地址，`:` 后面是使用的端口。
* `--numeric-hosts`：仅用数字显示主机信息，不影响端口和用户名。即是 `-n` 选项功能的一部分。
* `--numeric-ports`：仅用数字显示端口信息，不影响主机和用户名。即是 `-n` 选项功能的一部分。
* `--numeric-users`：仅用数字显示用户名信息，不影响主机和端口。即是 `-n` 选项功能的一部分。
* `-N` `--symbolic`：
* `-e` `--extend`：打印更详细的信息。此选项可以使用两次，使用两次时打印最全的信息。
* `-o` `--timers`：包括与网络计时器有关的信息。
* `-p` `--program`：包括每个 socket 归属的进程 ID 和进程名字。
* `-v` `--verbose`：打印更详细（冗长）的信息，尤其会包括一些关于未配置地址族的有用的信息。
* `-c` `--continuous`：持续打印网络连接信息，每秒打印一次。
* `-W` `--wide`：不截断地址长度。不加此参数的话，有些 IP 或者域名太长，就显示不全。



\-





我自己最常用的场景是用来查看某些服务是否正确启动了，例如某些 web 服务，或某些分布式服务。这些服务往往使用 TCP 协议，并且持续监听某个端口。所以在此场景中，我个人最常用的组合应该是 `-nltp`，既可以以数字的形式清晰的看到地址和端口信息，又能看到进程 ID 和名字。



在上面的场景中，你也可以自己修改，比如你关注的服务使用的是 UDP 协议，那就换成 `-nlup`。比如你发现打印的 IP 或主机名很长，被阶段了，则可以再加上 `-W` 选项。



这里就不再多说了。





## 3. 打印的网络连接信息中各个值的含义

我们以通常最常用的 `netstat -nltp` 为例：



```shell
$ netstat -nltp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:20048           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:43569         0.0.0.0:*               LISTEN      1622868/node        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:10808         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:10809         0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:34203           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33693         0.0.0.0:*               LISTEN      1638976/node        
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:2022            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:56525           0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::111                  :::*                    LISTEN      -                   
tcp6       0      0 :::20048                :::*                    LISTEN      -                   
tcp6       0      0 :::40445                :::*                    LISTEN      -                   
tcp6       0      0 :::2049                 :::*                    LISTEN      -                   
tcp6       0      0 :::36419                :::*                    LISTEN      -                   
tcp6       0      0 :::33060                :::*                    LISTEN      -                   
tcp6       0      0 :::2022                 :::*                    LISTEN      -                   
tcp6       0      0 :::3306                 :::*                    LISTEN      -
```

此命令前面的打印信息中提示我们这里显示的可能不全，有些进程连接需要 root 用户才能看到。这里我们暂时先不关注这里。

在 `-nltp` 选项下，有打印出 7列，我们下面逐个解释含义（部分值的理解需要计算机网络基础，如 TCP 建立连接过程等，这里不解释了）：



* `Proto`：此连接使用的协议。如 `tcp`、`tcp6`、`udp` 等。
* `Recv-Q`：需分情况考虑：
  * `State` 为 `Established`：连接到此 socket 的用户程序未复制的字节数。
  * `State` 为 `LISTEN`：自内核 2.6.18 版本后，此项表示当前半连接队伍的长度。
* `Send-Q`：需分情况考虑：
  * `State` 为 `Established`：socket 远端还没有确认的字节数。
  * `State` 为 `LISTEN`：自内核 2.6.18 版本后，此项表示半连接队列的最大长度。
* `Local Address`：socket 本地端的地址和端口号。这里冒号 `:` 前的无论是 `0.0.0.0` 还是 `127` 开头的环回地址，还是 IPv6 地址 `::`（参考 IPv6 地址的缩写规则），指的都是本机。冒号 `:` 指的是该连接使用的本地端口。
* `Foreign Address`：socket 远端的地址和端口号。格式和 `Local Address` 类似。`0.0.0.0:*` 和 IPv6 的 `:::*` 都表示没有远端。因为我这里打印的都是监听端口，都是在等待有连接到达的，所以目前没有远端。
* `State`：连接状态。有如下可能的值：
  * `UNKNOWN`：此 socket 的状态未知。
  * `ESTABLISHED` 表示已建立连接。
  * `LISTEN`：表示正在监听将要到来的连接。
  * `SYN_SENT`：该 socket 正在积极尝试建立连接。
  * `SYN_RECV`：已经收到了一个网络中的连接请求。
  * `FIN_WAIT1`：此 socket 已关闭，此连接正在关闭中。
  * `FIN_WAIT2`：连接已经关闭，socket 正在等待来自远端的关闭信息。
  * `TIME_WAIT`：socket 在关闭后等待处理仍在网络中的数据包。
  * `CLOSE  `：socket 没有被使用。
  * `CLOSE_WAIT`：远端已经关闭，正在等待 socket 关闭。
  * `LAST_ACK`：远端已经关闭，此 socket 已经关闭，正在等待最后的确认。
  * `CLOSING`：两端 socket 都已经关闭，但是我们还没有发完全部的数据。
* `PID/Program name`：此 socket 归属的进程 ID 和进程名字。

