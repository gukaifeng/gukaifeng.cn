---
title: 我的 Git 常用命令笔记
date: 2022-11-10 18:46:20
updated: 2023-05-22 16:33:20
categories: [技术杂谈]
tags: [Git]
---





## 1. 基础命令



这篇文章并不是一次性写完的，而是我想到什么就加进来什么的，所以下面的顺序并不重要。



### 1.1. 修改本地分支名字



```shell
git branch -m <old_branch_name> <new_branch_name>
```

修改后会与关联的远程分支失去联系，首次 push 需要指定远程分支，一般在你首次 push 的时候会给你个提示，告诉你怎样做，好解决。

比如我将本地的 master 分支改名为 main，然后首次 push 应当这样：

```shell
git push origin HEAD:master
```

这样会将本地的 main 分支与远程的 master 分支关联，就可以正常使用了。



修改本地分支关联的远程分支：

```shell
git branch --set-upstream-to=<remote_rep_name>/<remote_branch_name> <local_branch_name>
```

例如：

```shell
git branch --set-upstream-to=origin/main main
```

将本地分支 main 的与远程仓库 origin 的 main 分支关联。



### 1.2. 删除本地分支

```shell
git branch -d <local_branch_name>
```

`-d` 与 `--delete` 等价

例如：

```shell
git branch -d dev
```

```shell
git branch --delete dev
```



### 1.3. 删除远程仓库分支

```shell
git push <remote_rep_name> -d <remote_branch_name>
```

`-d` 与 `--delete` 等价

例如：

```shell
git push origin -d dev
```

```shell
git push origin --delete dev
```

### 1.4. 同时删除本地与远程分支

没有 git 命令来同时删除本地和远程分支，毕竟本地分支和远程分支是多对多的关系，而非一对一。

所以要想同时删除，就需要分别执行本地分支的删除和远程分支的删除，即上面两小节。

\-

如果不知道本地分支与远程分支的对应关系（因为分支名可能不同），可以通过下一小节查看。

### 1.5. 查看本地分支和远程分支的对应关系



```shell
git branch -vv
```

这个命令会显示 **所有** 本地分支和它们对应的远程分支。

上面的输出中，可以看到本地分支 `main` 对应的远程分支是 `origin/main`，本地分支 `feature` 对应的远程分支是 `origin/feature`，本地分支 `fix` 对应的远程分支是 `origin/fix`。

`origin/fix` 后面的 `ahead 1` 表示本地的 `fix` 分支比远程的 `fix` 分支多了一个提交。

例如：

```shell
$ git branch -vv
  main     12345ef [origin/main] A main branch commit
* feature  67890ab [origin/feature] A feature branch commit
  fix      23456cd [origin/fix: ahead 1] A fix branch commit
```







## 2. 场景



### 2.1. 关于贮藏

下面的所有列出场景都是我自己实际用过的场景，但 git stash 的功能远不止这些，如果你没有在下面找到你适用的场景，可以参考 https://man7.org/linux/man-pages/man1/git-stash.1.html 或 https://git-scm.com/docs/git-stash 查看更多内容。

#### 2.1.1. 贮藏代码



> 我们从分支 A 分出了分支 B，并在 B 分支上开发。  
> 在 B 分支开发结束前，我们接到了新的任务且优先级更高，需要暂时中断分支 B 的任务，并从分支 A 再分出一个新的分支 C，在分支 C 上进行新的开发。



此场景需要以下几步：

1. 暂存分支 B 中的修改；
2. 回到分支 A（并更新，如果有需要）；
3. 在分支 A 上新建分支 C，并切换到分支 C 开始新的开发；
4. 拓展：C 分支任务完成后，继续 B 分支的开发（或交替开发）。

\-

**1. 暂存分支 B 中的修改**

```shell
git stash push [-u|--include-untracked] [-m|--message <message>]
```

这会将你的修改暂存，索引工作树的代码都会恢复到 HEAD。

我这里给出了两个常用的**可选**参数：

* `-m`：给这个暂存条目写个描述，用法和 `git commit -m` 一样。不写也可以，但如果暂存的条目比较多的话容易混淆，建议写上。

* `-u`：表示包含未跟踪的文件。如果你有新添加的文件要贮藏，就需要加上这个。

> 还有更多参数见 https://git-scm.com/docs/git-stash。



然后可以通过 `git stash list` 命令看到你的 stash 队列。



**2. 回到分支 A（并更新，如果有需要）**

