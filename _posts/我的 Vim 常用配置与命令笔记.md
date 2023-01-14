---
title: 我的 Vim 常用配置与命令笔记
date: 2022-11-10 18:47:20
updated: 2022-11-29 22:05:20
categories: [技术杂谈]
tags: [Vim]
---





首先说明呢，我并不是 vim 的深度用户，我只是经常 ssh 到远程机器上编辑一些文件什么的，这时候 vim 就顺手用了。



所以呢，这篇文章记录的也都是些简单的东西，vim 高级功能我也用不上。





## 1. vim 常用配置



记录我的常用配置呢，主要就是每当我登录一个新机器并且需要使用 vim 的时候，可以把这里的配置直接 copy 一份省时省力 ~



当然我还是会说明每个配置项是干啥的。



vim 的配置文件有两个，一个是全局的，在 `/etc/.vimrc`，一个是用户级的，在 `~/.vimrc`，用户级的优先级高。



编辑哪个都可以，这里就编辑用户级的 `~/.vimrc` 了（不过如果你有在非 root 用户下 `sudo vim` 编辑的需求的话，建议把 `/etc/.vimrc` 也改了）：



```shell
vim ~/.vimrc
```

(注意不管是全局的还是用户级的配置文件，如果默认不存在的话，直接通过 vim 自动创建并编辑保存就好了 ~)

前面的单引号表示注释 `"`。

```.vimrc
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936,latin1
set termencoding=utf-8
set encoding=utf-8
set nu
set cursorline
set autoindent
set tabstop=4
set expandtab
set mouse=a
syntax on
```

* `fileencodings`: Vim 打开或写入文件时采用的编码格式。按顺序查找，直到第一个可用的。

* `termencoding`: 输出到终端时采用的编码格式，即 Vim 工作的终端的编码格式。

* `encoding`: Vim 内部使用的字符编码方式，包括 Vim 的 buffer (缓冲区)、菜单文本、消息文本等。

* `nu`: 设置行号。

* `cursorline`: 重点显示当前行（当前行会有个下划线）。

* `autoindent`: 自动缩进。

* `tabstop`: 设置 tab 的宽度（相当于几个空格）。

* `expandtab`: 自动将 tab 转换成对应数量的空格。

* `mouse`: 启用鼠标。

* `syntax on`: 开启语法高亮。

  

  

## 2. vim 常用命令



### 2.1. 查找与替换





### 2.2. 多行修改

1. 在第 1 行要修改的地方，按下快捷键 `ctrl+v` 进入可视化模式；
2. 按方向键下（或 `J` 键）上（或 `K` 键）多选要修改的行，注意这里只能垂直选择一列，如果你同时左右选了一块矩形的话是不行的；
3. 按大写字母 `I`，`shift+i` 或直接按大写锁定后的 `I` 都可以，此时光标会回到第 1 行我们选择的位置；
4. 在第 1 行的光标处修改，注意这里暂时只有第 1 行会修改，其他刚刚选中的行不会变化；
5. 修改完成后，按 `ESC` 键退出编辑，全部选择的行会自动应用第 1 行的修改，也可能会有短暂延迟。到此修改完成。

