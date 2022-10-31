---
title: 解决 CentOS 中 LD_LIBRARY_PATH 配置正确却找不到动态链接库的问题
date: 2022-10-30 16:46:49
updated: 2022-10-30 16:46:49
categories: [技术杂谈]
tags: [CentOS,Linux]
---



今天遇到一个很离谱的问题，场景如下：

运行某可执行程序时，提示某动态库找不到，错误如下：

```
libxxx.so: cannot open shared object file: No such file or directory
```

通过 `ldd`  命令查看可执行程序的链接库，可以得到其确实没有找到此库：

```shell
...
libxxx.so => not found
...
```



**但在这里，我可以 100% 确定，我的 `LD_LIBRARY_PATH` 配置没有问题，此动态库也确实存在！**

最终找出问题所在，但应当是 bug。

我们知道，动态库往往都是，有一个名字上带有详细版本号的真正的动态库文件，如 `libxxx.so.7.6.0`，有多个精简了版本的软连接指向前面那个真正的动态库，如 `libxxx.so`，`libxxx.so.7`，`libxxx.so.7.6` 等。就像下面这样：

```
libxxx.so -> libxxx.so.7.6.0
libxxx.so.7 -> libxxx.so.7.6.0
libxxx.so.7.6 -> libxxx.so.7.6.0
libxxx.so.7.6.0
```

所以我们前面没有找到的动态库 `libxxx.so`，是一个软连接，其指向真正的动态库文件。

我们创建此动态库的时候（不论是手动创建的，还是安装程序或脚本自动创建的），**很有可能使用的是相对路径！**

问题就出在这里，虽然相对路径看起来没有什么问题，比如 `ll` 命令查看时可以看到是正确的，但并不能被程序一定正确识别。

**所以解决方案是，使用绝对路径，重新建立上述软连接。**
