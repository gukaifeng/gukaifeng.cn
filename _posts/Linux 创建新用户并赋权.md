


本文所述内容需要在 root 用户或有 root 权限的用户下操作。



这里假定你已经以 root 用户登录到了 Linux。

## 1. 创建新用户





新建一个用户：

```shell
adduser <new_username>  
```

给新用户设置密码：

```shell
passwd <new_username>  
```





\-



例如我这里创建一个名为 gukaifeng 的用户，那么命令如下：

```shell
adduser gukaifeng  # 新建一个名为 gukaifeng 的用户
```

```shell
passwd gukaifeng  # 为 gukaifeng 用户设置密码
```

\-







上面的命令已经自动创建好用户目录了，例如我这里的 `/home/gukaifeng`。



这一步完成后，我们的新用户就可以正常登录使用了。



\-



**为了说起来方便，下面的步骤我都以新用户名为 "gukaifeng" 为例。**

## 2. 给新用户添加 sudo 权限



这里给出两个方法可以做到这一点，我建议使用方法二。

### 2.1. （不推荐）直接修改 `/etc/sudoers`



权限配置文件为 `/etc/sudoers`，这个文件默认是没有写权限的，所以我们需要先给其添加写权限：



```shell
chmod +w /etc/sudoers
```

然后打开此配置文件：

```shell
vim /etc/sudoers
```



找到下面这段配置：

```
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
```



仿照这个格式，为我们的新用户添加一行，我这里如下：

```
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
gukaifeng    ALL=(ALL)       ALL
```



保存退出后，执记得移除权限配置文件的写权限：

```
chmod -w /etc/sudoers
```



\-



权限配置文件为 `/etc/sudoers` 非常敏感，这一点从其默认没有写权限也可以看得出来，所以我这里建议使用下面的方法。



### 2.2. （推荐）修改 `/etc/sudoers.d/`





实际应用中，我们倾向修改 `/etc/sudoer.d/`，这是一个目录，目录内可以有很多个配置文件，每个配置文件的书写格式与 `/etc/sudoers` 一致。



显然 `/etc/sudoer.d/` 下的权限配置文件更易于维护。



我们创建并编辑我们自己的配置文件：



```shell
vim /etc/sudoers.d/gukaifeng
```



配置文件的名字是任意的，我这里起名与用户名一样，仅仅是为了易于理解。



然后在其中写入一行：

```shell
gukaifeng    ALL=(ALL)       ALL
```



保存并退出以后，我们的新用户就可以使用 sudo 权限了。



## 3. （可选）使 sudo 操作免密



配置完前面的内容以后，新用户虽然可以使用 sudo，但是需要输入用户密码，时间久了挺烦的。或者有些脚本里需要使用 sudo 权限，去改脚本自动填充密码什么的，也很麻烦。我们可以为用户配置，使用 sudo 的时候不再需要输入用户密码，这样就得劲多了。



这里以 2.2 中的操作为例，我们还是编辑自己的权限配置文件：



```shell
vim /etc/sudoers.d/gukaifeng
```



将我们之前写的一行：



```shell
gukaifeng    ALL=(ALL)       ALL
```



修改为：



```shell
gukaifeng    ALL=(ALL)       NOPASSWD:ALL
```



这样以后新用户使用 sudo 命令，就不再需要输入用户密码了。