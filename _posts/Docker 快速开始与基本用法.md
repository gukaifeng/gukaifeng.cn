---
title: Docker 快速开始与基本用法
date: 2022-11-30 23:43:00
updated: 2022-12-03 01:27:00
categories: [技术杂谈]
tags: [Linux, Docker]
---



本人不是 docker 重度用户，也不是从事运营 docker 或 k8s 相关工作的运维人员。  
我使用 docker 通常是偶尔试一些东西，避免把服务器环境搞乱；或者是想在一个相对干净的环境中临时编译一个项目等。  
所以我属于比较轻度的 docker 用户，基本上也就只会用到一些基础功能。本文也不会介绍相对复杂的 `Dockerfile` 配置。



本文的主要目标是：

1. 从 0 快速启动一个 docker 容器，并使用 `ssh` 登录该容器。
2. 给出一些 docker 比较常用的操作和容器配置（至少是作为轻度用户的我比较常用的）。

本文使用的 Linux 发行版为 CentOS 8。

另外，docker 的安装和使用默认需要 root 权限，如果你需要在没有权限的用户里使用 docker，请查阅官方文档。

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





> **镜像(Images)**的官方相关定义如下：
>
> 
>
> An *image* is a read-only template with instructions for creating a Docker container.
>
> 镜像是带有创建 Docker 容器说明的只读模板。

>**容器(Containers)**的官方相关定义如下：
>
>
>
>A container is a runnable instance of an image.
>
>容器是镜像的可运行实例。
>
>
>
>A container is defined by its image as well as any configuration options you provide to it when you create or start it.
>
>一个容器由其镜像以及创建或启动该容器时你提供给该容器的所有配置选项定义。







Docker 创建容器时必须指定一个镜像。

我们可以做个类比，想象我们正在一个虚拟机上安装操作系统，那么容器就相当于这个虚拟机，而 Docker 镜像就相当于我们要在虚拟机上安装的 iso 镜像。所以我们必须先有一个镜像，然后再考虑容器相关的事。





### 2.1. 快速操作



#### 2.1. 创建并启动容器



`docker run` 命令可以快速完成：使用一个镜像创建一个容器，并启动该容器。

基本用法如下：

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```



这个命令 `[OPTIONS]` 的很多，这里列出几个我认为非常常用的：

* `--name`：此容器的名字，如果不写的话会随机分配一个名字。
* `-i`：交互模式，适用于运行的命令 `COMMAND` 是交互型的。
* `-t`：分配一个伪 TTY。
* `-d`：在后台运行此容器，并打印容器 id。
* `--privileged`：给容器扩展的权限。**很重要！确保有真正 root 的权限，不然很多事都做不了！**
* `-p`：将容器中的端口映射到本地系统的端口。最简单的用法是后接 `local_port:container_port`，例如 `-p 2222:22` 表示将容器中的 22 端口映射到本地系统的 2222 端口，就可以使用 2222 端口通过 ssh 登录此容器了。每个 `-p` 参数后只可以接一组映射。
* `-m`：限制此容器使用的内存，单位字节。



`IMAGE` 为镜像的仓库名（可以附带 TAG，如 `centos:latest`）。此命令使用的镜像 `IMAGE` 如果在本地不存在，会自动在 docker hub 查找并下载，如果都没有就会报错。

`COMMAND` 容器启动时执行的命令。`ARG` 为此命令参数。





在本例中，我要创建并启动一个 centos 镜像的容器，**在绝大多数场景下，命令差不多是这样的**：

```shell
docker run -itd --privileged --name my-centos -p 2022:22 centos /sbin/init
```

然后我们可以使用 `docker ps` 查看正在运行的容器（加 -a 参数可以看所有的）：

```shell
$ docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED          STATUS          PORTS                                   NAMES
22c11cba0f03   centos    "/sbin/init"   23 seconds ago   Up 22 seconds   0.0.0.0:2022->22/tcp, :::2022->22/tcp   my-centos
```

这里我映射容器的 22 端口主要是为了以后想要用 ssh 登录的话比较方便。你也可以以后再搞，也可以映射别的端口。

#### 2.1.2. 进入容器



现在有一个新的问题是，因为我们加了 `-d` 参数，此容器在后台执行，那么我们怎么进去操作呢？

进入容器的本质和我们登录其他机器差不多，我们要的其实只是一个可以在其内部交互的终端窗口而已。

一个简单的方法是使用 `docker exec` 命令：

```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

