
**网络文件系统**（英语：**Network File System**，缩写作 **NFS**）是一种[分布式文件系统](https://en.wikipedia.org/wiki/Distributed_data_store)，力求客户端主机可以访问服务器端文件，并且其过程与访问本地存储时一样，它由[Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems)（已被甲骨文公司收购）开发，于1984年发布。NFS 的实现基于 ONC RPC（一个开放、标准的[RFC](https://en.wikipedia.org/wiki/Request_for_Comments)系统，任何人或组织都可以依据标准实现它）。



**本文仅介绍 NFS 最简单的用法：在两台不同的 Linux 机器上共享目录。**



而 NFS 那些更复杂的用法基本也都是在此简单用法之上的拓展，深入了解配置，举一反三便能做到。



\-



我这里以一个简单地实例的实现，来阐述整个过程：



假设有 A 和 B 两台不同的但可以相互连接的 Linux 服务器（我这里均以 CentOS 系统举例），我们要将 A 服务器上的 `/nfs-shared/` 目录共享给 B 服务器。在 B 服务器上，将 A 服务器上共享来的目录，挂载（NFS 是通过挂载的方式来访问共享内容的）到自己 `/nfs-shard-from-server/` 目录。完成后，A 服务器访问自己的 `/nfs-shared/` 目录，与 B 服务器访问目录 `/nfs-shard-from-server/` 看到的内容应当始终完全一致（不过 B 机器上可能会有延迟，毕竟是要远程放到 A 服务器的，具体取决 A、B 互联的网络质量）。



> A 和 B 的连接是内网或公网均可。但公网比较麻烦，要解决防火墙的问题。而且由于 NFS 是明文，所以一般不建议在公网使用。本文的示例用的公网主要是因为我在搞这个的时候手里暂时只有两个不在同一网段的机器，又懒得搞 docker 啥的=。=，不过你直接在内网照做也没什么区别。

我们将 A 称为 **NFS 服务器**，将 B 称为 **NFS 客户端**。下面也都将以 NFS 服务器和 NFS 客户端称呼，不再提及 A 和 B。

NFS 服务器共享目录，NFS 客户端访问 NFS 服务器共享出来的目录。



下面开始。



## 1. 通用操作：安装必要的程序



在 CentOS 中 NFS 的主程序软件包名为 `nfs-utils`。前文说过，NFS 的实现是基于 RPC 的，所以当我们安装 `nfs-utils` 时，相关依赖 `rpcbind` 包也会被自动检索到并安装。我们只需要安装一个 `nfs-utils` 即可，其依赖 yum 包管理器会解决：

```shell
yum install nfs-utils
```

NFS 服务器和 NFS 客户端都需要安装。





## 2. NFS 服务端操作

### 2.1. 编辑 NFS 服务器配置文件

NFS 服务器的配置文件在 `/etc/exports`：



我们打开（如果是空的直接编辑就好，如果不存在就创建新的）：

```shell
vim /etc/exports
```

其**配置非常非常非常简单！**每个要共享的目录占一行，每行格式如下：

```
<export> <host1>(<options>) <hostN>(<options>)...
```

* `export`：要共享的目录。注意必须是目录，NFS 共享的最小单位就是目录。
* `host`：要共享的目标主机，即可访问此共享目录的 NFS 客户端。这是 NFS 服务器唯一验证访问者身份的方式。
  * 可以使用完整的 IP 或者是网域，例如 `192.168.100.10` 或 `192.168.100.0/24` 或 `192.168.100.0/255.255.255.0` 都可以！
  * 也可以使用主机名，但这个主机名必须要在 `/etc/hosts` 内，或可使用 DNS 找到该名称才行！反正重点是可找到 IP 就是了。如果是主机名的话，那么他可以支持通配符，例如 `*` 或 `?` 均可。
  * 也可以直接为 `*` 表示允许所有人访问。
  * 注：NFS 同时支持公网和内网共享，但由于 NFS 是明文的，通常不建议公网共享，而且局域网内使用。
* `options`：给 `host` 共享 `export` 目录的选项，多个选项用逗号 `,`隔开。每组 `options` 只服务一个 `host`。



这里给出几个 `options` 的常用选项（更多详见 [man exports](https://man7.org/linux/man-pages/man5/exports.5.html)）：

| 参数值                              | 含义                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `rw`<br />`ro`                      | 设置目录分享权限。`rw` 表示可读写（read-write），`ro` 表示只读（read-only）。<br />最终的客户端对目录的访问权限为此选项与服务器系统用户实际拥有权限的交集。<br />比如这里设置 `rw` 权限，但是服务器中自己也没有写权限，那么客户端也就只能是只读权限。<br />**另外要注意的是，通常访问者被服务器看做其他用户，即权限设置的最后三位。例如，服务器共享的目录权限为 `drwxrwxr-x`，那么访问者的最大权限就是后三位的 `r-x`。** |
| `sync`<br />`async`                 | `sync` 代表数据会同步写入到内存与硬盘中，`async` 则代表数据会先暂存于内存当中，而非直接写入硬盘。 |
| `no_root_squash`<br />`root_squash` | 当 NFS 客户端访问此共享目录时，如果使用的是 root 身份，是否在服务器端保留同等的 root 身份（例如 NFS 客户端使用 root 用户在共享目录中添加了一个文件后，服务器端或其他客户端看到的此文件修改者仍是 root）。`no_root_squash` 为保留，默认是 `root_squash` 为不保留。 |
| `all_sqush`                         | 映射所有访问者的 UIDs 和 GIDs 为匿名用户。                   |
| `anonuid`<br />`anongid`            | 显式设定匿名访问者 UID 和 GID。设定后，所有匿名访问此共享目录的请求看起来都是同一个用户（相同的 UID 或 GID）。<br />与 `all_sqush` 搭配使用可以让所有访问用户都使用同一个指定的账户身份访问（先将所有访问者都映射为匿名的，再给匿名用户指定 UID 和 GID）。 |



本例添加一条如下：

```
/nfs-shared 47.93.49.1(rw)
```

即共享服务器目录 `/nfs-shared` 给 `47.93.49.1`，赋予读写权限。



\-



配置完 `/etc/exports` 后，如果你的 nfs 服务**正在运行**，则需要使用命令 `exportfs -arv ` 使其生效。

```shell
exportfs [-aruv]
```

* `-a`：全部挂载(或卸除) `/etc/exports` 内的设定。
* `-r`：重新挂载 `/etc/exports` 里面的设定，此外，亦同步更新 `/etc/exports` 及 `/var/lib/nfs/xtab` 的内容。
* `-u`：卸除某一目录。
* `-v`：在 `export` 的时候，将共享的目录显示到屏幕上！









### 2.2. 启动 NFS 服务



前面说过，NFS 服务依赖于 RPC，所以我们启动 NFS 服务前，需要先启动 RPC 服务：

```shell
systemctl start rpcbind
systemctl start nfs-server
```



启动后，我们可以检查一下 RPC 和 NFS 服务开启了哪些端口：

```shell
$ netstat -tulnp | grep -E '(rpc|nfs|1/systemd)'
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 0.0.0.0:20048           0.0.0.0:*               LISTEN      2049808/rpc.mountd  
tcp        0      0 0.0.0.0:56525           0.0.0.0:*               LISTEN      2049794/rpc.statd   
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::20048                :::*                    LISTEN      2049808/rpc.mountd  
tcp6       0      0 :::40445                :::*                    LISTEN      2049794/rpc.statd   
udp        0      0 0.0.0.0:20048           0.0.0.0:*                           2049808/rpc.mountd  
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1/systemd           
udp        0      0 127.0.0.1:778           0.0.0.0:*                           2049794/rpc.statd   
udp        0      0 0.0.0.0:50449           0.0.0.0:*                           2049794/rpc.statd   
udp6       0      0 :::20048                :::*                                2049808/rpc.mountd  
udp6       0      0 :::36774                :::*                                2049794/rpc.statd   
udp6       0      0 :::111                  :::*                                1/systemd
```

这里可以看到，`nfs-server` 本身没有占用端口，因为它是通过 `rpcbind` 来操作的，所以只能看到 `rpcbind` 的端口占用。

`rpcbind` 主程序的是在端口 111 启动的。



> 我这里还打印出了 `1/systemd` 的端口是因为，我这里的 rpcbind 是通过 systemctl 启动的，所以占用端口会挂载 systemd 名下，我们可以查看一下 rpcbind 的运行信息，可以看到这里的任务号为 1，所以查看的是 `1/systemd`的端口占用：
>
> ```shell
> $ systemctl status rpcbind
> ● rpcbind.service - RPC Bind
>    Loaded: loaded (/usr/lib/systemd/system/rpcbind.service; enabled; vendor preset: enabled)
>    Active: active (running) since Sat 2022-12-03 17:25:36 CST; 29min ago
>      Docs: man:rpcbind(8)
>  Main PID: 2049619 (rpcbind)
>     Tasks: 1 (limit: 49489)
>    Memory: 1012.0K
>    CGroup: /system.slice/rpcbind.service
>            └─2049619 /usr/bin/rpcbind -w -f
> 
> Dec 03 17:25:36 iZ8vbf7xcuoq7ug1e7hjk5Z systemd[1]: Starting RPC Bind...
> Dec 03 17:25:36 iZ8vbf7xcuoq7ug1e7hjk5Z systemd[1]: Started RPC Bind.
> ```



然后，我们也有必要验证一下，我们的目录是否已经共享出来了，使用 `showmount` 命令：

```shell
showmount [-ae] [hostname|IP]
```

* `-a` ：显示目前主机与客户端的 NFS 联机分享的状态；
* `-e` ：显示某部主机的 `/etc/exports` 所分享的目录数据。
* `[hostname|IP]`：要显示的主机名或 IP。不写的话就是本机。



```shell
$ showmount -e
Export list for iZ8vbf7xcuoq7ug1e7hjk5Z:
/nfs-shared 47.93.49.1
```

显示出我们在 `/etc/exports` 中配置的内容，就是成功了。





### 2.3. 打开 NFS 使用的端口



如果你要在**公网**使用 NFS，有一点也非常重要，我们必须确保 `nfs-server` 在 `rpcbind` 中注册的端口是可以访问的状态！



我们查看 rpc 相关的端口：

```shell
$ rpcinfo -p localhost | grep nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
```

可以看到我这里用的是 2049 端口，我们必须打开服务器的这个端口（例如阿里云中的安全组设置），否则客户端依然是无法访问/挂载的。





## 3. NFS 客户端操作



### 3.1. 启动 NFS 服务

客户端同样需要启动 `rpcbind` 和 `nfs-server`。

```shell
systemctl start rpcbind
systemctl start nfs-server
```

不过客户端这边没必要特别关注什么其他的东西，就不像 NFS 服务器那样多说了。





### 3.2. 挂载共享目录

挂载之前，我们同样可以使用 `showmount -e` 命令，查看服务器共享了哪些目录，例如：

```shell
$ showmount -e 8.142.120.167
Exports list on 8.142.120.167:
/nfs-shared
```

> 如果你是在公网共享的话，`showmount -e` 这里可能会面临防火墙问题，你需要打开服务器端相应的端口（不是挂载的端口，是个 UDP 端口）。这里暂时不是重点，先不说了，因为不影响挂载。





挂载目录使用 `mount` 命令。`mount` 的功能挺多的，具体见 [man mount](https://man7.org/linux/man-pages/man8/mount.8.html)。



这里就仅展示最基础的一个用法：

```shell
mount [options] <source> <directory>
```

我们暂不考虑 `options`，就非常简单了，就是 `mount` 后面接源目录地址和目的目录地址。

我这里把之前 NFS 服务器的 `/etc/exports` 配置在这里再写一次：

```
/nfs-shared 47.93.49.1(rw)
```

我这里的 NFS 服务器的地址为 `8.142.120.167`，故我这里源目录地址应当填 `8.142.120.167:/nfs-shared`。

目的目录地址应当是 NFS 客户端本地的一个**空**目录，这里现创建一个：

```shell
mkdir /nfs-shared-from-server
```

现在我们本例中 NFS 客户端中最后的挂载命令即：

```shell
mount 8.142.120.167:/nfs-shared /nfs-shared-from-server
```

如果没有错误信息的话，就是挂载成功了。

现在我们 NFS 客户端访问自己的目录 `/nfs-shared-from-server`，就是在访问服务器端的共享目录 `/nfs-shared` 了。



搞定！





## 4. 可能遇到的问题与解决方案

### 4.1. 使用非法端口



客户端挂载时出现下面的错误信息：

```
mount.nfs: access denied by server while mounting 8.142.120.167:/nfs-shared
```

问题有多种，如果你**非常确定你配置中的路径、地址等信息没有写错**，接着往下看。

在 NFS 服务器上查看 log：

```shell
$ sudo tail /var/log/messages | grep mount
Dec  3 20:35:27 iZ8vbf7xcuoq7ug1e7hjk5Z rpc.mountd[2050307]: refused mount request from 111.198.231.50 for /nfs-shared (/nfs-shared): illegal port 21497
Dec  3 20:35:42 iZ8vbf7xcuoq7ug1e7hjk5Z rpc.mountd[2050307]: refused mount request from 111.198.231.50 for /nfs-shared (/nfs-shared): illegal port 19473
Dec  3 20:37:46 iZ8vbf7xcuoq7ug1e7hjk5Z rpc.mountd[2050307]: refused mount request from 111.198.231.50 for /nfs-shared (/nfs-shared): illegal port 19959
Dec  3 20:48:43 iZ8vbf7xcuoq7ug1e7hjk5Z rpc.mountd[2050307]: refused mount request from 111.198.231.50 for /nfs-shared (/nfs-shared): illegal port 21324
```

可以看到显示非法端口。



这个问题的主要原因是，NFS 服务器的配置文件中 `/etc/export` 中有个默认选项 `secure`，此选项要求不使用 gss 的请求源自小于 `IPPORT_RESERVED` (1024) 的 Internet 端口。然而在很多场景中该端口总是大于 1024 的（例如在使用 NAT 网络地址转换时）。



在 NFS 服务器的配置文件 `/etc/export` 中指定 `--insecure` 来关闭此要求，例如：

```shell
/nfs-shared 47.93.49.1(rw,insecure)
```

> 注意：旧内核（上游内核版本 4.17 之前）对 gss 请求强制执行此要求，无法关闭。