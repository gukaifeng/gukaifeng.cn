---
title: "（CentOS/RedHat 系）Linux 安装最新版 Golang"
date: 2023-04-10 15:05:00
updated: 2023-04-10 15:05:00
categories: [技术杂谈]
tags: [Linux,CentOS,AlmaLinux]
---





在 https://go.dev/dl/ 页面可以看到 Golang 目前的最新版本，编辑此文章时是 1.20.3。

我们找到所需要的版本，我这里的环境是 x86_64 架构上的 AlmaLinux，所以需要的版本是：

[go1.20.3.linux-amd64.tar.gz](https://go.dev/dl/go1.20.3.linux-amd64.tar.gz)

如果你和我环境类似，也是 x86_64 架构上的红帽系 Linux，那么你应该选择和我一样的版本（最多版本号不一样。



我个人比较喜欢在一个干净的文件夹内做一件事，所以：

```shell
mkdir ~/golang_install && cd ~/golang_install
```



下载我们需要的 Golang 版本 tar 包：

```shell
wget https://go.dev/dl/go1.20.3.linux-amd64.tar.gz
```

删除机器上现有的 Golang，并将刚刚的 tar 包解压到 `/usr/local/go/`：

```shell
rm -rf /usr/local/go
tar -C /usr/local -xzf go1.20.3.linux-amd64.tar.gz
```

配置环境变量，打开 `/etc/profile`：

```sell
vim /etc/profile
```

在最后加入以下内容：

```
# Golang
export PATH=$PATH:/usr/local/go/bin
```

退出并保存，然后应用我们刚的修改（不然的话得重启系统或重新进入终端才会生效）：

```shell
source /etc/profile
```

到这里 Golang 的安装就完成了，我们检查下：

```shell
$ go version
go version go1.20.3 linux/amd64
```

可以看到，正确打印出了 Golang 的版本信息，安装成功，Over ！