* `[OPTIONS]`：这里只说两个我们用得上的

  * `-i`：交互模式。
  * `-t`：分配一个伪 TTY。

  有了这两个参数，我们就可以在直接在当前窗口操作容器内了。

* `CONTAINER`：要操作的容器 ID 或名字。

* `COMMAND`：要执行的命令。`ARG` 为该命令参数。



想要实现我们的效果，那么我们可以使用 `-it` 参数执行命令 `/bin/bash`，这样就起到了直接在容器内操作的作用：

```shell
docker exec -it e2019472ad98 /bin/bash
```

这样就会进入容器中的 `bash` 里，想推出的话键入 `exit` 即可。



> 如果是在本地系统进入此容器，`docker exec` 的方法是最方便的。
>
> 
>
> 但是如果我们想要进入不在本机上的容器，就要麻烦一点，可以使用 ssh 登录的方式：
>
> 1. 容器所在的本地系统使用 `docker exec` 进入容器，在其中安装并配置 `sshd` 服务。
> 2. 添加端口映射，将容器内 `sshd` 服务使用的端口（默认是 22）映射到容器所在本地系统的其他端口上。
> 3. 我们 ssh 登录时，只需要使用容器所在本地系统的地址，和映射容器内 22 的本地系统端口，就可以了。
>
> 这里就暂时先不说这个了，比较简单，记得 ssh 的包名叫 `openssh-server` 就行。

### 2.2. 分解操作





`docker run` 命令做了很多事，而大致也就是下面说的几个。

如果你只是和我一样的轻度使用，就是用 docker 偶尔做点小事，比如找个干净的环境编译代码，那么在创建和启动容器这方面，`docker run` 应该已经足够了，就不用看本小节了。

#### 2.2.1. 查找镜像