回到分支 A：

```shell
git switch A
```

更新（如果需要）：

```shell
git pull
```

这一步比较自由，可以看自己的需求，比如你也可以从 B 直接 pull 新的，但是从 B 创建 C。



**3. 在分支 A 上新建分支 C，并切换到分支 C 开始新的开发**

创建分支 C：

```shell
git branch C
```

切换到分支 C：

```shell
git switch C
```

现在可以开始新的开发了。



#### 2.1.2 恢复贮藏的代码



> 续 2.1.1：C 分支任务完成后，继续 B 分支的开发（或交替开发）
>
> 假设我们现在 C 分支已经开发完成，需要回到 B 分支，恢复我们暂存的内容，然后继续开发。



若 C 分支是刚刚开发完成的（已经提交完所有的修改），那么工作区应当是干净的，所以可以直接切回 B 分支：

```shell
git switch B
```

这一步如果分支 C 还没有开发完成，想要交替开发分支 B 的话，则在分支 C 上执行上面 2.1.1 中的 1、2 步即可。

我们可以先看看分支 B 中有哪些暂存条目：

```shell
$ git stash list
stash@{0}: On example_B: this is a description message of stash entry
```

可以看到我这里 B 分支的名字叫做 `example_B`，只有一个 stash 条目 `stash@{0}`，然后有关于这个 stash 条目的描述 "this is a description message of stash entry"。

我们恢复 `stash@{0}` 中的代码到工作区：

```shell
git stash pop

// or
git stash apply
```

现在我们的分支 B 就回到了一开始状态，可以继续开发了！

上面的 `git stash pop` 和  `git stash apply` 类似，二者的区别是 `pop` 会弹出 stash 栈中的条目，你再 `git stash list` 时就没有这条目了，而 `apply` 只是应用条目中的内容，但不在栈中将此条目弹出。如果你以后不再需要这个 stash 条目，就用 `pop`，反之用 `apply`。



#### 2.1.3. 在多个 stash 条目中选择一个恢复

> 假设当前分支有多个 stash 条目，我们想恢复其中一个（不是栈顶的那一个）。

我们知道 stash 条目是存在栈中的，2.1.2 中提到的 `git stash pop` 和 `git stash apply` 默认是恢复栈顶的那个条目，也就是最后 push 的条目。

如果我们想要恢复栈中间的也很简单，每个 stash 条目都有一个编号，例如上面的 `stash@{0}` 中的 `0` 就是该条目的编号，我们只需要在 `git stash pop` 和 `git stash apply`  命令后面加上编号即可，例如：

```shell
git stash pop 2  // 应用并删除条目 stash@{2}
git stash apply 3  // 应用但不删除条目 stash@{3}
```





### 2.2. 拉取只存在远程仓库的分支到本地

> 假设我们克隆了**曾经最新**的仓库到本地，其中有分支名为版本的 `v6.25.3` 的分支，然后我们在此分支基础上创建了本地分支 `v6.25.3-dev` 开发某些东西。后来，在我们的分支 `v6.25.3-dev` 开发完成前，远程仓库添加了新版本的分支 `v6.26.0`，而我们的开发内容最终要 push 到这个新分支 `v6.26.0` 上了，而不是旧的 `v6.25.3`。

此场景需要以下几步：

1. 贮藏本地开发中分支（本例为 `v6.25.3-dev` ）上的修改；
2. 拉取远程最新的分支（本例为 `v6.26.0`）到本地，并切换到该分支；
3. 在新分支上创建新的开发分支（本例为 `v6.26.0-dev`），并切换到该分支。
4. 在新的开发分支上，恢复我们之前贮藏的修改，并继续开发。



贮藏比较简单，我们只需要执行下面的命令：

```shell
git stash push [-u|--include-untracked] [-m|--message <message>]
```

其中 `-u` 表示也贮藏未跟踪的文件，`-m` 就类似备注。

然后我们要拉取远程的新分支，因为直接 `git pull` 或者 `git fetch` 是没用的，所以可能很多人会卡在这一步。

这个最麻烦的是，我们在本地是看不到远程的新分支的（这个分支是在我们 `git clone` 之后创建的）。

使用下面的命令：

```shell
git fetch <远程仓库别名> <远程新分支名>:<本地新分支名>
```

