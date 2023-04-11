---
title: "Linux 中查看进程信息命令 ps 的常见用法"
date: 2022-12-06 02:33:00
updated: 2022-12-08 00:48:00
categories: [技术杂谈]
tags: [Linux]
---



`ps` 命令用于查看当前进程的快照信息，其功能很多，本文只介绍我个人常用的。完整手册见 [ps(1) — Linux manual page](https://man7.org/linux/man-pages/man1/ps.1.html)。





`ps` 命令通常需要配合选项使用，如果不加选项的话，只会打印出当前 `bash` 和 `ps` 本身的简略进程信息：

```shell
$ ps
    PID TTY          TIME CMD
2057074 pts/0    00:00:00 bash
2057151 pts/0    00:00:00 ps
```

不加选项的 `ps` 命令通常没什么意义，所以我们需要配合选项使用。



## 1. 参数的种类



与大多程序不同，`ps` 命令有三种选项种类：

1. Unix 选项，必须以前缀 `-` 开始，如 `ps -ef`。
2. BSD 选项，必须没有前，如 `ps aux`。
3. GNL 长选项，必须以前缀 `--` 开始，如 `ps --deselect`。



不同类型的选项可以自由组合，但使用不当可能会发生冲突。

有一些选项是同义的，功能完全相同，但 `ps` 的实现中为了兼容性而将这些同义的选项都保留了下来。

由于 ps 选项比较多，本文不打算逐一介绍，而是按场景介绍，讲述每个场景该怎么用（用哪些选项），为什么这么用（为什么是这些选项）。受限于我个人的使用经验，我没办法把全部场景都罗列出，但本篇文章会逐渐追加更新我遇到的新场景中的新用法。



## 2. 各种场景中的选项使用



下面给出的所有用法中，均只包含 `ps` 命令本身，而筛选命令（如 `grep`）是不包括的，你需要自己筛选。



我在 2.1 小节给出两个最常用的命令，即打印全部进程信息的两种方法，而后面小节都是在 2.1 小节上的选项扩展。多个扩展也可以重新组合，打印出新的内容。



### 2.1. 打印全部进程信息



打印全部进程信息有两种常用的方式，分别是：


* `ps -ef`：使用标准语法打印全部进程信息。
* `ps aux`：使用 BSD 语法打印全部进程信息。



两种方法都很常用，主要取决于你需求打印哪些信息，如果你需要的信息两者都有（如进程 ID，进程命令），那么使用哪个就看你心情。但如果你想打印的信息只有某一种有，那么就用那个。

通常于我自己而言，如果想同时查看进程的 CPU 和内存占用，就用 `ps aux`，如果想顺便看看某进程的父进程 ID，就用 `ps -ef`。除此之外，`ps -ef` 显示的信息更少，更简洁一些。



下面是几个常用命令使用的选项说明：



* `a`：取消 BSD 风格中仅打印所有者是自己的进程的限制。与选项 `x` 配合使用会打印出所有进程。
* `u`：以面向用户的格式打印。会额外打印出进程的拥有者、进程 CPU 和内存占用等信息。
* `x`：取消 BSD 风格中仅打印有 TTY 的进程限制。与选项 `a` 配合使用会打印出所有进程。
* `-e`：显示全部进程。
* `-f`：进程列表将以完整的格式打印。通常 `-f` 会和其他选项一起使用以打印额外的列，例如与 `-L` 使用，会同时打印出进程拥有的线程数（NLWP 列），与线程 ID（LWP 列）。
* `e`：在命令列后打印出此进程的环境变量。







### 2.2. 打印指定用户的进程

指定用户的选项为 `-u`（还有几个类似的但不常用），后接用户名。



这里给出两种方法，`<username>` 为指定的用户名：

* `ps u -u <username>`：相比查看所有进程的 `ps aux` 这里删除了 `ax`，因为与 `-u` 冲突。选项 `u` 也可以不写，但会缺少很多信息，看着不得劲（当然如果你确实不需要那些信息，就完全可以不写了）。
* `ps -f -u <username>`：相比查看所有进程的 `ps -ef` 这里删除了 `-e`，因为与 `-u` 冲突。





### 2.3. 打印指定 PID 的进程

指定用户的选项为 `-q` 或 `q` 后接进程 PID。

这里给出两种方法，`<pid>` 为指定的用 PID：

* `ps u -q <pid>`：相比查看所有进程的 `ps aux` 这里删除了 `ax`，因为与 `-q` 冲突。选项 `u` 也可以不写，但会缺少很多信息，看着不得劲（当然如果你确实不需要那些信息，就完全可以不写了）。
* `ps -f -q <pid>`：相比查看所有进程的 `ps -ef` 这里删除了 `-e`，因为与 `-q` 冲突。



### 2.3. 自定义打印的列



自定义打印列的选项为 `-o`（还有几个类似的但不常用），后接列名，多个列名用空格或逗号 `,` 隔开。

例如：

```shell
ps -eo user,pid,ppid,%cpu,%mem,start,cmd
```

这个命令自定了输出了用户名列(`user`)、进程 ID 列(`pid`)，父进程列(`ppid`)，CPU 占用(`%cpu`)，内存占用(`%mem`)，开始时间(`start`)，进程命令(`cmd`)。



这里可选的列名可太多了，我这里只列出了几个我认为常用的，全部列见 [STANDARD FORMAT SPECIFIERS](https://man7.org/linux/man-pages/man1/ps.1.html#STANDARD_FORMAT_SPECIFIERS)。



此选项还可以自定义列的名字。还是上面的命令，我们可以这样写：

```shell
ps -eo user=USR,pid=MyPID,ppid,%cpu,%mem,start,cmd
```

效果与之前一样，但 `user` 列名被改为了 `USR`，`pid` 列名被改为了 `MyPID`。

这里 `=` 后面可以为空。例如：



```shell
ps -eo user=USR,pid=MyPID,ppid=,%cpu=,%mem,start,cmd
```

此时，打印出的信息中，第一行列名将不显示 `ppid` 和 `%cpu` 的列名。

我们可以将所有列名设置为空，这样 `ps` 打印的列表中将没有表头：

```shell
ps -eo user=,pid=
```

这就只打印出了用户名和进程 ID，这种通常是在代码里用 shell 命令获取信息的时候，方便解析。





### 2.4. 根据命令名（可执行程序）查看指定进程信息



这个场景其实很常见，我们已知一个命令名，然后想根据这个命令名找到这个进程 ID 执行操作（比如杀掉），这在我们编写 shell 脚本的时候可能比较常用。



有了上面 2.3 的基础，我们就很简单了。



引入 `ps` 选项 `-C`：根据命令名查找进程信息。



这里的命令名指的是具体执行的程序名，举个例子：

```shell
UID          PID    PPID  C STIME TTY          TIME CMD
root     1962969       1  0 Nov16 ?        00:10:50 /usr/local/aegis/aegis_update/AliYunDunUpdate
```

在这里，`CMD` 列的值为 `/usr/local/aegis/aegis_update/AliYunDunUpdate`，那么命令名应当是最后的这个 `AliYunDunUpdate`，不包括前面的路径。



我们实践一下：

```shell
$ ps -fC "AliYunDunUpdate"
UID          PID    PPID  C STIME TTY          TIME CMD
root     1962969       1  0 Nov16 ?        00:10:50 /usr/local/aegis/aegis_update/AliYunDunUpdate
```

```shell
$ ps -fC sshd
UID          PID    PPID  C STIME TTY          TIME CMD
root        1041       1  0 Apr20 ?        00:00:30 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,aes128-gcm@openssh.com,aes128-ctr,aes128-cbc -oMACs=hmac-sha2-256-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-128-etm
root     2047293 2046353  0 Dec03 ?        00:00:00 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,aes128-gcm@openssh.com,aes128-ctr,aes128-cbc -oMACs=hmac-sha2-256-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-128-etm
root     2057071    1041  0 Dec06 ?        00:00:00 sshd: gukaifeng [priv]
gukaife+ 2057073 2057071  0 Dec06 ?        00:00:00 sshd: gukaifeng@pts/0
root     2057117    1041  0 Dec06 ?        00:00:00 sshd: gukaifeng [priv]
gukaife+ 2057119 2057117  0 Dec06 ?        00:00:00 sshd: gukaifeng@pts/1
```

假如我们要打印出所有 `sshd` 进程的 ID，并且不输出列名（在其他程序内读取时就省了去列头的操作），结合 2.3 的内容：

```shell
$ ps -C sshd -o pid=
   1041
2047293
2057071
2057073
2057117
2057119
```

完美搞定！注意这里不要加 `-f` 了，因为和 `-o` 冲突的。



\-



如果你的需求和上面的例子一样（即非常精确的指定命令可执行文件名，且只需要打印进程 ID），那么还有个 `pgrep` 命令更合适。  
这里 `pgrep` 的用法非常简单，只需要 `pgrep` 后接可执行文件名即可：

```shell
$ pgrep sshd
1041
2047293
2057071
2057073
2057117
2057119
```

`pgrep` 也有一些常用操作，不过不是本文重点，只简单提一下，感兴趣可以看 [pgrep(1) — Linux manual page](https://man7.org/linux/man-pages/man1/pgrep.1.html)。

### 2.5. 根据完整的执行命令行查看指定进程信息



上一节根据命令名（可执行文件）查找的方法可能不够精确。比如我们有多个由同一个可执行文件启动的进程，但启动时命令行的参数不同，我们只想杀掉其中某一个，而不影响由其他命令行参数启动的。这时候 2.4 中的方法就不适用了。



此场景下，只靠 `ps` 中提供的现成选项是不够的，如果使用 `grep` 和 `awk` 命令去从 `ps` 的输出中筛选，也确实很麻烦。

好在有个命令提供了更方便的方法，就是 `pgrep -f`。



`pgrep` 命令主要用于查找满足匹配条件的进程 ID，`-f` 参数可以理解为一个字符串匹配，匹配进程的执行命令（即便有一大长串也不怕）。如果你还需要进程 ID 以外的其他信息，那么和 `ps` 配合使用也非常简单。



下面举个例子：



假设我们如下两个进程，命令名字（可执行程序名）均为 `docker-proxy`。

```shell
$ ps -f -C "docker-proxy"
UID          PID    PPID  C STIME TTY          TIME CMD
root     2046312 2032496  0 Dec03 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 2022 -container-ip 172.17.0.2 -container-port 22
root     2046318 2032496  0 Dec03 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 2022 -container-ip 172.17.0.2 -container-port 22
```



我们举几个 `pgrep` 的命令用来筛选出这两个进程。



```shell
$ pgrep -f "docker-proxy"
2046312
2046318

$ pgrep -f "ker-prox"
2046312
2046318

$ pgrep -f "/usr/bin/docker-proxy -proto tcp -ho"
2046312
2046318
```

上述三个示例 `-f` 后接的都是两个进程命令中任意共同部分，可以看到，只要命令中含有我们 `-f` 指定的字符串，那么这个进程的 `id` 就会被打印出来。

想要精确打印进程，我们只需要使用此进程命令中独一无二的部分就可以了，例如上面两个进程的命令参数中 `-host-ip` 后不同，我们就可以使用这个特征来查找（当然你写的非常完整的一大长串也没啥问题）：

```shell
$ pgrep -f "\-host-ip 0.0.0.0"
2046312

$ pgrep -f "\-host-ip ::"
2046318
```

注意下这里查找字符串的第一个 `-` 需要转义一下，不然会被错误识别，后面的就不需要转义了。或者你干错直接从 `host-ip` 开始，不带前面的 `-` 避免转义，也没问题。





`pgrep` 更详细操作不是本文重点，感兴趣可以看 [pgrep(1) — Linux manual page](https://man7.org/linux/man-pages/man1/pgrep.1.html)。



