---
title: 使用 PicGo + GitHub 搭建个人免费图床
mathjax: false
date: 2020-12-30 09:36:29
updated: 2020-12-30 09:36:29
tags: [GitHub, PicGo, 图床]
categories: [技术杂谈]
toc: true
---
PicGo 是一个用于快速上传图片并获取图片 URL 链接的工具。
PicGo 的 Github 地址为 https://github.com/Molunerfinn/PicGo

* PicGo 支持 Windows、macOS 和 Linux。
* PicGO 支持七牛图床、腾讯云 COS、又拍云、GitHub、SM.MS、阿里云 OSS、Imgur 共8种图床。

[点击此处下载 PicGo应用](https://github.com/Molunerfinn/PicGo/releases)

这篇文章不在赘述 PicGo 更多的细节信息，只记录如何使用，关于 PicGo 更多信息，可以参考其 Github。
<!--more-->

## 创建一个新的 GitHub 仓库

创建仓库比较基础，这里只需要注意一下，权限要选择 `public`。

## 新建一个 GitHub 个人访问令牌

在 GitHub 页面 Settings -> Developer settings -> [Personal access tokens](https://github.com/settings/tokens) 。
点击 Generate new token 新建一个个人访问令牌。

* `Note` 处自己简单写一下这个令牌是干嘛的，做个标记就可以。
* 下面的 Select scopes 多选框，把 `repo` 一栏全部打钩，其他留空就可以。

创建好令牌后，把令牌保存好备用。

## 配置 PicGo 客户端

打开 PicGo 客户端，图床设置 -> GitHub 图床，填写相关信息。

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E4%BD%BF%E7%94%A8%20PicGo%20%2B%20GitHub%20%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%85%8D%E8%B4%B9%E5%9B%BE%E5%BA%8A_1.png)

* 设定仓库名：填 GitHub 上刚建的那个库，注意从 username 开始写。
* 设定分支名：一般就写 master。
* 设定Token：把刚刚的个人访问令牌填在这里。
* 指定存储路径：建议写 /img
* 设定自定义域名：如果有就写，没有可以留空。

点击确定，然后自己在把 GitHub 图床设定为默认图床。

## 开始使用

* 使用非常简单，可以自己上传，也会自动识别剪贴板的内容。
* 上传后的文件，就会出现在 GitHub 中我们刚刚的建的那个库中的 img 文件夹里了。
* 我们可以在相册中找到我们上传的图片，并且可以一键复制 Markdown 链接，插入到博客中非常方便。
-
* 建议再 PicGo 设置中打开上传前重命名，在上传前修改图片名，避免图片多了以后混淆。