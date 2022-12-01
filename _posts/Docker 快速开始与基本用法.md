---
title: Docker 快速开始与基本用法
date: 2022-11-30 23:43:00
updated: 2022-12-02 01:44:00
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

## 1. 安装 Docker



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

\-

如果你的网络访问官方的仓库比较困难的话，这里给出两个国内源供大家选择：

```shell
# aliyun mirrors
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

```shell
# tsinghua mirrors
sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
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

上面的命令中， `docker run` 会下载一个镜像 `hello-world`（如果镜像在本地找不到的话，就会自动去官方提供的远程仓库找，都没有就会报错），并在一个容器内运行这个镜像，容器运行起来后，就会打印出上面的小作文，然后退出。镜像 `hello-world` 是官方提供的一个用来验证 docker 是否正确安装的镜像，如果你此步骤和我的输出一致，就说明你的 docker 安装全部成功了。





## 2. 创建并登录一个容器



### 2.1. 查找镜像



Docker 创建容器时必须指定一个镜像，如果你已经懂得镜像相关内容并且已有所需镜像，则可跳过本小节。

我们可以做个类比，想象我们正在一个虚拟机上安装操作系统，那么容器就相当于这个虚拟机，而 Docker 镜像就相当于我们要在虚拟机上安装的 iso 镜像。所以我们必须先有一个镜像，然后再考虑容器相关的事。

