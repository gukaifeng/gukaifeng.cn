---
title: Tomcat入门
mathjax: false
date: 2020-03-18 11:03:53
updated: 2020-03-18 11:03:53
tags: [Java,Tomcat,Servlet,服务器]
categories: [网络编程]
toc: true
---

## 1. 什么是服务器？

服务器就是代码编写的一个可以根据用户请求实时的调用执行对应的逻辑代码的一个容器。

服务器程序在普通用户看来就是一个安装在远程电脑上的应用程序。

我们只需要将服务器在操作系统上进行安装，并将我们事先编写好的逻辑处理代码根据规则放到服务器的指定位置，启动服务器，那么服务器就会自动根据接收到的请求调用并执行对象的逻辑代码进行处理。

<!--more-->

## 2. Tomcat

### 2.1. 安装与配置(MacOS)

1. 下载

    打开 [Apache Tomcat](http://tomcat.apache.org/) 官网，选择你需要的版本的 Core 下 的 tar.gz 进行下载。

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/Tomcat%E5%85%A5%E9%97%A8_1.png)

2. 解压

    把解压后的文件夹改名为 tomcat，放入 ~/Library/ 下。

3. 设置权限

    打开终端输入 `sudo chmod 755 ~/Library/tomcat/bin/*.sh` 设置权限。

4. 启动

    进入到 `~/Library/tomcat/bin/` 目录下，输入 `sudo sh startup.sh`。

    或直接在 ~ 目录输入 `sudo sh ~/Library/tomcat/bin/startup.sh`。

5. 验证是否启动

    在浏览器中输入 http://localhost:8080/，若能正常访问说明启动成功。

6. 停止

    进入到 `~/Library/tomcat/bin/` 目录下，输入 ` sh shutdown.sh`。

    或直接在 ~ 目录下输入 `sh ~/Library/tomcat/bin/shutdown.sh`。





### 2.2. Tomcat目录结构介绍



| 目录     | 存放内容                       |
| -------- | ------------------------------ |
| \bin     | 启动和关闭 tomcat 的可执行文件 |
| \conf    | tomcat 的配置文件              |
| \lib     | 库文件                         |
| \logs    | 日志文件                       |
| \temp    | 临时文件                       |
| \webapps | web 应用                       |
| \work    | JSP 转换后的 Servlet 文件      |