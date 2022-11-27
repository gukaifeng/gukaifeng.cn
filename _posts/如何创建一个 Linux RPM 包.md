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

>本文会介绍较为常用的相关内容，应当足以帮助大部分人入门，并支持大部分一般场景。  
>如果你有本文没有提到的、更复杂的需求，建议参考官方指导：[RPM Packaging Guide](https://rpm-packaging-guide.github.io/)。

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


### 2.2. 构建文件树

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

### 2.3. 将脚本放到指定目录中

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

### 2.4. 创建 `.spce` 文件

>PS：由于 `.spec` 文件的配置是制作 RPM 包过程中最复杂的部分，对初次接触的人来说，一下接受太多内容是很难的，所以本小节只是个样例，没有做太多相关项的解释。更详尽更复杂的相关内容，我们会在第 6 小节以后进一步解释。

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


### 2.5. 检查 `.spec` 文件中的错误（rpmlint）

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

### 2.6. 构建包（rpmbuild）

构建 RPM 包需要用到 rpmbuild 命令。在本文早前的 2.2 小节，我们提到过 `.src.rpm`（源 RPM 包） 和 `.rpm` 包的区别。

**（可选）** 使用下面的命令创建 `.src.rpm` 包：

```shell
$ rpmbuild -bs ~/rpmbuild/SPECS/hello.spec
Wrote: /home/gukaifeng/rpmbuild/SRPMS/hello-0.0.1-1.el8.src.rpm
```

其中的参数 `-bs` 表示：
* `-b`: build
* `-s`: source

**（必须）** 使用下面的命令创建二进制 `.rpm` 包：

```shell
$ rpmbuild -bb ~/rpmbuild/SPECS/hello.spec
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.6Zwexi
+ umask 022
+ cd /home/gukaifeng/rpmbuild/BUILD
+ cd /home/gukaifeng/rpmbuild/BUILD
+ rm -rf hello-0.0.1
+ /usr/bin/tar -xof /home/gukaifeng/rpmbuild/SOURCES/hello-0.0.1.tar.gz
+ cd hello-0.0.1
+ /usr/bin/chmod -Rf a+rX,u+w,g-w,o-w .
+ exit 0
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.AhF9fh
+ umask 022
+ cd /home/gukaifeng/rpmbuild/BUILD
+ '[' /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64 '!=' / ']'
+ rm -rf /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
++ dirname /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
+ mkdir -p /home/gukaifeng/rpmbuild/BUILDROOT
+ mkdir /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
+ cd hello-0.0.1
+ rm -rf /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
+ mkdir -p /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64//usr/bin
+ cp hello.sh /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64//usr/bin
+ '[' noarch = noarch ']'
+ case "${QA_CHECK_RPATHS:-}" in
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-ldconfig
/sbin/ldconfig: Warning: ignoring configuration file that cannot be opened: /etc/ld.so.conf: No such file or directory
+ /usr/lib/rpm/brp-compress
+ /usr/lib/rpm/brp-strip /usr/bin/strip
+ /usr/lib/rpm/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
+ /usr/lib/rpm/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile '' 1
+ /usr/lib/rpm/brp-python-hardlink
+ PYTHON3=/usr/libexec/platform-python
+ /usr/lib/rpm/redhat/brp-mangle-shebangs
Processing files: hello-0.0.1-1.el8.noarch
Provides: hello = 0.0.1-1.el8
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Checking for unpackaged file(s): /usr/lib/rpm/check-files /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
Wrote: /home/gukaifeng/rpmbuild/RPMS/noarch/hello-0.0.1-1.el8.noarch.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.7Gxbkj
+ umask 022
+ cd /home/gukaifeng/rpmbuild/BUILD
+ cd hello-0.0.1
+ rm -rf /home/gukaifeng/rpmbuild/BUILDROOT/hello-0.0.1-1.el8.x86_64
+ exit 0
```

其中的参数 `-bb` 表示：
* `-b`: build
* `-b`: binary


也可以使用 `-ba` 同时创建 `.src` 和二进制 rpm 包。

构建过程完成后，我们的目录结构应当是下面这样的（如果没有 `-bs` 或者 `-ba` 的话，只用了 `-bb`，那么就没有 **SRPMS** 目录中的内容，别的都一样）：

```
$ tree ~/rpmbuild/
/home/gukaifeng/rpmbuild/
├── BUILD
│   └── hello-0.0.1
│       └── hello.sh
├── BUILDROOT
├── RPMS
│   └── noarch
│       └── hello-0.0.1-1.el8.noarch.rpm
├── SOURCES
│   └── hello-0.0.1.tar.gz
├── SPECS
│   └── hello.spec
└── SRPMS
    └── hello-0.0.1-1.el8.src.rpm

8 directories, 5 files
```

到这里，我们的包就构建完了，其中 **RPMS** 目录下的 noarch 目录表示其中的包是不区分 CPU 架构的，其内的 `hello-0.0.1-1.el8.noarch.rpm` 就是我们最终打包好的 RPM 包。

>这一小节讲的其实是非常基础的 RPM 打包过程，有一些细节也没有说的足够详细（比如 .spec 文件一些项和值的含义没有说），我们先借此熟悉流程，然后会在后面介绍更复杂的场景。


## 3. 安装我们自己的 RPM 包

上面小节已经成功构建了包含我们 `hello.sh` 脚本的包 `hello-0.0.1-1.el8.noarch.rpm`。

现在我们可以通过 `dnf` 命令安装了：

```shell
$ sudo dnf install ~/rpmbuild/RPMS/noarch/hello-0.0.1-1.el8.noarch.rpm
[sudo] password for gukaifeng: 
Last metadata expiration check: 0:24:52 ago on Wed 23 Nov 2022 12:06:23 AM CST.
Dependencies resolved.
=========================================================================================================
 Package              Architecture          Version                    Repository                   Size
=========================================================================================================
Installing:
 hello                noarch                0.0.1-1.el8                @commandline                6.5 k

Transaction Summary
=========================================================================================================
Install  1 Package

Total size: 6.5 k
Installed size: 29  
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                 1/1 
  Installing       : hello-0.0.1-1.el8.noarch                                                        1/1 
  Verifying        : hello-0.0.1-1.el8.noarch                                                        1/1 

Installed:
  hello-0.0.1-1.el8.noarch                                                                               

Complete!
```

相当熟悉的界面！到这里就安装完成了！

当然你也可以直接使用 `rpm` 命令安装，像下面这样：

```shell
sudo rpm -ivh ~/rpmbuild/RPMS/noarch/hello-0.0.1-1.el8.noarch.rpm
```

这里不做进一步解释了，使用包管理器 `dnf` 安装和直接使用 `rpm` 命令安装的区别不是本文重点。


## 4. 验证我们的包是否已被安装

我们从两个方面来验证，我自己把这两个方面归为理论和实际。

\-

**理论方面**就是我们通过 rpm 的查询命令，来查看我们的包是否已被安装以及其他相关信息：

```shell
$ rpm -qi hello
Name        : hello
Version     : 0.0.1
Release     : 1.el8
Architecture: noarch
Install Date: Wed 23 Nov 2022 12:31:39 AM CST
Group       : Unspecified
Size        : 29
License     : GPL
Signature   : (none)
Source RPM  : hello-0.0.1-1.el8.src.rpm
Build Date  : Wed 23 Nov 2022 12:30:57 AM CST
Build Host  : iZ8vbf7xcuoq7ug1e7hjk5Z
Relocations : (not relocatable)
Summary     : A simple hello world script
Description :
A demo RPM build
```

可以看到，`rpm -qi` 命令查到了我们的 hello 包已经成功安装，并打印了一些相关信息。

我们在 `.spec` 文件中写的 changelog 也可以查看：

```shell
$ rpm -q hello --changelog
* Tue Nov 15 2022 gukaifeng <892859816@qq.com> - 0.0.1
- First version being packaged
```

我们还可以查看一下这个包内都有些什么：

```shell
$ rpm -ql hello
/usr/bin/hello.sh
```

可以看到我们的包中只有一个文件 `/usr/bin/hello.sh`，这也是我们期望中的。

\-

**实际方面**就是，不使用 rpm 相关的命令。我们安装一个包后，最重要的，就是使用它，所以我这里的实际方面，就是我们实际使用这个包试试看，如果能用，就说明包安装成功了。

我们可以用下看看，不过要注意我们的只是个脚本，并不是一个直接的可执行文件，所以执行的时候像下面这样：

```shell
$ sh hello.sh
Hello world
```

**注意哦，我执行上面 `bash hello.sh` 的目录中是没有 `hello.sh` 文件的，所以 `bash` 实际执行的是我们安装在 `/usr/bin` 目录下的那个！**


## 5. 移除我们安装的 RPM 包

同样的，我们可以通过包管理器 dnf 删除：

```shell
$ sudo dnf remove hello
[sudo] password for gukaifeng: 
Dependencies resolved.
=========================================================================================================
 Package             Architecture         Version                      Repository                   Size
=========================================================================================================
Removing:
 hello               noarch               0.0.1-1.el8                  @@commandline                29  

Transaction Summary
=========================================================================================================
Remove  1 Package

Freed space: 29  
Is this ok [y/N]: y
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                 1/1 
  Erasing          : hello-0.0.1-1.el8.noarch                                                        1/1 
  Verifying        : hello-0.0.1-1.el8.noarch                                                        1/1 

Removed:
  hello-0.0.1-1.el8.noarch                                                                               

Complete!
```

也可以通过 rpm 命令直接删除：

```shell
sudo rpm --verbose --erase hello
```

移除成功以后，我们在上一节的实际方面的验证就无法再使用了：

```shell
$ sh hello.sh
sh: hello.sh: No such file or directory
```

因为我们的包已经被卸载了嘛。

## 6. 进阶：配置更完整复杂的 `.spec` 文件

`.spec` 文件中描述了 `rpmbuild` 实际构建一个 RPM 包的具体信息。

`.spec` 文件主要由两个部分组成：**Preamble** 和 **Body**：

* **Preamble** 部分包含了一系列元数据项，这些数据项会用在 body 部分中。

* **Body** 部分包含整个构建的主要信息。



### 6.1. Preamble

下表列出了 RPM SPEC 文件中在 Preamble 部分可使用的项。

| Preamble 指示符 | 定义                                                         |
| --------------- | ------------------------------------------------------------ |
| `Name`          | 包的基本名称，需要和 SPEC 文件的名字一致。                   |
| `Version`       | 软件的上游版本号。                                           |
| `Release`       | 软件当前版本已经发布(release)的次数。通常我们设置初始值为 `1%{?dist}`，其中前面的 1 是值，后面的 `%{?dist}` 是一个宏。每当有新 release 的时候把这个值加 1；当软件有新版本的时候，把这个值重设回 1。关于宏 `%{?dist}` 我们在本小节最后来说。 |
| `Summary`       | 摘要，关于这个软件包的一行概述。                             |
| `License`       | 要打包的软件的许可。比如如果是开源软件的话，这里应该写上此软件包遵循的开源许可协议。 |
| `URL`           | 关于打包的软件的更多信息的完整 URL。大多时候要打包的软件的上游项目网址。 |
| `Source0`       | 上游源码压缩后的归档文件（没有打补丁的，补丁在其他地方处理）的路径或者 URL。这指向的应该是这个归档文件的一个可达可靠的存储，比如把归档文件放在上游页面，而不是本地存储。如果有需要的话，也可能增加更多 Source 项，比如 `Source1`、`Source2`、`Source3` ... `SourceX` 等等。 |
| `Patch0`        | 要应用到源码的第一个补丁的名字,没有补丁可以为空。和 Source 项一样，可以有更多 `Patch1`、`Patch2`、`Patch3` ... `PatchX` 等。 |
| `BuildArch`     | 运行此软件所依赖的处理器架构。如果这个软件不依赖某个具体架构，比如这个软件完全由一个解释型语言编写，那就可以设置为 `BuildArch: noarch`。如果不写的话，这个软件会自动继承构建此包的机器的架构，比如 `x86_64`。 |
| `BuildRequires` | 构建以编译语言编写的程序所需的包列表，包名之间用逗号或空格分隔。可以有多个 `BuildRequires` 条目，每个条目在 SPEC 文件中都有自己的行。 |
| `Requires`      | 安装后，运行软件所需的包列表，包名之间用逗号或空格分隔。可以有多个 `BuildRequires` 条目，每个条目在 SPEC 文件中都有自己的行。 |
| `ExcludeArch`   | 如果软件无法在特定的处理器体系结构上运行，则可以在此处排除该体系结构。 |

`Name`、`Version`、`Release` 三个指示符共同构成了 RPM 包的文件名。RPM 包管理者和系统管理员通常可以叫这三个指示符为 **N-V-R** 或者 **NVR**，因为 RPM 包文件名的格式就是 `NAME-VERSION-RELEASE`。

我们可以通过其他包名举一个例子：

```shell
$ rpm -q git
git-2.27.0-1.el8.x86_64
```

我们以 git 包为例，名字构成中，`Name` 是 "git"，`Version` 是 "2.27.0"，`Release` 是 "1.el8"。  
这里我们可能会认为与刚说的 NVR 不同，但其实是一样的，只是最后的 "x86_64" 不是由我们制作包的程序员直接控制的，其由 `rpmbuild` 的构建环境定义。不依赖构建环境的例外情况是 `noarch` 的包。



\-



`%{?dist}` 宏是很常见的，其表示分发标签(distribution tag)，其表示了我们正在构建的分发。

例如：

```shell
# 在 RHEL 8.X 的机器上，比如 CentOS 8.2

$ rpm --eval %{?dist}
.el8
```

```shell
# 在 Fedora 23 机器上

$ rpm --eval %{?dist}
.fc23
```



### 6.2. Body

Body 部分的指示符均已 `%` 开头。

下表列出了 RPM SPEC 文件中在 Body 部分使用的项。

| Body 指示符    | 定义                                                         |
| -------------- | ------------------------------------------------------------ |
| `%description` | RPM 中包装的软件的完整描述。此描述可以跨越多行，可以分段落。 |
| `%prep`        | 在构建的软件前需要执行的命令或一系列命令。例如，解压 `Source0` 中的归档。该指示符可以包含 shell 脚本。 |
| `%build`       | 将软件实际构建到机器码（用于编译语言）或字节码（对于某些解释的语言）中的命令或一系列命令。 |
| `%install`     | 从 `％builddir`（构建发生的地方）复制所需构建文件的命令或多个命令到 `％buildroot` 目录（其中包含带有要包装文件的目录结构）。这通常意味着将文件从 `〜/rpmbuild/build` 复制到 `〜/rpmbuild/buildroot`，并在 `〜/rpmbuild/buildroot` 中创建必要的目录。这仅在创建软件包时运行，而不是当终端用户安装软件包时运行。有关更详细的信息见 [Working with SPEC files](https://rpm-packaging-guide.github.io/#working-with-spec-files)。 |
| `%check`       | 测试软件的命令或一系列命令。通常包括单位测试之类的内容。     |
| `%files`       | 最将要安装在终端用户系统中的文件列表。                       |
| `%changelog`   | 发生在不同 `Version` 或 `Release` 构建之间的改动记录         |



### 6.3 更高级的项

SPEC 文件还可以包含高级的项。

例如，规格文件可以具有*脚本段(scriptlets)*和*触发器(triggers)*，它们可以在终端用户的安装过程中的某个点触发（而不是我们创建包的构建过程）。

详见 [Scriptlets and Triggers](https://rpm-packaging-guide.github.io/#triggers-and-scriptlets)。

## 7. 进阶：打包需要编译的程序案例

我们前面的入门案例打包的是一个 shell 脚本。因为 shell 脚本不需要编译，所以省下了很多操作。

现在我们试着制作一个需要编译的入门案例，我们写一个简单地 C++ 程序。