* **远程仓库别名**：就是远程仓库在本地的标识。可以用 `git remote` 看到，也可以再加个如 `-v` 参数看的更详细一点。默认情况可能是 `origin`。
* **远程新分支名**：我们要拉取的远程分支。在这个场景里就是远程仓库新建的那个分支。
* **本地新分支名**：新拉取的远程分支对应的本地分支名字。通常建议与**远程新分支名**和一致，当然你也可以起别的名字，只要你自己知道就好。

此例我这里的命令为，注意远程分支名和本地新分支名中间是冒号 `:` 隔开的：

```shell
git fetch origin v6.26.0:v6.26.0
```

该命令执行完成后，可以用 `git branch` 看到本地已有新分支了 `v6.26.0`。

后面的操作就比较简单自由了，我这里完成上述场景。

由于我们前面已经贮藏过旧分支的改动，所以可以直接切换到新分支：

```shell
git switch v6.26.0
```

创建并切换到新的开发分支：

```shell
git branch v6.26.0-dev
git switch v6.26.0-dev
```

恢复我们旧开发分支中贮藏的内容（可能需要合并冲突）：

```shell
git stash pop
```

到这就搞定了，然后后面就接着开发就 ok 了！



### 2.3. 追新 fork 的源仓库



> 你 fork 了一个仓库，并在自己 fork 出的仓库里操作某些东西。后来你的 fork 的源仓库更新了，你想同步源仓库。



我这里以 facebook 的 rocksdb 仓库为例，因为写这里的时候我刚好在操作这个仓库。

首先我们可以看一下当前的远程仓库信息：

```shell
$ git remote -v
origin	git@github.com:gukaifeng/rocksdb.git (fetch)
origin	git@github.com:gukaifeng/rocksdb.git (push)
```

可以看到远程只有我自己 fork 后的仓库，所以我们得把源仓库加上才行。

```shell
$ git remote add fb_origin git@github.com:facebook/rocksdb.git
```

其中 `fb_origin` 是给新远程仓库起的别名，比较随意，随便起，自己知道咋回事就行，然后后面跟的是源仓库的地址。



现在再看一下远程仓库信息：

```shell
$ git remote -v
fb_origin	git@github.com:facebook/rocksdb.git (fetch)
fb_origin	git@github.com:facebook/rocksdb.git (push)
origin	git@github.com:gukaifeng/rocksdb.git (fetch)
origin	git@github.com:gukaifeng/rocksdb.git (push)
```

可以看到已经有了源仓库了，然后就可以拉取源仓库追新了，例如：

```shell
$ git pull fb_origin main
```

`fb_origin` 为远程仓库别名，`main` 为要拉取的分支，这样就可以追更源仓库了（如果你自己也有修改的话，可能需要处理冲突）！







### 2.3. 将本地项目文件夹上传到 GitHub 仓库



> 本地有一个项目文件夹，其中没有使用过 git，我们现在想把这个文件夹放到远程 git 仓库去（比如 GitHub）。



这里以 GitHub 为例，先在网页上创建一个新的仓库，主要什么都不要创建（比如取消勾选创建 `README.md` 等），保证新仓库是空的。  
然后我们会拿到一个新仓库的 git 链接，这里假定是 `git@github.com:gukaifeng/xxx.git`，  
另外 GitHub 目前的默认分支是 main，这些信息后面会用到。



首先进入要上传的本地项目根目录，进行初始化操作：

```shell
git init
```

然后提交仓库内的全部内容（如果有需要排除的，建议编辑 `.gitignore` 或在 `add` 命令中排除，这些不进一步解释了，不是重点）：

```shell
git add .
git commit -m "first commit"
```

创建本地分支 `main`：

```shell
git branch -M main
```

添加远程仓库：

```shell
git remote add origin git@github.com:gukaifeng/xxx.git
```

其中 `origin` 是远程仓库别名，只在本地有效，随便起。后面的 git 链接就是我们之前在 GitHub 上创建的新仓库链接。



最后 push：

```shell
git push -u origin main
```

这里简单解释一下，我们在前面创建了本地分支 `main`，因为当前本地就这一个分支，所以我们直接就是在本地 `main` 分支下继续操作的。  
这里的 `-u` 参数全称是 `--set-upstream`，即设置上游的意思，后面跟着的 `origin` 和 `main` 表示，我们要将当前本地分支，与上游仓库 `origin` 的分支 `main` 关联。最后就是 `git push` 本身的操作，将我们本地的内容推到远程仓库库去。

到这里就完成了，后面的使用就都和普通仓库一样了。
