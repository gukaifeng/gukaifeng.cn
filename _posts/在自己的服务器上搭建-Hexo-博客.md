---
title: 在自己的服务器上搭建 Hexo 博客
date: 2021-11-07 00:58:32
updated: 2021-11-07 00:58:32
categories: [技术杂谈]
tags: [Hexo, Blog]
toc: true
---

以前用 Hexo 是搭载 GitHub Page 上的，但是确实稳定性很迷。
于是尝试在自己的服务器上搭建 Hexo 博客。

在自己的服务器上搭建 Hexo 博客流程，主要分为两部分，**服务器配置**和**本地 Hexo 配置**。

下面开始！

<!--more-->

## 1. 服务器配置

本文的服务器系统环境为 CentOS 8.2。  
你的服务器需要已经安装好 Nginx 与 Git，这个过程不是本文重点，就不说了。

服务器配置主要要做下面几件事：

1. 新建一个 Linux 用户，并赋予 root 权限；
2. 创建一个 Hexo 的 Git 仓库目录；
3. 创建一个 Hexo 的工作目录，也是 Nginx 提供 Web 服务的根目录；
4. 配置 Git Hook（这个在 3.3 节说）；
5. 配置并启动 Nginx。

这里再简单解释下，第 1 条中 Hexo 的 Git 仓库，存储的是 Git 仓库信息，不是最终的仓库。第 2 条中的工作目录，就是真正显示在网页中的根目录，即有 `index.html` 的那个目录，在 Github Page 上搭建 Hexo 博客时，你的库目录与这个等价。

下面开始一个个步骤说。

### 1.1. 新建一个 Linux 用户

由于我们不打算在 root 用户直接操作（危险），所以我们新建一个用户 hexo，在这个 hexo 用户下操作。

假定你现在在以 root 登录 Linux 服务器。

依次输入命令：


{% codeblock "Shell" lang:shell %}
adduser hexo  # 新建一个名为 hexo 的用户
{% endcodeblock %}

```shell
passwd hexo  # 为 hexo 用户设置密码
```

然后再依次输入以下命令：

```shell
chmod +w /etc/sudoers  # 给权限配置文件添加写权限
```

```shell
vim /etc/sudoers  # 编辑权限配置文件
```

找到下面这段代码

```shell
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
```

仿照这个格式，为我们的 hexo 用户添加一行，即：

```shell
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
hexo    ALL=(ALL)       ALL
```

保存退出后，执行命令：

```shell
chmod -w /etc/sudoers  # 记得移除权限配置文件的写权限
```

然后，使用刚新建的用户 hexo，重新登录主机。

后面所有的步骤，都是在 hexo 用户下操作的。




### 1.2. 创建一个 Hexo 的 Git 仓库目录

这个 Git 仓库的位置可以是任意的，我这里直接放在了 hexo 用户根目录下面。

```shell
git init --bare ~/hexo.git  # 新建一个名为 hexo.git 的空库，我这里目录是 ~
```

### 1.3. 创建一个 Hexo 的工作目录

这个 Hexo 工作目录其实也是任意的，但是不太建议任意。
我这里 Nginx 的 Web 默认根目录是 `/usr/share/html`，我没有删除原来的目录，我这里把 Hexo 工作目录放在了跟默认的 Web 根目录同级（当然你可以自己换成你想要的位置，注意下后面用到这个目录的时候换成你自己的就好了），即：

```shell
sudo mkdir /usr/share/hexo
```

然后修改目录所有者：

```shell
sudo chown -R hexo /usr/share/hexo
```



### 1.4. 配置 Nginx

我们打开 Nginx 的配置文件：

```shell
sudo vim /etc/nginx/nginx.conf
```



在配置文件的前几行，找到 `user` 字段，将其值改为 hexo，这是为了保持 nginx 运行用户和我们的博客目录一致，避免出现权限问题。



然后找到里面 `http` -> `server` 中的 `root` 字段，将其值改为我们的 Hexo 工作目录（1.3 节创建的那个）。

再把 `http` -> `server` -> `location / ` 里的 `root` 字段值改成 "hexo"。

你记得改成你自己的（如果你的目录和我不一样的话）。

改完以后，我这里的 `server` 字段是这样的：

```
server {
    ...
    root         /usr/share/nginx/hexo;
    ...
    location / {
      root hexo;
      ...
    }
    ...
}
```



保存退出。

然后重启 Nginx：

```
systemctl restart nginx
```

