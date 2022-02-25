---
title: 预览或引用 GitHub 仓库中资源的方法
date: 2022-01-14 16:54:55
updated: 2022-01-14 16:54:55
categories: [技术杂谈]
tags: [GitHub]
---



目前我已知的，只有 html 文件预览，和图片资源的预览。

其中 html 的预览可以加载 css 等资源（js 本人不确定，因为我不是做前端的），

图片资源的预览链接可以直接作为图片 url 来使用的。



### 1. html 预览

[http://htmlpreview.github.io/](http://htmlpreview.github.io/)



### 2. 图片预览

预览链接以 `https://raw.githubusercontent.com/` 开头，具体格式如下：

```
https://raw.githubusercontent.com/用户名/仓库名/分支/资源路径
```

-

举个例子，假如我（GitHub 用户名为 `gukaifeng`）有一张图片，在仓库 `example-repo` 的 `main` 分支中的 `/img/` 路径下，名字为 `example.png`，即该图片在 GitHub 上的全路径应为：

```
https://github.com/gukaifeng/example-repo/blob/main/img/example.png
```

那么这张图片的预览路径为：

```
https://raw.githubusercontent.com/gukaifeng/example-repo/main/img/example.png
```



<!--more-->

