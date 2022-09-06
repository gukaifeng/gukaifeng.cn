---
title: Linux 中 nohup 命令的用法
date: 2022-09-07 00:37:00
updated: 2022-09-07 00:37:00
categories: [技术杂谈]
tags: [Linux]
toc: true
---



## 1. `nohup` 命令的作用和语法

Linux 中的每个命令都会在其执行时开始一个进程，并且此进程会在终端退出时自动终止。假设你正在通过 SSH 执行程序，如果连接断开，会话会被终止，所有正在执行的进程都会终止，然后你可能会面临一个巨大的意外风险。在这个例子中，在后台运行命令对用户来说就有用了，这便是 `nohup` 命令的使用场景。`nohup` (No Hang Up，即不悬挂) 是 Linux 系统命令，用于运行进程，即便用户已经从 shell 或终端登出了也不会终止。

通常，Linux 系统中的每个进程都会在关闭/退出终端后，收到一个 **SIGHUP**（挂起信号），这个信号代表终止该进程。`nohup` 命令会在关闭/退出 shell 或终端时组织进程收到此信号。如果一个任务如果使用 `nohup` 命令开始或执行，那么对用户来说标准输入 **stdin** 将不可用，标准输出 **stdout** 和标准错误 **stderr** 将默认输出到 **nohup.out** 文件（在执行命令的目录生成）中。也可以把 `nohup` 命令的输出重定向到其他文件，这样的话 **nohuo.out** 文件就不会有了。

`nohup` 命令语法：

```shell
nohup command [command-argument ...]
```

* 如果标准输入是终端，请将其从不可读的文件重定向。

* 如果标准输出是终端，如果可能，将输出追加到 **nohup.out**，否则为 **$HOME/nohup.out**。

* 如果标准错误是终端，请将其重定向到标准输出。

* 要将输出保存到文件，请使用 `nohup > FILE`。



## 2. 使用 `nohup` 命令

我们可以先用下面的命令检查一下 nohup 的版本，我这里是 8.30：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z ~]$ nohup --version
nohup (GNU coreutils) 8.30
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Jim Meyering.
```

> 注意：你的 shell 可能有自己的 `nohup` 版本，它通常会取代此处描述的版本。 请参阅你的 shell 的文档中有关它支持的选项的详细信息。    

### 2.1. 在前台运行命令

在前台运行命令并将命令的输出重定向到 **nohup.out** 文件中（如果没有专门指定重定向文件的话）。

我们看一个例子：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ nohup ping gukaifeng.cn
nohup: ignoring input and appending output to 'nohup.out'

```

我们使用 `nohup` 执行了一个 `ping` 进程，然后有提示输出告诉我们，输入被忽略了，输出被追加到 **nohup.out** 中。

我们停止这个进程，然后看看所谓的 **nohup.out**。

首先看到当前目录下确实有 **nohup.out** 这个文件。

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ ll nohup.out 
-rw------- 1 gukaifeng gukaifeng 7950 Sep  6 23:39 nohup.out
```

然后看看最后的 10 行内容：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ tail -10 nohup.out 
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=104 ttl=58 time=8.17 ms
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=105 ttl=58 time=8.22 ms
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=106 ttl=58 time=8.20 ms
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=107 ttl=58 time=8.22 ms
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=108 ttl=58 time=8.20 ms
64 bytes from 47.93.49.1 (47.93.49.1): icmp_seq=109 ttl=58 time=8.22 ms

--- gukaifeng.cn ping statistics ---
109 packets transmitted, 109 received, 0% packet loss, time 108029ms
rtt min/avg/max/mdev = 8.144/8.207/8.286/0.087 ms
```

这就是我们 `ping` 进程的输出。

如果想要重定向输出的话，和普通的重定向方法没有什么区别：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ nohup ping gukaifeng.cn > myoutput.txt
nohup: ignoring input and redirecting stderr to stdout
```

### 2.2. 搭配 `&` 在后台运行命令（最常用）



`nohup` 命令只是阻止了挂起信号，但并不会让应用在后台执行。

最常见的场景是 `nohup` 和 `&` 搭配使用（`&` 加在命令行末，表示该命令在后台执行）。由于我们大多时候使用 ssh 登录 Linux，也经常在后台执行一些程序，但当我们的 ssh 终端断开时，我们运行的所有进程（包括前台和后台进程）都会终止。在很多时候，我们不希望一个进程终止，`nohup` 和 `&` 的搭配就可以做到这一点。

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ nohup ping gukaifeng.cn &
[1] 1792120
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ nohup: ignoring input and appending output to 'nohup.out'

[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$
```

> 这里有一点比较奇怪，就是上面会在第 3 行命令位置输出信息，然后在第 4 行卡住，按一下回车键就好了，问题不大！

在后台的进程也可以通过 `fg` 命令转到前台运行。

在后台运行最常见的情况应该不是使用默认的重定向输出文件，我们最好根据实际需求，将后台运行的命令的输出重定向到一个合理的位置，写法就是这样：

```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z shell]$ nohup ping gukaifeng.cn > myoutput.txt 2>&1 &
[1] 1792257
```

上面的  `nohup ping gukaifeng.cn > myoutput.txt 2>&1 &`：

* 使用 `nohup` 和 `&` 后台执行命令 `ping gukaifeng.cn`。

* 将 `nohup` 的标准输出 **stdout** 和标准错误 **stderr** 都重定向到 **myoutput.txt** 文件中。

这应该是最常用的方法了。