我这里是用 `systemctl` 来操作的，如果你使用的是其他方式，用你自己的来重启 Nginx。



## 2. 本地 Hexo 配置



本地 Hexo 这边，与在 Github Page 上搭建 Hexo 博客是完全一样的。

你只需要安装并在本地测试好 Hexo。

这个过程不是本文重点，略过了。如果你不懂这里，可以查看官方文档或者其他博客。

-

这里只说下需要安装一个插件 `hexo-deployer-git`，这个插件是用来使用 git 来推送更新的。

输入命令：

```shell
npm install hexo-deployer-git --save
```







## 3. 连通本地与服务器



这里假定你的 Hexo 已经安装完成，并且正常工作。



### 3.1. 编辑配置文件



打开本地 Hexo 根目录下的 `_config.yml`，在下面添加/修改以下字段：

{% codeblock "_config.yml" lang:yaml %}
deploy:
  type: git
  repo: hexo@47.93.49.1:~/hexo.git
  branch: master
  message: "Hexo update."
{% endcodeblock %}

* `type`：我们是用 git 更新博客的，所以这里写 "git"；
* `repo`：这里是你的 git 仓库地址，即步骤 1.2 中创建的 hexo.git 的位置；格式为 `[你的用户名]@[你的主机地址]:[你的 git 仓库位置]`。
* `branch`: 更新仓库的分支，我们写默认的主分支 `master`。
* `message`：git 推送时的附加信息。和我们平时用 git 一样，其实这个写什么不重要，你不写这个字段也可以。



### 3.2. 推送一次 Hexo 更新到服务器

我们上面虽然设置了要操作的 git 仓库分支为 `master`，但是我们之前创建的是一个空仓库，是没有任何分支的，所以我们先推送一次 Hexo，这样 git 仓库就会自动创建一个新分支 `master`。

在本地 Hexo 根目录输入命令：

```shell
hexo d -g
```

如过你的主机和你的服务器没有配置 ssh 免密登录的话，应该会让你输入密码。

提示信息最后类似下面这样，这一步就成功了。

```
To 47.93.49.1:~/hexo.git
 * [new branch]      HEAD -> master
Branch 'master' set up to track remote branch 'master' from 'hexo@47.93.49.1:~/hexo.git'.
INFO  Deploy done: git
```

现在服务器上的 git 仓库就有了分支 `master`。



### 3.3. 配置 Git Hook



如果你查看服务器上的 git 仓库的内容的话，你应该会发现，那里并没有我们的博客信息，那里存储的只有关于 git 仓库的一些信息。现在我们要配置一下 git hook，以把我们的博客内容更新到 Nginx 的 Web 目录下。



在服务器上，输入下面的命令，在 git 仓库中的 `hooks` 目录下新建一个 `pre-receive` 文件。

```shell
vim ~/hexo.git/hooks/pre-receive
```

在 `pre-receive` 文件中写入以下内容，然后保存退出。

```
#!/bin/sh
git --work-tree=/usr/share/nginx/hexo --git-dir=/home/hexo/hexo.git checkout -f
```

* `--work-tree` 后面跟的是你 Nginx 的 Web 目录，也就是我们在 1.3 节创建的那个目录；
* `--git-dir` 后面跟的是你的 git 仓库目录，也就是我们在 1.2 节创建的那个目录。



然后输入下面的命令，给 `pre-receive` 加上可执行权限：

```shell
chmod +x ~/hexo.git/hooks/pre-receive
```





### 3.4. 测试结果

先清理下 Hexo 本地的缓存内容（重要）：

```shell
hexo clean
```

然后在本地 Hexo 根目录输入命令：

```shell
hexo d -g
```

如过你的主机和你的服务器没有配置 ssh 免密登录的话，应该会让你输入密码。



最后输出下面这样的信息，就代表这一步成功了。

```
To 47.93.49.1:~/hexo.git
   4add738..3833597  HEAD -> master
Branch 'master' set up to track remote branch 'master' from 'hexo@47.93.49.1:~/hexo.git'.
INFO  Deploy done: git
```

这个时候你可以去看看你服务器中的 Nginx 的 Web 工作目录 `/usr/share/nginx/hexo` 看一下，里面应该已经有你网站的内容了（比如有 index.html）。

现在你可以在浏览器中输入你的网站地址，就可以看到初始的 Hexo 页面了。

-

完工！

另外，你现在每次更新都需要输入密码，是很麻烦的，你可以配置下 SSH 免密登录。  
当然这不是本文重点，就不在这里说啦！