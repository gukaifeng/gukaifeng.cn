这里主要指的是忘记了现在的密码，需要强制修改。

首先登录 Mon 机器。

在一个文件内写入新密码，例如：

```shell
vim passwd.txt
```

在里面写上密码，注意就直接密码，不需要任何格式。文件名也没有任何要求，随便起。

然后执行：

```shell
ceph dashboard ac-user-set-password <username> -i <file> [--force-password] 
```

* `username`：用户名，比如管理员默认的 `admin`。
* `file`：刚刚创建的包含了新密码的文件。
* `--force-password`：强制修改，不加的话弱密码会被拒绝。

