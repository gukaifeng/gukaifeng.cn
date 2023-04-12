---
title: "配置 Windows 为 SSH 跳板机"
date: 2023-04-10 20:07:00
updated: 2023-04-12 13:17:00
categories: [技术杂谈]
tags: [SSH,Windows,Linux]
---



## 1. 场景

本文以 Windows 10 为例，10/11 均适用。

我在 Windows 上装了 VMware，里面运行着一个 Linux 虚拟机。

我有另外一台**内网上的**机器，想通过 SSH 登录到这个 Linux 虚拟机上开发。

由于某些原因（我这里是公司内网分配 IP 需要一些这个场景下无法完成的验证），虚拟机的桥接模式设定不适用，无法给 Linux 虚拟机一个单独的内网 IP 地址，也就无法在其他机器上通过内网直接登录此虚拟机。

但是 Windows 宿主机是有内网的 IP 的，所以我们可以先 SSH 登录到此 Windows 宿主机，再以 Windwos 宿主机作为跳板，登录到虚拟机（Windows 宿主机与虚拟机也组成一个内网）。

此方案同样适用从内网某机器以 Windows 作为跳板机登录宿主机所在其他内网的其他机器。



## 2. 安装 OpenSSH 服务器

Windows 里 OpenSSH 客户端通常是自带的，但是 OpenSSH 服务器需要手动安装。

**Step 1：**我们打开 Windows 设置 -> 应用 -> 应用和功能，选择“可选功能”，如下：

![Step 1](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_1.png)





**Step 2：**点击“添加功能”。



![Step2](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_2.png)



**Step 3：**搜索并安装“OpenSSH 服务器”：

![Step 3](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_3.png)





到这里就需要的功能就安装完了，可以在“已安装功能”里搜索看一下，像下面这样就是正确的：

![“已安装功能”应当已有 “OpenSSH 服务器”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_4.png)



## 3. 启动 OpenSSH 服务



这里图形界面和命令行都能完成，随便选个就 OK。



### 3.1. 图形界面方式

首先在开始菜单里搜索“服务”并打开：



![打开“服务”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_5.png)



在右侧找到 OpenSSH SSH Server 项，点击左面的“启动”。



![启动 SSH 服务](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_6.png)

\-

如果后面不需要了，关闭的方法也是一样的，启动后上图的“启动”的位置会变成“停止”，点一下就关了。



### 3.2. 命令行方式

以管理员身份打开 PowerShell 或 CMD，输入命令：

```powershell
net start sshd
```

输出如下信息，表示 OpenSSH 服务启动成功：

```powershell
PS C:\Windows\system32> net start sshd
OpenSSH SSH Server 服务正在启动 ..
OpenSSH SSH Server 服务已经启动成功。
```

\-

如果后面不需要了，关闭的命令为：

```powershell
net stop sshd
```





## 4. SSH 登录 Windows 跳板机

首先我们要知道 Windows 的内网地址，在 PowerShell 或 CMD 中可以使用 `ipconfig` 命令查看：

```powershell
PS C:\Windows\system32> ipconfig

Windows IP 配置


以太网适配器 以太网:

   连接特定的 DNS 后缀 . . . . . . . : mioffice.cn
   临时 IPv6 地址. . . . . . . . . . : fdec:5623:de91:5c00:258a:46af:62c3:9b08
   IPv6 地址 . . . . . . . . . . . . : fdec:5623:de91:5c00:c94d:5d7b:818:640c
   本地链接 IPv6 地址. . . . . . . . : fe80::c94d:5d7b:818:640c%6
   IPv4 地址 . . . . . . . . . . . . : 10.189.62.0
   子网掩码  . . . . . . . . . . . . : 255.255.224.0
   默认网关. . . . . . . . . . . . . : 10.189.32.1
  
...
...
...
```

可以看到我这里的 Windows 宿主机的内网 IP 地址为 `10.189.62.0`。



现在掏出我们处于 Windows 宿主机所在内网内的另一台机器，我这里是一台 mac 笔记本 ~

在命令行通过 SSH 登录到 Windows，这里用我们刚刚查到的 Windows 跳板机的内网 IP：

```shell
% ssh gukaifeng@10.189.62.0
gukaifeng@10.189.62.0's password:

```

这里输入你 Windows 的用户密码，就会进入到下面这个页面：

```powershell
Microsoft Windows [版本 10.0.19044.1766]
(c) Microsoft Corporation。保留所有权利。

gukaifeng@DESKTOP-78H2KFL C:\Users\gukaifeng>
```

到这里我们就成功登录 Windows 跳板机了！



## 5. 从 Windows 跳板机 SSH 登录虚拟机

我们得知道 Linux 虚拟机的内网 IP，我们在虚拟机内使用命令 `hostname -I` 查看：

```shell
$ hostname -I
192.168.80.132 
```

可以看到我这里 Linux 虚拟机的内网 IP 为 `192.168.80.132`。

其实这个场景就是，Windows 宿主机处于两个局域网当中，但这两个局域网是不通的。

我们上一节已经成功从一台内网上的 mac 笔记本登录到了这个 Window 机器，后面就很简单了，就直接 SSH 登录虚拟机就 OK 了。例如：

```shell
Microsoft Windows [版本 10.0.19044.1766]
(c) Microsoft Corporation。保留所有权利。

gukaifeng@DESKTOP-78H2KFL C:\Users\gukaifeng> ssh gukaifeng@192.168.80.132
Last login: Mon Apr 10 20:36:55 2023 from 192.168.80.1
[gukaifeng@localhost ~]$
```