至于镜像从哪里来，官方有提供 [docker hub](https://hub.docker.com/search?q=&type=image)，我们可以在上面找到各种镜像、扩展和插件（这里暂不考虑第三方渠道或其他镜像源）。

不过一般除非有特定需求，不然没有必要去网站上找，使用命令 `docker search` 可以搜索镜像，其结果和 docker hub 上面的一致。

我这里以查找 `centos` 镜像为例：

```shell
$ sudo docker search centos
NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                       DEPRECATED; The official build of CentOS.       7420      [OK]       
kasmweb/centos-7-desktop                     CentOS 7 desktop for Kasm Workspaces            26                   
couchbase/centos7-systemd                    centos7-systemd images with additional debug…   5                    [OK]
dokken/centos-7                              CentOS 7 image for kitchen-dokken               4                    
dokken/centos-stream-8                                                                       3                    
continuumio/centos5_gcc5_base                                                                3                    
dokken/centos-stream-9                                                                       2                    
dokken/centos-8                              CentOS 8 image for kitchen-dokken               2                    
spack/centos7                                CentOS 7 with Spack preinstalled                1                    
spack/centos6                                CentOS 6 with Spack preinstalled                1                    
ustclug/centos                               Official CentOS Image with USTC Mirror          0                    
dokken/centos-6                              CentOS 6 image for kitchen-dokken               0                    
datadog/centos-i386                                                                          0                    
bitnami/centos-extras-base                                                                   0                    
couchbase/centos-72-java-sdk                                                                 0                    
corpusops/centos-bare                        https://github.com/corpusops/docker-images/     0                    
couchbase/centos-72-jenkins-core                                                             0                    
corpusops/centos                             centos corpusops baseimage                      0                    
couchbase/centos-70-sdk-build                                                                0                    
couchbase/centos-69-sdk-build                                                                0                    
couchbase/centos-69-sdk-nodevtoolset-build                                                   0                    
bitnami/centos-base-buildpack                Centos base compilation image                   0                    [OK]
fnndsc/centos-python3                        Source for a slim Centos-based Python3 image…   0                    [OK]
spack/centos-stream                                                                          0                    
dokken/centos-5                              EOL DISTRO: For use with kitchen-dokken, Bas…   0
```

可以看到输出了 25 个与关键词 "centos" 有关的条目，默认就是输出所有结果的前 25 条，并且结果按 `STARS` 降序。



具体命令使用方法如下：

```shell
docker search [OPTIONS] TERM
```

其中 `[OPTIONS]` 有：

* `-f` `--filter filter`：基于提供的条件筛选输出。
* `--format string`：使用一个 Go 模板来格式化输出。
* `--limit int`：显示的最大条目数量（默认为 25）。
* `--no-trunc`：不截断输出（一般是镜像的描述 `DESCRIPTION`）。



#### 2.2.2. 下载镜像



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
> `docker pull centos` 等价于 `docker pull centos:latest`。



下载默认的 `centos` 镜像：

```shell
docker pull centos
```

然后我们可以使用 `docker images` 命令查看我们已有的镜像：

```shell
$ sudo docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
centos     latest    39f63d416992   2 weeks ago     190MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB
```

可以看到我们机器上已经有了一个 TAG 为 `latest` 的 centos 镜像了。

下面就可以开始创建容器了。

#### 2.2.3. 创建容器

创建容器的命令如下：

```shell
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

or

docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

`docker create` 和 `docker container create` 等价，用哪个都行，这里就用短的前者了。

这个命令的 `[OPTIONS]` 太多了，这里只列出几个我比较常用的：

* `--name`：给该容器赋予一个名字。这个名字在很多关于该容器的操作中会用到，如果没有设定名字，会随机出一个。
* `-p` `--publish` ：将容器中的端口映射到本地系统的端口。最简单的用法是后接 `local_port:container_port`，例如 `-p 2222:22` 表示将容器中的 22 端口映射到本地系统的 2222 端口，就可以使用 2222 端口通过 ssh 登录此容器了。每个 `-p` 参数后只可以接一组映射。

命令中的其他值解释如下：

* `IMAGE`：镜像的仓库名字，可以附加 Tag 等信息。
* `COMMAND`：容器启动时要执行的命令，默认是 `/bin/bash`。该命令执行完毕后后容器就会退出。
* `ARG`：命令 `COMMAND` 的参数。



这里以我们前面下载的镜像 `centos:latest` 为例（加上 `latest` 是为了演示 tag 的用法，不加的话默认一般就是 `latest`）：

```shell
$ sudo docker create --name demo_centos centos:latest
cd0b2e6cefc3ca296181758992ca63c01484ea08c455e3a1a82377c66adab5fc
```

该命令会输出一个 64 位的 sha256 哈希值，这就表示容器创建成功了。

我们现在使用命令查看本机上的容器列表：

```shell
$ sudo docker container ls -a
CONTAINER ID   IMAGE              COMMAND       CREATED         STATUS    PORTS     NAMES
cd0b2e6cefc3   centos:latest   "/bin/bash"   2 minutes ago   Created             demo_centos
```

给命令 `docker container ls` 加上参数 `-`a 表示列出所有容器，否则只列出正在运行的容器，因为我们新建的容器还没有启动，不加 `-a` 是看不到的。

我们可以看到容器的 ID 就是刚刚输出的哈希值的前 12 位。







#### 2.2.4. 启动容器



在启动 docker 之前，我们需要重申一件事：

docker 容器内必须有一个前台进程正在运行，一旦没有正在运行的前台进程，这行容器就会被认为是空闲的，随即关闭。

我们知道有些 docker 镜像中是自带了服务的，比如 nginx，这样在容器启动时可以自动启动 nginx 服务，而由于该服务持续运行，该容器不会关闭。



启动容器命令是：

```shell
docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

这里的 `[OPTIONS]` 我最常用的只有一个 `-i`，就是开启交互模式。











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



### 3.3. 将现有容器打包成镜像



[Docker 将现有容器打包成镜像的方法](https://gukaifeng.cn/posts/docker-jiang-xian-you-rong-qi-da-bao-cheng-jing-xiang-de-fang-fa/)



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









## 4. 遇到过的问题





**问题1：在做很多事的时候都可能遇到**

```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

**解决方案1：**将此容器的 COMMAND 替换为 `/sbin/init`（而非 `/bin/bash`）。



