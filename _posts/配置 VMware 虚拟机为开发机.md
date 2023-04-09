---
title: "配置 VMware 虚拟机为开发机"
date: 2023-04-08 17:35:00
updated: 2023-04-08 17:35:00
categories: [技术杂谈]
tags: [Linux]
---





## 1. 我为什么选择虚拟机做开发机

我的开发机最终选择是在 VMware 上的 Linux 虚拟机。



我个人采用这样的开发方式，很大原因是非常喜欢这种模式的便捷性。

* VMware 是跨平台的，我们的虚拟机可以直接通过 U 盘拷走，随处安置。

* 虚拟机可以随时制作快照，备份、回滚非常简单，完全不用担心把环境搞坏了。



关于这个开发模式，我自己也是初次使用，后面如果遇到哪些问题，或者有哪些新的心得，我也会更新到这篇文章里 ~

## 2. 镜像安装



我选择装在虚拟机中的 Linux 发行版及版本是 AlmaLinux-8.7-x86_64-minimal。

线上生产环境是 CentOS 7.3，但是公司电脑与我家里电脑的 CPU 比较新，无法通过 CentOS 7.x 的硬件检测。

所以在开发机上，与其装已经不再维护的 CentOS 8.x，不如用 AlmaLinux 了。

这里没有选择最新的 AlmaLinux 版本，而是选择了 8.7（截止写此文章时，最新版本为 9.1），主要是为了规避过新版本的潜在风险。



