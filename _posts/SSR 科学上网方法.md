---
title: SSR 科学上网方法
date: 2019-02-17
categories: [技术杂谈]
tags: [科学上网, SSR]
toc: true
---

# SSR 科学上网方法

### 1. 购买 VPS

VPS 选择谁家的都可以，例如搬瓦工和 Vultr，我使用的是 Vultr 的。  
系统建议选择 Debian，因为本文后面要使用的脚本在 Debian 上支持一键配置谷歌 BBR 加速。  
本篇文章重点在于配置 SSR，就不在购买 VPS 这里多码字了。

<!--more-->

注意，并不是所有的 IP 地址都可行的，我们在开始配置 SSR 之前，要确保我们 VPS 的 22 号端口为开放状态。  
如果 22 号端口是关闭的，那么我们就要摧毁并重建 VPS，直到获得一个 22 号端口为开放的 IP 地址。你可以用命令行的方式来判断某个端口是否打开，也可以使用一些网站工具来帮助你。这里分享几个可以进行端口扫描的网站：  
1\. https://www.yougetsignal.com/tools/open-ports/  
2\. http://coolaf.com/tool/port  
3\. http://tool.chinaz.com/update.html


### 2. 下载并执行脚本

#### 2.1. 使用SSH连接VPS

MacOS 使用终端，输入 `ssh root@xxx.xxx.xxx.xxx`，后面是你的 IP 地址，再输入密码即可。  
Windows 建议使用 XShell 工具，手动输入 IP 地址，用户名 "root" 和密码。

#### 2.2. 下载并运行SSR安装脚本

下载脚本，执行下面的命令：

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh
```

对下载好的脚本添加权限，并执行：

```bash
chmod +x ssr.sh
bash ssr.sh
```

### 3. 配置SSR

按脚本索引操作即可。

### 4. 配置谷歌BBR加速

按脚本索引操作即可。

### 5. 配置多用户（可选）

当有多个用户使用同一个 SSR 的时候，配置多用户会让我们管理起来更方便。

打开 SSR 的配置文档，在该脚本中，选择 "8. 手动 修改配置"，  
删除字段 `"server_port"` 和 `"password"`，添加新字段 `"port_password"`。  
新字段格式示例如下，前面为端口号，后面为该端口的密码，修改完成后，保存退出即可。

```bash
"port_password":{
    "1001":"cas^1a",
    "1002":"asu7(D",
    "1003":"(&^sac"
},
```