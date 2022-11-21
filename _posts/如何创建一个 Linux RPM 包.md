---
title: 如何创建一个 Linux RPM 包
date: 2022-11-14 01:40:30
updated: 2022-11-22 01:48:30
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

一个 RPM 包由一个 `.spce` 文件定义。`.spec` 文件的语法很严格，不过 `rpmdev` 给我们提供了样板。

```shell
rpmdev-newspec hello
```

这句命令会生成一个名为 `hello.spec` 的文件，我们得把这个文件移动到 `SPEC` 目录中。

`hello.spec` 的初始内容像下面这样：
```spec
Name:           hello
Version:        
Release:        1%{?dist}
Summary:        

License:        
URL:            
Source0:        

BuildRequires:  
Requires:       

%description


%prep
%autosetup


%build
%configure
%make_build


%install
rm -rf $RPM_BUILD_ROOT
%make_install


%files
%license add-license-file-here
%doc add-docs-here



%changelog
* Tue Nov 15 2022 gukaifeng <892859816@qq.com>
- 
```

运行 `tree ~/rpmbuild` 可以看到我们当前的目录结构像线面这样：

```
/home/gukaifeng/rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
│   └── hello-0.0.1.tar.gz
├── SPECS
│   └── hello.spec
└── SRPMS

5 directories, 2 files
```


生成的 `.spec` 文件给我们提供了一个不错的起点，但是其中没有指定任何有关我们要构建的项目的信息。这个生成的 `.spec` 文件假定我们要编译、构建软件。


我们要打包的是一个 Bash 脚本，所以我们要做的很简单。对于这个例子，我们不需要 **BUILD** 过程，因为我们没有代码需要编译。我们还可以添加 **BuildArch: noarch**（表示这个包不区分 CPU 架构），因为这个包在 32 位、64 位，ARM 或任何其他能运行 Bash 的 CPU 架构都可以用。

我们还需要添加 **Requires: bash**，这样这个包就会确保 Bash 已经安装。我们示例中这个简单的 "hello world" 脚本可以运行在任何 shell 中，但不是所有的脚本都可以，所以声明依赖是很有必要的。

我们现在补充上述 `hello.spec` 文件，如下：

```spec
Name:           hello
Version:        0.0.1
Release:        1%{?dist}
Summary:        A simple hello world script
BuildArch:      noarch

License:        GPL
Source0:        %{name}-%{version}.tar.gz

Requires:       bash

%description
A demo RPM build

%prep
%setup -q

%install
rm -rf $RPM_BUILD_ROOT
mkdir -p $RPM_BUILD_ROOT/%{_bindir}
cp %{name}.sh $RPM_BUILD_ROOT/%{_bindir}

%clean
rm -rf $RPM_BUILD_ROOT

%files
%{_bindir}/%{name}.sh

%changelog

* Tue Nov 15 2022 gukaifeng <892859816@qq.com> - 0.0.1
- First version being packaged
```

我们可以看到 `.spec` 文件中有很多**宏**，例如 `%{name}`、`%{version}`。在 `.spec` 文件中使用宏是非常重要的，宏可以帮助我们在所有 RPM 系统中确保一致性，避免文件名、版本号出错，并且我们打包的程序要更新版本的话，使用了宏的 `.spec` 文件的更新要容易一些。更多宏相关的内容可以参考 [Fedora packaging documentation](https://docs.fedoraproject.org/en-US/packaging-guidelines/RPMMacros/)。

例如，我们需要明确指定哪些文件要安装在 **%files** 部分下，我们有明确写法如下：

```spec
%files
%{_bindir}/%{name}.sh
```

这里表示我们要把脚本放在 **%{_bindir}** 目录下，宏 **%{_bindir}** 的默认值为 `/usr/bin`（也可以配置到其他位置，比如 `/usr/local/bin`）。我们可以运行以下命令查看宏的值：
```shell
$ rpm --eval '%{_bindir}'
/usr/bin
```

这里再列出几个其他常用的宏：

* `%{name}`：包的名字（由 `.spec` 文件中的 `Name` 域定义）。
* `%{version}`：包的版本（由 `.spec` 文件中的 `Version` 域定义）。
* `%{_datadir}`：共享数据目录（默认为 `/usr/sbin`）。
* `%{_sysconfdir}`：配置目录（默认为 `/etc`）。


### 2.4. 检查 `.spec` 文件中的错误（rpmlint）

`rpmlint` 命令可以用找出 `.spec` 文件中的错误：

```shell
$ rpmlint ~/rpmbuild/SPECS/hello.spec
/home/gukaifeng/rpmbuild/SPECS/hello.spec: W: no-%build-section
/home/gukaifeng/rpmbuild/SPECS/hello.spec: W: invalid-url Source0: hello-0.0.1.tar.gz
0 packages and 1 specfiles checked; 0 errors, 2 warnings.
```

没有 errors，有两个 warnings。  
第一个 warning 是说我们没有写 build 部分，我们这个脚本程序不需要这个，所以不用管。  
第二个 warning 是因为 `hello-0.0.1.tar.gz` 是一个本地文件，没有网络 URL。

这两个 warning 都可以忽略，所以目前我们的 `.spec` 文件没有问题。

### 2.5. 构建包（rpmbuild）