镜像下载地址：[AlmaLinux-8.7-x86_64-minimal.iso](https://mirror.sjtu.edu.cn/almalinux/8.7/isos/x86_64/AlmaLinux-8.7-x86_64-minimal.iso)



Vmware 中的配置可以随意修改，我这里暂时设定的是：

* 硬盘：20GB
* 内存：8GB
* 处理器数量： 8
* 每个处理器的内核数量：1







安装过程按部就班就可以了，别忘记 root 账户密码就好了。





## 3. 创建个人用户并赋权



首先登录以 root 账户登录我们刚刚装好的 AlmaLinux-8.7 虚拟机（以下统一简称“虚拟机”），  
然后创建一个新用户，我这里用户名是 gukaifeng ：





依次输入命令：

```shell
adduser gukaifeng  # 新建一个名为 gukaifeng 的用户
```

```shell
passwd gukaifeng  # 为 gukaifeng 用户设置密码
```

然后再依次输入以下命令：

```shell
chmod +w /etc/sudoers  # 给权限配置文件添加写权限
```

```shell
vi /etc/sudoers  # 编辑权限配置文件，minimal 版本是没有预装 vim 程序的
```

找到下面这段代码

```
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
```

仿照这个格式，为我们的 gukaifeng 用户添加一行，即：

```
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
gukaifeng    ALL=(ALL)       ALL
```

保存退出后，执行命令：

```
chmod -w /etc/sudoers  # 记得移除权限配置文件的写权限
```

\-



然后，使用刚新建的用户 gukaifeng，重新登录主机。

本文后面所有的步骤，都是在 gukaifeng 用户下进行的。





## 4. 启动网卡

如果你的网络不可用，则需要按此部分配置。否则可以跳过本小节。



有这一小节主要是我发现 AlmaLinux 8.7（对应 CentOS 8.x）默认是不启动网卡的。



这样的话，你使用各种相关命令都是查不到 IP 的，例如：

```shell
$ hostname -I

# 输出为空
```

没有 IP 我们基本什么都做不了了，下面来解决这个问题。



我们编辑网络配置文件：

```shell
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

其默认内容像下面这样：

```properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPU4_FAILURE_FATAL=no
IPU6INIT=yes
IPU6_AUTOCONF=yes
IPU6_DEFROUTE=yes
IPU6_FAILURE_FATAL=no
IPU6_ADDR_GEN_MODE=eui64
NAME=ens33
UUID=a415c02c-c2db-4f94-8216-1143a39bbd49
DEVICE=ens33
ONBOOT=no
```

我们需要修改 `ONBOOT` 字段为 `yes` 即可，即表示默认启动网卡。

```properties
ONBOOT=yes
```

退出并保存，  
然后应用我们的更改（或者重启系统）：

```shell
sudo nmcli c reload
```



现在网络就可用了。






## 5. 安装必要的程序



因为我们安装的是 minimal 版本的镜像，所以很多必要的程序是没有的（比如 vim，我们之前的示例都在用 vi）。

这小节记录一下我安装的一些通用程序，读者可以按需选择。

```shell
sudo yum -y install vim wget net-tools
```





## 6. 宿主机 SSH 登录虚拟机





如果你打算将虚拟机当做一个普通的服务器，在宿主机上 SSH 到这个虚拟机开发的话，就需要这小节的内容。



### 6.1. 准备密钥

这里假定你已经有了 SSH 的密钥文件：

* 公钥：`id_rsa.pub`
* 私钥：`id_rsa`

如果你不知道如何生成一对密钥，建议查阅 [使用 ssh-keygen 生成密钥](https://learn.microsoft.com/zh-cn/azure/virtual-machines/linux/create-ssh-keys-detailed)。

如果你是在虚拟机内新创建的密钥，则等下需要将私钥传输出来；如果你是有一对现成的密钥，则等下需要将公钥传输进虚拟机内。

我这里将以将公钥传输进虚拟机内为例。

### 6.2. 查看虚拟机 IP

在虚拟机内使用命令 `ifconfig` 或 `hostname -I` 可查看虚拟机 IP，这个 IP 通常是内网 IP。

```shell
$ hostname -I
192.168.17.131
```

我这里的虚拟机 IP 是 `192.168.17.131` ，记得这个 IP，后面会用到。



### 6.3. 传输密钥

首先我们需要在宿主机上创建 `.ssh` 目录，并修改权限为 700：

```shell
mkdir ~/.ssh
chmod 700 ~/.ssh
```



我在宿主机上已经存有一对密钥，所以这里以将公钥传输进虚拟机内为例。

在宿主机上（如 Windows 的 PowerShell、MacOS 的 Terminal 等等）键入命令如下：

```shell
scp /path/to/your/id_rsa.pub username@hostname:~/.ssh/authorized_keys
```

即将宿主机上的 `id_rsa.pub` 内容写入到我们虚拟机用户目录下的`~/.ssh/authorized_keys` 文件内（`authorized_keys` 中可以写很多个公钥，这里因为这是第一个公钥，所以直接在传输的时候改名就行了，如果你后续有更多的公钥要写入，那么应该追加写此文件）。



我这里的具体命令为：

```shell
scp C:\Users\micro\.ssh\id_rsa.pub gukaifeng@192.168.17.131:~/.ssh/authorized_keys
```

然后输入我们虚拟机中用户的密码，就成功了。



回到虚拟机内，我们的 `~/.ssh/authorized_keys` 中已经有了所需公钥 `id_rsa.pub` 中的内容了。

另外，我们得将 `authorized_keys` 文件的权限修改为 600：

```shell
chmod 600 ~/.ssh/authorized_keys
```



### 6.4. 测试连接



我这里在 Windows 宿主机上的 PowerShell 中操作：

```shell
PS C:\Users\micro> ssh gukaifeng@192.168.17.131
Last login: Fri Apr  7 19:43:21 2023 from 192.168.17.1
[gukaifeng@localhost ~]$
```

可以看到，不需要密码，直接登录成功了，到这里 SSH 的配置登录就完成了。



注：`ssh` 程序默认的私钥是 `~/.ssh/id_rsa`。如果你的私钥是其他的，则需要使用参数 `-i` 指定具体的私钥。





## 7. 内网其他主机 SSH 登录虚拟机



如果你有让局域网内其他机器访问此虚拟机的需求，请将虚拟机的网络适配器设定为桥接模式：



![虚拟机设置 -> 网络适配器 -> 桥接模式](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_1.png)



注意这里改动后，虚拟机的 IP 会变化，你需要重新查看虚拟机的 IP 再进行后续操作，例如我这里：



```shell
[gukaifeng@MiWiFi-RM1800-srv ~]$ hostname -I
192.168.31.220
```

设定为桥接模式以后，在宏观上，我们的虚拟机将于我们的宿主机处于同一级别。从 IP 地址的网络段上就可以看出来，我这里的宿主机 IP 是 192.168.31.241，虚拟机的 IP 是 192.168.31.220，子网掩码是 255.255.255.0，宿主机和虚拟机的 IP 属于同一网段。上面的主机名也从 localhost 变成了 MiWiFi-RM1800-srv（我家用的小米路由器）。





现在，同一网段下的所有机器，都可以 SSH 登录此虚拟机了。

例如，我手里有一台 macOS 笔记本，同样连接到家里的小米路由器，内网地址为 192.168.31.142，与虚拟机处于同一网段，则可以直接连接虚拟机（macOS 已持有私钥）：

```shell
(base) gukaifeng@gukaifengdeMacBook-Pro ~ % ssh gukaifeng@192.168.31.220
Last login: Fri Apr  7 23:46:55 2023 from 192.168.31.241
[gukaifeng@MiWiFi-RM1800-srv ~]$
```



## 8. 外网主机 SSH 登录虚拟机



如果在外部网络有一台主机想要登录我们的这个虚拟机，那该怎么办呢？

这里给出我知道的两种方法（如果读者知道更多方法，欢迎补充~）。

### 8.1. 通过内网 VPN

首先我个人的场景是：我公司有一个台式开发机，我在台式机上装了 Windows 系统，在系统里装了 VMware，在 VMware 里装了这个 AlmaLinux 8.7 虚拟机。



整个公司有一个内网，我们拿其他机器连接到公司网络（比如公司的有线网络或 WIFI 网络）的时候，这个机器和我们的虚拟机（得先按第 6 小节把网络适配器设定为桥接模式）就处于同一网段下。此时，这个问题就转变为了第 6 小节的问题，按上述流程即可解决。



> 关于如何连接到公司内网的 VPN，每个公司可能有不同的方法。
>
> 以我的公司为例，公司有提供某办公类应用程序（Windows、macOS、Linux、IOS、Android 都有），此应用程序中有统一的内网身份认证后，认证完成后就可以一键接入公司内网了。





### 8.2 通过路由转发



如果你有你所处公网 IP 的起点路由的权限，则可以在路由器上设置端口转发，将某端口的请求转发到虚拟机中的 22 端口上，这样就可以在外网主机上使用公网地址和配置转发的端口，通过 SSH 登录我们的虚拟机了。



> 此方法理论可行，但我自己没有测试过，因为我家这里是公共猫，一个楼的公网 IP 都是一样的，我没有起点路由的权限。







## 9. 虚拟机移植



这里补充一下我的虚拟机创建的环境：

* 系统：Windows 11
* VMware 产品：VMware® Workstation 16 Pro
* VMware 版本：16.1.0 build-17198959
* AlmaLinux 虚拟机的硬件兼容性：Workstation 16.x
  * 兼容产品：Fusion 12.x / Workstation 16.x
  * 限制：128 GB 内存 / 32 个处理器 / 10 个网络适配器 / 8 TB 磁盘大小 / 8 GB 共享图形内存





前文说过，我选择这个开发模式，原因之一就是虚拟机的可移植性。



我手里刚好有一台苹果电脑，相关配置如下：

* 系统：macOS Big Sur 11.1
* VMware 产品：VMware Fusion
* VMware 版本：专业版 12.1.0(17195230)



可以看到，我在 Windows 上创建的这个 AlmaLinux 虚拟机，是可以在 macOS 上的 VMware Fusion 上运行的。



我这里通过 U 盘，将前文创建的 AlmaLinux 虚拟机（记得先关闭虚拟机）转存到苹果电脑上（注意不要漏文件，最好将整个虚拟机文件夹压缩后直接拷贝、解压）。



移植非常简单，我没有遇到任何障碍。下面说一下步骤就好了。



**Step 1**：在 VMware 菜单栏选择 文件 -> 打开并运行（这里选打开也是一样的，就是运行要再点一下）。



![Step 1](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_2.png)



**Step 2**：选择我们之前的虚拟机文件夹中的 `AlmaLinux-8.7-x86_64.vmx`（如果你虚拟机是单个文件，直接选就行）。



![Step 2](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_3.png)



**Step 3**：这里可能会虚拟机是否正在使用的提示，因为我们确定没有在使用，选择“获得所有权”。



![Step 3](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_4.png)



**Step 4**：提示虚拟机可能被移动或拷贝，这里我们选择“已拷贝”（一般在单个机器上给虚拟机换位置才选“已移动”）。



![Step 4](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_5.png)



**Step 5**：提示安装 VMware Tools，这个没啥好说的。



![Step 5](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_6.png)



**Step 6**：登录测试，我这里正确登录到了虚拟机中，移植成功了。



![Step 6](https://gukaifeng.cn/posts/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji/pei-zhi-vmware-xu-ni-ji-wei-kai-fa-ji_7.png)
