---
title: 解决服务器 SSH 连接无响应断开时间过短的问题
date: 2021-09-05 20:55
categories: [技术杂谈]
tags: [SSH]
toc: true
---



使用 ssh 连接到阿里云服务器的时候，总是一会儿不动就被断开连接了，有点烦，这里解决一下这个问题。

先编辑 sshd_config 文件

```shell
vim /etc/ssh/sshd_config
```

找到下面两行

```
#ClientAliveInterval 0
#ClientAliveCountMax 3
```

去掉前面的注释符号 `#`，改成

```
ClientAliveInterval 60
ClientAliveCountMax 1440
```

* `ClientAliveInterval 60` 表示当客户端没有向服务端发送消息时，服务器端每 60 秒向客户端请求一次消息，客户端响应以保持连接。默认值为 0，即不服务器端不向客户端请求消息。

* `ClientAliveCountMax 1440` 表示服务器发出请求后客户端没有响应的次数达到 1440，就自动断开 ssh 连接。结合 ClientAliveInterval，刚好是 24 小时。

重启 sshd 服务，之后新 ssh 过来的连接就都生效了

```
service sshd restart
```



<!--more-->

