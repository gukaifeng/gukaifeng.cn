---
title: Docker 快速开始与基本用法
date: 2022-11-30 23:43:00
updated: 2022-12-01 01:06:00
categories: [技术杂谈]
tags: [Linux, Docker]
---



本人不是 docker 重度用户，也不是从事运营 docker 或 k8s 相关工作的运维人员。  
我使用 docker 通常是偶尔试一些东西，避免把服务器环境搞乱；或者是想在一个相对干净的环境中临时编译一个项目等。  
所以我属于比较轻度的 docker 用户，基本上也就只会用到一些基础功能。



本文的主要目标是：

1. 从 0 快速启动一个 docker 容器，并使用 `ssh` 登录该容器。
2. 给出一些 docker 比较常用的操作和容器配置（至少是作为轻度用户的我比较常用的）。

本文使用的 Linux 发行版为 CentOS 8。

另外，docker 的安装和使用默认需要 root 权限，如果你需要在没有权限的用户里使用 docker，那么本文不适用，请查阅官方文档。

## 1. 快速开始



### 1.1. 卸载旧版本 Docker

旧版本的 docker 的包名叫做 `docker` 或者 `docker-engine`，我们需要把旧版本的 docker 卸载掉（如果有的话）：

```shell
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```

卸载命令会保留 `/var/lib/docker/` 下的镜像、容器、卷、和网络的相关内容，不用担心会丢东西。

直到该命令提示所有列出的包都不存在，我们就可以开始正式的安装了。



现在的新版本的 docker 包名为 `docker-ce`。



### 1.2. 给 yum 添加 Docker 上游仓库

包管理器 `yum` 或者 `dnf` 里面现在默认是没有包 `docker-ce` 的（而且总所周知，`yum` 里的包版本大多低的离谱）。

我们要先给 yum 添加 docker 官方提供的上游仓库，然后才可以安装 `docker-ce`。

我们安装 `yum-utils`（其提供了 `yum-config-manager` 工具），并用 `yum-config-manager` 添加上游仓库：



```shell
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```



### 1.3. 安装最新版 Docker Engine

现在我们可以直接用 yum 安装最新版的 Docker Engine、containerd 以及 Docker Compose：

```shell
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

如果你想安装或更新到指定版本，建议去看官方的文档原文 [Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/#install-docker-engine)。本文笔记向，就不记那么多了。





### 1.4. 启动 Docker



启动很简单：

```shell
sudo systemctl start docker
```

启动完成后，可以用下面的命令验证：

```shell
$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2022-12-01 00:51:01 CST; 10s ago
     Docs: https://docs.docker.com
 Main PID: 2032496 (dockerd)
    Tasks: 9
   Memory: 28.0M
   CGroup: /system.slice/docker.service
           └─2032496 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

其中 `Active `项的 `active (running)` 表示我们的 docker 正在运行，即启动成功了。



### 1.5. 验证 Docker

上面的我们 `systemctl status` 只是验证了我们 docker 正在运行，官方还提供了验证是否正确工作的方法。



直接运行命令：



```shell
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:faa03e786c97f07ef34423fccceeec2398ec8a5759259f94d99078f264e9d7af
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

命令 `docker run` 会运行一个镜像，如果镜像在本地找不到的话，就会自动去官方提供的远程仓库找，都没有就会报错。

镜像 `hello-world` 是官方提供的一个用来验证 docker 是否正确安装的镜像，如果你此步骤和我的输出一致，就说明你的 docker 安装全部成功了（命令打印的小作文看不看都行 ~）。



## 2. 启动并登录一个容器



## 3. 常用操作



