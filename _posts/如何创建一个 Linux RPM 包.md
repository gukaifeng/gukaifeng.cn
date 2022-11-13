---
title: 如何创建一个 Linux RPM 包
date: 2022-11-14 01:40:30
updated: 2022-11-14 01:40:30
categories: [技术杂谈]
tags: [RPM]
---




这篇文章包含以下内容：

1. 什么是 RPM 包。
2. 如何创建一个 RPM 包。
3. 如何安装(install)、查询(query)、移除(remove)一个 RPM 包。


## 1. 什么是 RPM 包？

RPM 全称 *Red Hat Package Manager*，即红帽包管理器，这是一个由 Red Hat 开发，主要用在基于红帽的操作系统上的（如 Fedora、CentOS、RHEL 等）。

RPM 包使用 `.rpm` 扩展名，是一个不同文件的捆绑包（一个集合），其可以包含以下内容：
* 二进制文件，也就是我们常说的可执行文件（如 `nmap`、`stat`、`xattr`、`ssh`、`sshd` 等）。
* 配置文件（如 `sshd.conf`、`updatedb.conf`，`logrotate.conf` 等）。
* 文档文件（如 `README`、`TODO`、`AUTHOR` 等）。

RPM 包的文件名格式如下：
```
<name>-<version>-<release>.<arch>.rpm
```
例如：
```
bdsync-0.11.1-1.x86_64.rpm
```
一些软件包还包括其构建的发行版的速记版，像下面这样：
```
bdsync-0.11.1-1.el8.x86_64.rpm
```


## 2. 如何创建一个 RPM 包？

在构建一个 RPM 包前，我们需要准备以下内容：

* 一个运行基于 RPM 的分布（如 RHEL 或 Fedora）的工作站或的虚拟机。
* 要创建 RPM 包的软件。
* 这个包的源码。
* 用于构建此 RPM 包的 `SPEC` 文件。

### 2.1. 安装需要的软件

要构建 RPM 包，首先需要安装下面的软件：
```shell
sudo dnf install -y rpmdevtools rpmlint
```

* `rpmdevtools`：顾名思义，是 rpm 的开发者工具。
* `rpmlint`：用于检查 rpm 软件包中的常见错误。

安装完 `rpmdevtools` 后，创建我们需要构建 RPM 包的文件树：

```shell
rpmdev-setuptree
```

如果你以非 root 用户构建 RPM 包，上述命令会在你的用户 home 目录中放置构建环境，即 `~/rpmbuild` 目录，其目录结构如下：
```
rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS
```

我们也可以验证这一点：
```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z ~]$ ll ~/rpmbuild/
total 0
drwxrwxr-x 2 gukaifeng gukaifeng 6 Nov 14 00:53 BUILD
drwxrwxr-x 2 gukaifeng gukaifeng 6 Nov 14 00:53 RPMS
drwxrwxr-x 2 gukaifeng gukaifeng 6 Nov 14 00:53 SOURCES
drwxrwxr-x 2 gukaifeng gukaifeng 6 Nov 14 00:53 SPECS
drwxrwxr-x 2 gukaifeng gukaifeng 6 Nov 14 00:53 SRPMS
```

* **BUILD** 目录：在构建 RPM 包期间使用，用于存放、移动临时文件等。
* **RPMS** 目录：存放构建好的 RPM 包。如果有在 `.sepc` 中或构建期间指定，其中也可能包含不同架构的包，或不区分架构的包，
* **SOURCES** 目录：顾名思义，包含源。源可以是一个脚本、一个需要编译的复杂的 C 项目、一个已经编译好的程序等等。通常，源会被压缩为 `.tar.gz` 或 `.tgz` 文件。
* **SEPC** 目录：包含 `.spec` 文件，这个 `.spec` 文件具体定义了一个包如何构建。我们后面会进一步说。
* **SRPMS** 目录：包含 `.src.rpm` 包，一个源 RPM 包不属于任何一个架构或分发。实际的 `.rpm` 包构建是基于这个 `.src.rpm` 包的。


`.src.rpm` 包是非常灵活的，其可以在所有其他基于 RPM 的分布和架构上构建和重建。

现在我们对每个目录是干什么的已经基本了解了，现在我们先创建一个简单的脚本用来分发。

```shell
$ cat << EOF >> hello.sh
#!/bin/sh
echo "Hello world"
EOF
```

这创建了一个名为 `hello.sh` 的简单 shell 脚本，其会在终端中打印 "Hello world"。这个脚本非常简单，但是对于演示 RPM 包的构建过程来说足够了。

### 2.2. 将脚本放到指定目录中

要为我们刚刚的脚本构建 RPM 包，我们必须把这个脚本放在 RPM 构建系统期望的位置。我们创建一个目录，目录名使用大部分项目都遵循的[语意版本控制(Semantic Versioning)](https://semver.org/)，然后将 `hello.sh` 放进去。

```shell
mkdir hello-0.0.1
mv hello.sh hello-0.0.1
```

大部分源码都是作为归档（压缩包）发布的，所以我们使用 `tar` 命令将我们的程序目录压缩。

```shell
tar --create --file hello-0.0.1.tar.gz hello-0.0.1
```

然后将此压缩包移到我们前面说过的 `SOURCES` 目录中。

```shell
mv hello-0.0.1.tar.gz SOURCES
```

### 2.3. 创建 `.spce` 文件