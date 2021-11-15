---
title: CentOS 8 在 root 和普通用户间切换的方法
mathjax: false
date: 2020-12-31 10:39:11
updated: 2020-12-31 10:39:11
tags: [Linux, CentOS]
categories: [技术杂谈]
toc: true
---


## 从普通用户切换到 root 用户

在终端中输入 `su`，然后输入密码，即可切换到 root 用户。

##  从 root 用户切换到普通用户
假定一个普通用户名为 "gukaifeng"，
则在终端中输入 `login -f gukaifeng`，即可切换到普通用户 "gukaifeng"。