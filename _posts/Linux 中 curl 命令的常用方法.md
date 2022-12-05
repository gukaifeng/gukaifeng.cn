---
title: Linux 中 curl 命令的常用方法
date: 2021-12-09 18:02:33
updated: 2022-12-05 01:44:33
categories: [技术杂谈]
tags: [Linux,CentOS,curl]
toc: true
---







## 1. 什么是 `curl`？



`curl` 是一个使用 URL 语法传输数据的命令行工具和库。



`curl` 是开源的，项目地址为 https://github.com/curl/curl ，官网为 https://curl.se/ 。

0

`curl` 其支持的协议有 `DICT`、`FILE`、`FTP`、`FTPS`、`GOPHER`、`HTTP`、`HTTPS`、`IMAP`、`IMAPS`、`LDAP`、`LDAPS`、`MQTT`、`POP3`、`POP3S`、`RTMP`、`RTMPS`、`RTSP`、`SCP`、`SFTP`、`SMB`、`SMBS`、`SMTP`、`SMTPS`、`TELNET` 和 `TFTP`。



`curl` 命令被设计为无需用户交互即可工作。



`curl` 提供了大量有用的技巧，例如代理支持、用户身份验证、FTP 上传、HTTP post、SSL 连接、cookie、文件传输恢复等。`curl` 提供的功能的数量可能会让我们眼花缭乱头晕炫目。





## 2. `curl` 常用方法



>`curl` 的功能非常强大，非常多，本文只会介绍笔者认为比较常用的用法。详细文档请参阅 [man curl](https://man7.org/linux/man-pages/man1/curl.1.html) 和 [man curl-config](https://man7.org/linux/man-pages/man1/curl-config.1.html)。
>
>同样因为 `curl` 的功能实在是太多了，所以本文大概会偶尔追加更新一些我自己逐渐新用到的功能。



