---
title: "VSCode 在编辑器中显示垂直标尺"
date: 2023-04-30 11:48:00
updated: 2023-04-30 11:48:00
categories: [技术杂谈]
tags: [VSCode]
---





我们知道一行代码不宜过长，不同的公司、团队、项目通常有不同的规范。



最常见的行最大字符数的规范比如：

* 每行字符数最大不得超过 80。
* 每行字符数最大不得超过 120。
* 每行建议不超过 80 字符，最大不得超过 120 字符。
* ......





VSCode 支持设定垂直标尺，即在编辑器中，在指定字符数的位置给你画一条竖线。



例如下面这样，就是分别在 80 字符处和 120 字符处分别花了一条竖线，这样方便我们在写代码的时候控制每行字符数。



![1](D:\Profession\hexo\_posts\VSCode 在编辑器中显示垂直标尺\1.png)





设置方法也很简单，我们依次点击 File -> Perferences -> Setting：



![2](D:\Profession\hexo\_posts\VSCode 在编辑器中显示垂直标尺\2.png)



然后在上面的搜索栏里搜索 "Editor.rulers"：



![3](D:\Profession\hexo\_posts\VSCode 在编辑器中显示垂直标尺\3.png)

点击 "Edit in setting.json"，在里面修改 `editor.rulers` 字段，像下面这样就是在 80 和 120 字符处显示垂直标尺：



![4](D:\Profession\hexo\_posts\VSCode 在编辑器中显示垂直标尺\4.png)



然后保存就 OK 了。
