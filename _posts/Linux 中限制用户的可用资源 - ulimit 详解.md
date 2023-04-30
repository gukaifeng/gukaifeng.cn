---
title: "Linux 中限制用户的可用资源 - ulimit 详解"
date: 2023-04-30 15:51:00
updated: 2023-05-01 00:17:00
categories: [技术杂谈]
tags: [Linux]
---



## 1. `ulimit` 命令的作用



`ulimit` 命令是系统自带的，用于修改 Shell 的资源限制。



具体来说，`ulimit` 命令用于限制一个 Shell 以及由这个 Shell 创建的所有进程的可用资源总量。



注意是限制一个 Shell，不是一个用户。比如你 SSH 登录到服务器，你当前所处的就是一个 Shell，退出后这个限制就失效了。







## 2. `ulimit` 命令的用法



`ulimit` 命令的用法如下：

```shell
ulimit [Options] [limit]
```

* `options`：选项。除了 `-S`、`-H`、`-a`，每个选项代表一个资源，具体在下面说。
* `limit`：指定资源的限制值。



\-



选项 `options` 可取值如下：



有三个选项是通用的：

* `-S`：使用软资源限制，允许资源超出，资源超出后仅警告。
* `-H`：使用硬资源限制，资源严格控制，不允许超出。
* `-a`：列出当前所有的资源限制。

其他选项，每个代表一种系统资源：

* `-b`：Socket 的 buffer 大小。
* `-c`：创建的核心文件(core files)的最大大小
* `-d`：进程的数据段(data segment)最大大小。
* `-e`：最大调度优先级（即命令 `nice` 输出的值）。
* `-f`：Shell 及其子进程写入的文件的最大大小。
* `-i`：挂起信号(signals)的最大数量。
* `-k`：为此进程分配的最大 kqueue 数。
* `-l`：进程可以锁定到内存中的最大大小。
* `-m`：驻留集(resident set)的最大大小。
* `-n`：打开文件描述符的最大数量。
* `-p`：管道的 buffer 大小。
* `-q`：POSIX 消息队列中的最大字节数。
* `-r`：最大实时调度优先级。
* `-s`：最大栈大小。
* `-t`：最大 CPU 时间，以秒为单位。
* `-u`：最大用户进程数。
* `-v`：虚拟内存大小。
* `-x`：文件锁的最大数量。
* `-P`：伪终端(pseudoterminals)的最大数量
* `-T`：线程的最大数量。



上述资源限制中，涉及“大小”的，除了管道 buffer 大小（`-p`）以 512 字节为增量，其他均以 1024 字节为增量。



注意 `ulimit` 对上述选项列出的各种资源的限制，**并不是在所有平台都有效的**，得平台和操作系统支持才可以。











## 3. 查看资源限制



我们可以通过 `ulimit -a` 来显示当前所有的资源限制：

