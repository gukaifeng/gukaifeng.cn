---
title: macOS 下使用 VS Code 配置远程开发工具
mathjax: false
date: 2020-12-31 13:39:49
updated: 2020-12-31 13:39:49
tags: [macOS,SSH, VSCode]
categories: [技术杂谈]
toc: true
---

因为本人正在使用 macOS 系统，最近打算做 C/C++ 服务器开发，但是毕竟 macOS 和 Linux 的 API 还是不一样的，所以在 macOS 完成开发的全部工作是不合适的。本打算用虚拟机装 Linux 进行开发，但尝试了一点时间后，体验很差，所以决定换个方式。最终决定，使用 VS Code 在本地编写代码，再传到服务器中编译运行。

## 1. 安装 Remote-SSH 插件
在 VS Code 扩展中搜索 Remote-SSH 插件并安装，这是一个 VS Code 提供的远程连接工具。

## 2. 配置 Remote-SSH 插件
安装好 Remote-SSH 插件 以后，在 VS Code 左栏，扩展图标的上面，会多出一个 Remote Explorer 图标。点击这个新图标，点击 SSH TARGETS 右侧的齿轮状设置按钮，选择第一个 `/User/gukaifeng/.ssh/config`，这里 `gukaifeng` 是我本机的用户名，大家的会有自己的用户名。点开后，在 config 文件中写入一下内容:
```config
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host tencentCloud
    HostName 49.232.2.120
    User root
```
* Host：这里随便写，这只是一个此 SSH 链接的标识。我写的是 tencentCloud。
* HostName：这里写你的服务器 IP 地址。我写的是我的服务器地址 49.232.2.120。
* User：这里写要登录的用户。我以 root 用户登录。

## 3. 连接远程服务器


* 配置完成后，在 SSH TARGETS 就会有一个新的 SSH 链接了，名字就是我们刚刚起的 tencentCloud。
* 点击右边的小图标（Connect to Host in New Window），会打开一个新的 VS Code 窗口。
* 在新窗口中，会提示你输入服务器密码，完成以后，就可以连接服务器了。到这里，新窗口中的 VS Code 就是工作在远程服务器的了，可以尝试打开文件夹或文件，看看是不是都是服务器上的文件呢？大功告成！
-
* 另外注意，在这个新窗口中的 VS Code 与我们本地的 VS Code 是相对独立的，我们在 SSH 或本地的 VS Code 中安装插件，是不会影响另一个的。


## 4. 配置 SSH 免密登录（可选）
按照上面的配置步骤，每次连接到远程服务器，都需要输入服务器登录密码，比较麻烦。
我们这里可以配置 SSH 免密登录，具体操作查看 [macOS 下配置 SSH 免密登录](https://gukaifeng.me/2020/12/31/macOS-%E4%B8%8B%E9%85%8D%E7%BD%AE-SSH-%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95/)。