至于镜像从哪里来，官方有提供[docker hub](https://hub.docker.com/search?q=&type=image)，我们可以在上面找到各种镜像、扩展和插件（这里暂不考虑第三方渠道）。

不过一般除非有特定需求，不然没有必要去网站上找，使用命令 `docker search` 可以搜索镜像，其结果和 docker hub 上面的一致。

我这里以查找 `almalinux` 镜像为例：

```shell
$ sudo docker search almalinux
NAME                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
almalinux                  The official build of AlmaLinux OS.             98        [OK]       
almalinux/almalinux        DEPRECATION NOTICE: This image is deprecated…   9                    
almalinux/8-micro          AlmaLinux OS 8 official micro container image   2                    
almalinux/arm64v8          AlmaLinux OS 8 official aarch platform images   2                    
almalinux/mirror_service   AlmaLinux OS mirror service.                    1                    
almalinux/ks2rootfs        Kickstart to RootFS file builder in docker/p…   0                    
almalinux/9-base           AlmaLinux 9 Base container image                0                    
almalinux/9-micro          AlmaLinux 9 Micro container image               0                    
almalinux/8-minimal        AlmaLinux OS 8 official minimal image           0                    
almalinux/9-init           AlmaLinux 9 Init container image                0                    
almalinux/ppc64le          AlmaLinux OS 8 official ppc64le platform ima…   0                    
almalinux/8-init           AlmaLinux OS 8 official init image              0                    
almalinux/9-minimal        AlmaLinux 9 minimal container image             0                    
almalinux/amd64            AlmaLinux OS 8 official x86_64 platform imag…   0                    
almalinux/s390x            AlmaLinux 9 s390x arch container image          0                    
almalinux/8-base           AlmaLinux OS 8 official base image              0                    
almalinux/pause                                                            0                    
dokken/almalinux-8         Alma Linux 8 image for use with the kitchen-…   0                    
dokken/almalinux-9         AlmaLinux 9.x image for use with Test Kitche…   0                    
almalinux/i386             AlmaLinux container images for i686 arch - e…   0                    
almalinux/java                                                             0                    
almalinux/python                                                           0                    
almalinux/buildah                                                          0                    
almalinux/golang                                                           0                    
almalinux/node                                                             0                    
```

可以看到输出了 25 个与关键词 "almalinux" 有关的条目，默认就是输出所有结果的前 25 条。



具体命令使用方法如下：

```shell
docker search [OPTIONS] TERM
```

其中 `[OPTIONS]` 有：

* `-f` `--filter filter`：基于提供的条件筛选输出。
* `--format string`：使用一个 Go 模板来格式化输出。
* `--limit int`：显示的最大条目数量（默认为 25）。
* `--no-trunc`：不截断输出（一般是镜像的描述 `DESCRIPTION`）。



### 2.2. 下载镜像



搜索到合适的镜像以后，我们就可以下载镜像到本地了，我们选择一个镜像，下载镜像需要使用镜像仓库的完整名字（即上面示例中的 `NAME` 列）。

下载镜像的命令的是 `docker pull`：

```shell
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

* `[OPTIONS]`
  * `-a` `--all-tags`：下载镜像仓库中的所有 Tag。这会把 `NAME` 仓库中的各种版本的镜像都下载下来，我个人不太建议。
  * `--disable-content-trust`：跳过镜像验证（默认为 true）。
  * `--platform string`：设置平台（如果服务支持多平台）。
  * `-q` `--quiet`：仅输出简略的信息。
* `NAME`：镜像所在仓库的名字。即 `docker search` 结果中的第一列，必须写完整。
* `TAG`：要下载的镜像标签，默认为 `latest`。
* `DIGEST`：镜像的摘要。其值为一个 64 位的 sha256 哈希值，在镜像打包时生成，且不可再更改。



> 这里的 `TAG` 和 `DIGEST` 在命令 `docker search` 的搜索结果里是不显示的，如果你想下载非默认版本的镜像，还得去 [docker hub](https://hub.docker.com/search?q=&type=image) 里看看对应的 `TAG` 或 `DIGEST` 是什么才行。我这里使用默认的下载了：  
> `docker pull almalinux` 等价于 `docker pull almalinux:latest`。



下载默认的 `almalinux` 镜像：

```shell
docker pull almalinux
```

然后我们可以使用 `docker images` 命令查看我们已有的镜像：

```shell
$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
almalinux     latest    39f63d416992   2 weeks ago     190MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB
```

可以看到我们机器上已经有了一个 TAG 为 `latest` 的 almalinux 镜像了。

下面就可以开始创建容器了。

### 2.3. 创建容器





## 3. 常用操作

### 3.1. 在本地文件系统和容器之间拷贝

使用 `docker cp` 命令，方法如下：

```shell
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

* `[OPTIONS]` 选项（可选）:
  * `-a` `--archive`：归档模式（拷贝所有 uid/gid 信息）。
  * `-L` `--follow-link`：同时拷贝源目标中的符号链接，类似 `cp` 命令中的参数 `-a`。
* `CONTAINER`：容器 id，即拷贝的源或目的容器的 id，即 `docker ps` 命令输出中第一列的 `CONTAINER ID`。
* `SRC_PATH`：要拷贝的源路径。源路径是本地文件系统路径时，其值可以为 `-` 表示从标准输入 `stdin` 读取一个 `tar` 归档，并在目的路径提取。
* `DEST_PATH`：要拷贝的目的路径。目的路径是本地文件系统时，其值可以为 `-` 将容器源的 `tar` 存档流式传输到标准输出 `stdout`。



有一点要注意的是，**书写容器中的路径时，一定要写绝对路径**，像 `~` 这种是不行的，因为其值是在本地文件系统解析的。



另外，`docker cp` 与系统提供的 `cp` 有一点区别是，`docker cp` 不支持正则，也就是源路径和目的路径必须是完整确切的路径。这样的话，如果需要同时使用参数 `-L` 拷贝符号链接的话，似乎就只能是拷贝上级整个目录。



### 3.2. 删除容器

命令 `docker rm` 用于删除一个或多个容器：

```shell
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

其中 `[OPTIONS]` 有如下：

* `-f` `--force`：强制删除正在运行中的容器（会使用 `SIGKILL`）。默认情况下是不允许删除正在运行中的容器的。
*  `-l` `--link`：删除指定的链接。
* `-v` `--volumes`：删除与容器关联的匿名卷。



### 3.n. 卸载 Docker

卸载与安装的命令其实是相对应的：

```shell
sudo yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

机器上的镜像、容器、卷或自定义配置文件不会自动删除。要删除所有镜像，容器和卷：

```shell
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```