```shell
$ ulimit -a
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 30931
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65535
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 30931
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

可以看出，如果有限制，则会显示限制的值。如果没有限制，则会显示 `unlimited`。



`ulimit` 默认显示的是软限制，即 `ulimit -a` 等价于 `ulimit -Sa`，我们可以使用 `ulimit -Ha` 指定查看硬限制。





\-



我们可以直接加参数（不加限制值），来显示某一个资源限制情况。

例如，我们查看打开文件描述符的最大数量（即 open files，`-n`）:

```shell
$ ulimit -n  # equal to `ulimit -Sn`
65535
```

也可以指定查看软硬限制，注意 `S` 或 `H` 要在 `n` 的前面：

```shell
$ ulimit -Sn
65535
```

```shell
$ ulimit -Hn
65535
```

我这里的软硬限制值是一样的。



\-





`ulimit` 命令如果不接参数，则等价于 `ulimit -f`，即 `-f` 是默认选项：



```shell
$ ulimit  # equal to `ulimit -f`
unlimited
```





## 4. 修改资源限制







### 4.1. 仅对当前 Shell 临时修改



```shell
ulimit [Options] [limit]
```

前文演示的 `ulimit` 用法并没有接最后这个 limit 值，接上这个限制值，就可以修改对应资源的限制情况了。





参数 `-S` 表示设置软限制，参数 `-H` 表示设置硬限制，当都不指定时，表示设置软限制和硬限制为指定值。



这里有几点需要注意：

* 软限制的值不可以超过硬限制，即便设置也不会生效，软限制值必须小于等于硬限制值。
* 普通用户下，硬限制只能调低，无法调高（使用 `sudo` 也不行）。只有 root 用户（或普通用户 `sudo su` 切换到 root 用户）可以增大硬限制。



\-



**我这里以调整打开文件描述符的最大数量（即 open files，`-n`）为例。**



首先查看下当前的限制值：



```shell
$ ulimit -n
65535
$ ulimit -Hn
65535
```

可以看到软硬限制都是 65535。



\-



先单独调整一下软限制：

```
ulimit -Sn 8192
```

然后查看下新的限制值：

```shell
$ ulimit -n
8192
$ ulimit -Hn
65535
```

可以看到软限制已经调整好了，硬限制没有变，符合预期。



\-



单独调整硬限制：

```shell
ulimit -Hn 16384
```

然后查看下新的限制值：

```shell
$ ulimit -n
8192
$ ulimit -Hn
16384
```

\-

最后再试试同时调整软硬限制：



```shell
ulimit -n 4096
```

然后查看下新的限制值：

```shell
$ ulimit -n
4096
$ ulimit -Hn
4096
```

可以看到软硬限制都同时修改了，符合预期。





\-



通过 `ulimit` 命令修改的资源限制，仅对当前 Shell 有效，要想永久修改这些限制，就需要下面的方法。

### 4.2. 永久修改：使用可插拔认证模块（PAM）



>其实有一个投机的做法 ~
>
>我们知道每次登录服务器的时候，都会执行一些配置脚本文件，例如登录 Bash 时的 `~/.bashrc`。
>
>我们其实可以在里面写上像刚刚临时修改资源限制时的那些命令，也可以达到预期的“永久修改”效果。
>
>不过这个方法不是我们重点关注的，也比较简单，就不展开说了。



**这里同样以调整打开文件描述符的最大数量（即 open files，`-n`）为例。**

#### 4.2.1. 什么是 PAM





先说一下什么是 PAM。



[PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module)，可插拔认证模块(Pluggable Authentication Modules)，是由Sun提出的一种认证机制，通过提供一些动态链接库和一套统一的 API，将系统提供的服务和该服务的认证方式分开，使得系统管理员可以灵活地根据需要给不同的服务配置不同的认证方式而无需更改服务程序，同时也便于向系统中添加新的认证手段。



我们可以通过 PAM 来管理对系统资源的限制。





#### 4.2.2. 修改 PAM 限制系统资源的配置文件





首先我们先编辑 PAM 的关于系统资源限制的配置文件（需要 root 用户或 `sudo` 权限）：



```shell
vim /etc/security/limits.conf
```

其默认内容像下面这样：

```shell
# /etc/security/limits.conf
#
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
#
#Also note that configuration files in /etc/security/limits.d directory,
#which are read in alphabetical order, override the settings in this
#file in case the domain is the same or more specific.
#That means for example that setting a limit for wildcard domain here
#can be overriden with a wildcard setting in a config file in the
#subdirectory, but a user specific setting here can be overriden only
#with a user specific setting in the subdirectory.
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

# End of file
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```

此文件中，每一行是一个资源限制配置项，格式如下：

```shell
<domain>        <type>  <item>  <value>
```

具体用法在文件内的注释里有些，这里简单翻译一下。



* `domain`：设定资源限制的目标。可以是用户名、组名或通配符。注意通配符 `*` 不包含 root 用户。
* `type`：资源限制的类型。可以有三个值，`soft`、`hard` 或 `-`，分别表示设定软限制、设定硬限制、同时设定软限制和硬限制。
* `item`：要限制的资源。注意看上面的注释，这里的资源项的名字和 `ulimit` 里的不一样，例如这里的 `nofile` 对应 `ulimit -n`。
* `value`：资源的限制值。





\-



例如，我这里将上面文件最后的对 nofile 的限制，修改如下：

```shell
* soft nofile 1024
* hard nofile 8192
```





#### 4.2.3. 启动 PAM 对系统资源的限制



我们上面只是编辑好了 PAM 关于对系统资源限制的配置文件，我们还需要启用 PAM 对系统资源的限制功能。



我们编辑 PAM 的启动项配置（需要 root 用户或 `sudo` 权限）：

```shell
vim /etc/pam.d/login
```

在文件最后追加一行：

```
session    required     pam_limits.so
```

保存并退出即可。



到这里，我们对系统资源的限制就永久的修改了。





#### 4.2.4. 验证



上面的 PAM 修改都完成以后，需要重启系统才会生效。



我们刚刚修改的是普通用户下的打开文件描述符的最大数量，对应 `ulimit -n`，重启系统以后，我们验证一下：



```shell
$ ulimit -n
1024
$ ulimit -Hn
8192
```

可以看到，符合预期，修改成功了 ~



完结撒花 ~