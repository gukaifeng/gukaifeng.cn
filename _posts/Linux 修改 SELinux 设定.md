




## 1. 为什么要修改防火墙设定



我们有些时候做一些操作会遇到一些莫名其妙的问题 ~



比如使用 `systemctl` 启动某些服务时被拒绝（root 用户也不行）：

```
Failed to enable unit: Access denied
```

或者有些时候直接提示找不到 unit：

```
Failed to start xxx.service: Unit not found
```

但是查看 `/etc/systemd/system/` 等 unit 存放目录，我们的 `xxx.service` 就安静的躺在那里。。。







\-



这些问题可能是由于 Linux 防火墙 SELinux 的设定引起的。







## 2. SELinux 的工作模式



SELinux 有三种工作模式，分别为：



* `enforcing`：强制模式，一切违背 SELinux 政策的行为都将被禁止。
* `permissive`：宽容模式，违背 SELinux 政策的行为可以进行，但会被记录到日志中。
* `disbaled`：完全禁用 SELinux。







## 3. 修改 SELinux 的设定



### 3.1. 临时修改



临时修改可使用 `setenforce ` 命令：



```shell
setenforce [ Enforcing | Permissive | 1 | 0 ]
```



下面两个命令等价，将 SELinux 的工作模式临时修改为强制模式：

```shell
setenforce Enforcing
```

```shell
setenforce 1
```



下面两个命令等价，将 SELinux 的工作模式临时修改为宽容模式：

```shell
setenforce Permissive
```

```shell
setenforce 0
```



\-



`setenforce` 命令不能将 SELinux 的工作模式由 enforcing 或 permissive 修改为 disabled，或将 disabled 修改为 enforcing 或 permissive。



临时修改只对当前终端有效，退出或重新登录后就会恢复。



\-



与这个命令对应，我们可以使用命令 `getenforce` 查看当前的 SELinux 工作模式，如：



```shell
$ getenforce
Permissive
```







### 3.2. 永久修改



SELinux 的配置文件是 `/etc/selinux/config`，我们打开：

```shell
vim /etc/selinux/config
```

可以看到配置文件中默认有以下内容：

```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

我这里将 SELinux 的工作模式修改为宽容模式：

```shell
SELINUX=permissive
```

这样我们之前的操作就可以顺利进行了。



\-



永久修改需要重启系统才会生效。



\-

注意配置文件中的 `SELINUXTYPE` 字段轻易不要动！这个有点危险，搞不好都开不开机的。

除非你明确知道你的改动代表什么，否则不要修改这个字段！





## 4. 扩展：查看 SELinux 日志



SELinux 的日志文件为 `/var/log/audit/audit.log`。



但是我们通常不会直接查看这个日志文件，因为这个记录的东西太多了，我们可以使用 `audit2why` 帮助我们只查看错误信息：



```shell
audit2why < /var/log/audit/audit.log
```

如果有错误就会输出，没有输出的话就是没有错误。



注意 `/var/log/audit/audit.log` 的访问需要 root 权限。



另外这个命令需要在 enforcing 或 permissive 模式下使用，否则会提示找不到防火墙的政策文件。