（我这里已经配置好了 SSH 免密登录）



登录成功，完结撒花 ~



> 一点题外话：
>
> 如果你要把 Windows 始终开着跑这个虚拟机，但人又不在 Windows 机器前，建议设置下 Windows 机器锁屏但不休眠。锁屏还是有必要的，避免公司内其他人动你的电脑。避免休眠能够保持虚拟机始终可以登录。





## 6. 扩展：配置开机自启

之前的做法是手动启动 Windows 的 SSH 服务和 VMware 虚拟机的，  
如果 Windows 重启的话，我们还得再手动开，比较麻烦，有必要设置一下自动启动。

配置完以后，我们启动或重启 Windows 后，是不需要登入的，也就是在用户登录的锁屏页面，SSH 服务和虚拟机就已经启动了。这样即便机器处于关机状态，想找别人帮忙开机的话，也可以避免其他人登入系统。

### 6.1. Windows SSH 服务开机自启



首先在开始菜单里搜索“服务”并打开：



![打开“服务”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_5.png)

找到 OpenSSH SSH Server 项，右键选择“属性”：

![打开 “OpenSSH SSH Server” 的属性](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_7.png)

注意这里 SSH 服务得是“正在运行”的状态。重启 Windows 前 SSH 服务正在运行，下面设置自启后才会自启。如果重启 Windows 前 SSH 服务是停止状态，那么即便配置了自启，也不会生效。

选择“启动类型”为“自动”，然后点击“确定”。

![选择“启动类型”为“自动”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_8.png)

这样 SSH 服务就可以开机自启了。



### 6.2. VMware 虚拟机开机自启



这里墙裂建议使用 17.x 及以上版本的 VMware Workstation，因为从这个版本开始，VMware 自带了开机自启虚拟机功能。

如果你用的是旧版本的 VMware，可以看看网上的其他资料（比较麻烦），这里以 VMware 自带的自启虚拟机功能来说。

我这里以 VMware Workstation 17.0.0 Pro 为例。

**Step 1：**再次打开“服务”，找到 “VMware 自动启动服务”，点击左边的“启动”：

![启动 “VMware 自动启动服务”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_9.png)

**Step 2：**打开 “VMware 自动启动服务” 属性：

![打开 “VMware 自动启动服务” 的属性](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_10.png)

**Step 3：**选择“启动类型”为“自动”

![选择“启动类型”为“自动”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_11.png)

注意配置这里时同样 VMware 自动启动服务 得是“正在运行”的状态。



**Step 4：**以**管理员身份**启动 VMware，以**管理员身份**启动 VMware，以**管理员身份**启动 VMware。



如果不是管理员的话会有权限问题，也只有配置虚拟机开机自启的时候需要以管理员身份启动，后面正常使用就不需要了。



>这里的权限要求，严格来说是我们的用户必须具有以下权限：
>
>- 对位于 %ALLUSERSPROFILE%\VMware\VMware Workstation\vmAutoStart.xml 的 vmAutoStart.xml 文件的写入访问权限。
>- vmAutoStart.xml 文件中指定的 VMX 文件的所有权。
>
>直接用管理员身份启动 VMware 是最省事的。





**Step 5：**选择“文件” -> “配置自动启动虚拟机”：

![配置自动启动虚拟机](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_12.png)

**Step 6：** 给我们要设置自启的虚拟机的 “自动启动” 下面打勾：

![“自动启动” 下面打勾](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_13.png)

如果你有多个虚拟机要自启，可以设置下右边的“启动顺序”，我这里就这一个，就不演示了。

\-

另外有一点需要注意，**自动启动的虚拟机，并不会在 VMware 的控制面板里体现**：

![任务栏图标状态显示“没有正在运行的虚拟机”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_14.png)

![VMware 控制面板显示虚拟机状态为 “已关机”](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_15.png)

我们也无法在控制台里重复启动此虚拟机，会有像下面这样的提示：

![无法以独占方式锁定配置文件](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_16.png)

![启动虚拟机失败](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_17.png)

但是不用担心，**我们的虚拟机是正确运行了的，是可以在 Windows  宿主机上  SSH  直接或跳板登录的。**这里就不演示了。

虽然我们无法在 VMware 控制面板里再操作我们的虚拟机，但我们虚拟机内的任何变化都是会正确写入虚拟机目录的，我们不用担心对虚拟机的修改消失。





\-

在这个情况下，我们无法通过 VMware 控制台对我们的虚拟机做任何事，例如制作快照也会失败：

![制作快照失败](https://gukaifeng.cn/posts/pei-zhi-windows-wei-ssh-tiao-ban-ji/pei-zhi-windows-wei-ssh-tiao-ban-ji_18.png)

当我想要对虚拟机做些操作的时候（例如制作快照、修改属性、备份虚拟机等），我的做法是，先 SSH 登录到 Linux 虚拟机内部，然后执行命令关机：

```shell
sudo shutdown -h now
```

关机后，就可以在 VMware 控制页面对虚拟机进行操作了，这时就恢复到了和没有配置虚拟机自启时一样的状态，后面可以在 VMware 控制面板里再次手动启动虚拟机。

（直接在任务管理器里杀掉虚拟机进程是不理智的。。。）

\- 

到这里自动启动相关的就都配置好了，现在重启 Windows，SSH 服务和虚拟机都会自启了，可以愉快的开发了 ~