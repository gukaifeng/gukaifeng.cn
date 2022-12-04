---
title: CentOS 下使用 systemd 守护自定义的进程
date: 2021-12-09 17:45:10
updated: 2022-12-05 01:11:10
categories: [技术杂谈]
tags: [Linux,CentOS,systemd]
toc: true
---





我们知道有一些服务型的 Linux 的进程始终在后台运行，并且通常可以做到开机自启，意外退出后自动重启等，始终保持进程常驻不死。  
我们怎样可以让自己编写的程序实现这样的效果呢？这篇文章就来说说这个，主角便是 `systemd`。



本文不打算溯源历史，我们只需要知道 `systemd` 是**当前 Linux 系统的基本构建块套件**。它提供了一个系统和服务管理器，该系统和服务管理器以 `PID 1` 运行并启动系统的其余部分。



 `systemd` 是目前绝大部分发行版的 Linux 系统的 1 号进程：

```shell
$ sudo cat /proc/1/status | grep Name:
Name:	systemd
```

`systemd` 直接与内核交互，性能出色，功能极其强大。我们完全可以将程序交给 `systemd`，让系统统一管理，成为真正意义上的系统服务。



当然了，我们本文要说的进程开机自启、意外退出重启等也只是 `systemd` 所提供功能的冰山一角，不过足以熟悉其基本使用流程，以后深入使用也更容易。



> 官方说法：`systemd` 的拼写是的 "**systemd**"，而不是 "**system D**" 或 "**System D**"，甚至 "**SystemD**"，而且也不是 "**system d**"。因为它是一个系统(system)守护(daemon)程序，在 Unix/Linux 下全部都是小写字母，并且带着一个小写字母 **d** 的后缀代表 daemon。



\-



下面开始，在 CentOS 下使用 `systemd` 守护自定义的进程。



我将步骤整理如下：

1. 准备一个要守护的程序（本文会用一个简单的脚本来演示过程）。
2. 配置 `systemd` 单元文件。
3. 通过 `systemctl` 来控制我们的程序，如启动、重启、停止等。





## 1. 准备一个要守护的程序



我这里写了一个非常简单的 shell 脚本 `/home/gukaifeng/systemd_demo/systemd_demo.sh`：

```shell
#!/bin/bash

demo_log=/home/gukaifeng/systemd_demo/systemd_demo_log.txt

while :
do
    echo "this is the log of systemd demo, current time is" `date` >> $demo_log
    sleep 5s
done
```



这个脚本非常简单，大意就是每隔 5 秒打印一条包含时间信息的日志到 `/home/gukaifeng/systemd_demo/systemd_demo_log.txt` 中，就不演示了。

我们稍后，就将此脚本作为示例程序，配置 `systemd` 使其做到开机自启和意外退出时自动重启。





## 2. 配置 `systemd` 单元文件



### 2.1. 什么是单元(Unit)和单元文件(Unit file)



每一个由 `systemd` 的管理的程序，称为**单元(Unit)**。**单元文件(Unit file)**包含描述单元并定义其行为的配置指令，供 `systemd` 使用。



单元文件的名字格式如下：

```
unit_name.type_extension
```

即 `.` 前面是单元名字，后面是类型扩展，如 `.service` 和 `.socket`。本文只关注 `.service`， 即服务类型的单元。





在 CentOS 中，这些单元文件被统一放在三个目录内（优先级递减，即如在下面的多个目录中出现同名单元文件，`systemd` 只认可更高优先级目录中的）：

* `/etc/systemd/system`：`systemd` 默认读取的单元文件目录。通常由系统管理员或用户配置的单元文件会放在这里。比如阿里云服务器中自家服务的单元文件 `aliyun.service` 就放在此目录。**在系统开机时，`systemd` 只会读取此目录**，所以如果你想要程序开机自启，就需要将其单元文件放在这里。
* `/run/systemd/system`：存放运行时(runtime)单元文件。通常不需要关注此目录。
* `/usr/lib/systemd/system`：已安装包的单元文件放在这里。这里说的 “已安装包” 指的是由 `yum`、`dnf` 等包管理器安装或者 `rpm` 直接安装的包。比如 `ssh` 的单元文件 `sshd.service` 就在这里。



综上，作为系统管理者或普通用户的我们其实只需要关注 `/etc/systemd/system` 这个目录，偶尔关注 `/usr/lib/systemd/system` 目录。





### 2.2. 单元文件的结构



单元文件由三个部分组成，`[Unit]`、`[Unit type]` 和 `[Install]`：

* `[Unit]`：包含不取决于单元类型的通用选项。这些选项提供了单元描述，指定单元的行为，并设置对其他单元的依赖。
* `[Unit type]`：如果单元具有特定于类型的指令，则将这些指令分组为以单元类型命名的部分。具体而言，服务单元文件的此部分应名为 `[Service]` 。
* `[Install]`：包含有关 `systemctl enable` 和 `disable` 命令的单元安装的信息。





### 2.3. `[Unit]` 部分常用选项

| 选项            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `Description`   | 有关单元的有意义的描述。在 `systemctl status` 命令的输出中显示此文本。 |
| `Documentation` | 提供了该单元的 URIs 引用文档列表。                           |
| `After`         | 定义启动此单元的顺序。此单元仅在 `After` 中指定的单元启动后才会启动。与 `Requires` 不同，`After` 不会启动其中指定的单元。`Before` 具有相反的功能。 |
| `Requires`      | 配置对其他单元的依赖性。`Requires` 列出的单元会与此单元一起激活。如果任何 `Requires` 中列出的单元未能启动，则不会激活该单元。 |
| `Wants`         | 配置比 `Requires` 更弱的依赖性。如果列出的单元没有成功启动，则不会对激活此单元产生影响。这是建立自定义单元依赖性的推荐方法。 |
| `Conflicts`     | 配置冲突的依赖关系，这与 `Requires` 相反。                   |

注：在大多数情况下，若单元有依赖的启动顺序，仅在单元文件中设定顺序依赖 `Before` 和 `After` 就足够了。如果你仍有设定 `Wants`（建议）或 `Requires` 的需求，则仍需要设定顺序依赖关系 `Before` 或 `After`。这是因为顺序依赖（`Before` 和 `After`）和要求依赖（`Wants` 和 `Requires`）彼此独立起作用。



若要查看全部 `[Unit]` 部分的选项，见 [systemd.unit(5)](https://man7.org/linux/man-pages/man5/systemd.unit.5.html)。

### 2.4. `[Service]` 部分常用选项

| 选项              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `Type`            | 配置影响 `Execstart` 和相关选项功能的单元进程启动类型。为下列之一：<br />*  `simple` – 默认值。表示由 `ExecStart` 启动的进程是该服务的主进程。<br />* `forking` – 由 `ExecStart` 启动的进程会派生出一个子进程作为该服务的主进程，当子进程完全启动后，父进程便会退出。<br />* `oneshot` – 与 `simple` 类似，但是进程会在启动后续单元之前退出。<br />* `dbus` – 与 `simple` 类似，但是后面的单元仅在主进程获得 D-Bus 名字后才开始。<br />* `notify` – 与 `simple` 类似，但是后面的单元仅在由函数 `sd_notify()` 发出通知信息后才会被启动。<br />* `idle` – 与 `simple` 类似，但是服务二进制程序的实际执行会被推迟，直到所有作业都完成，以避免状态信息的输出和服务的 shell 输出混在一起。 |
| `ExecStart`       | 指定在此单元启动时要执行的命令或脚本。可通过 `ExecStartPre` 和`ExecStartPost` 指定在 `ExecStart` 之前或之后要执行的命令。设定`Type=oneshot` 后，即为指定随后将要按顺序执行的多个自定义命令。 |
| `ExecStop`        | 指定当此单元被停止时要执行的命令或脚本。                     |
| `ExecReload`      | 指定当此单元被重载时要执行的命令或脚本。                     |
| `Restart`         | 启用此选项后，服务在其进程退出后重新启动。搭配 `RestartSec` 可设定重启的间隔时间，单位为秒。重启策略的选择比较多，我在下一个表格中列出。 |
| `RemainAfterExit` | 默认值为 False。如果设为 True，则当该服务所有进程全部退出时，仍会将此服务认定为活跃状态。这个选项在设定了 `Type=oneshot` 时尤其有用。 |



`Restart` 的重启策略：

| 重启策略 / 退出原因    | `no` | `always` | `on-success` | `on-failure` | `on-abnormal` | `on-abort` | `on-watchdog` |
| ---------------------- | ---- | -------- | ------------ | ------------ | ------------- | ---------- | ------------- |
| 干净的退出状态码或信号 |      | √        | √            |              |               |            |               |
| 不干净的退出码         |      | √        |              | √            |               |            |               |
| 不干净的信号           |      | √        |              | √            | √             | √          |               |
| 超时                   |      | √        |              | √            | √             |            |               |
| 看门狗(Watchdog)       |      | √        |              | √            | √             |            | √             |





若要查看全部 `[Service]` 部分的选项，见 [systemd.service(5)](https://man7.org/linux/man-pages/man5/systemd.service.5.html)。

### 2.5. `[Install]` 部分常用选项

| 选项              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `Alias`           | 设定此单元的别名，多个别名由空格隔开。除了 `systemctl enable` 以外的大部分 `systemctl` 命令都可以通过别名来操作这个单元。 |
| `RequiredBy`      | 依赖此单元的单元列表。当此单元启动后，`RequiredBy` 列表中的单元若有在 `Require ` 中设定此单元，那么将会满足。 |
| `WantedBy`        | 弱依赖此单元的单元列表。当此单元启动后，`WantedBy` 列表中的单元若有在 `Wants ` 中设定此单元，那么将会满足。 |
| `Also`            | 指定与此单元一起安装或卸载的单元列表。                       |
| `DefaultInstance` | 仅限于实例化单元，此选项指定启用该单元的默认实例。详见 [Working with instantiated units](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/assembly_working-with-systemd-unit-files_configuring-basic-system-settings#con_working-with-instantiated-units_assembly_working-with-systemd-unit-files)。 |





若要查看全部 `[Install]` 部分的选项，见 [systemd.unit(5)](https://man7.org/linux/man-pages/man5/systemd.unit.5.html)。



### 2.6. 本例中的单元文件示例

前文说过，对于我们自己实现的单元，关于单元文件目录我们只需要关注 `/etc/systemd/system`。

我们需要给我们的服务起一个名字，这里暂定为 `systemd_demo`，我们的单元类型应为 `service`，故单元文件名应为 `systemd_demo.service`。



我们编辑文件：

```shell
sudo vim /etc/systemd/system/systemd_demo.service
```

写入以下内容，并保存退出：

```shell
[Unit]
Description=A systemd demo

[Service]
ExecStart=bash /home/gukaifeng/systemd_demo/systemd_demo.sh

Restart=on-failure
RestartSec=5
```

这里的 `Type` 我们没有写，则为默认的 `simple`。我们设定了描述，单元启动时执行我们的 `systemd_demo.sh` 脚本，并设定了重启策略和重启间隔时间 5 秒。



这就是一个非常简单的单元文件配置了。



## 3. 启动并验证单元

前文说过，**系统启动时**，`/etc/systemd/system/` 是 `systemd` 唯一检索的单元文件目录，**在此目录内的单元都会开机自启**。



但单元文件配置好后，如果不重启系统，是不会立即启动的，所以我们这里先手动启动：



```shell
systemctl start systemd_demo

or

systemctl start systemd_demo.service
```

这里 `systemctl start` 后面接单元名字 `systemd_demo`，或者单元文件名字 `systemd_demo.service` 均可，会自动识别。



启动后，我们可以通过 `systemctl status` 命令查看状态：

```shell
$ systemctl status systemd_demo
● systemd_demo.service - A systemd demo
   Loaded: loaded (/etc/systemd/system/systemd_demo.service; static; vendor preset: disabled)
   Active: active (running) since Mon 2022-12-05 00:50:19 CST; 8s ago
 Main PID: 2053314 (bash)
    Tasks: 2 (limit: 49489)
   Memory: 624.0K
   CGroup: /system.slice/systemd_demo.service
           ├─2053314 /usr/bin/bash /home/gukaifeng/systemd_demo/systemd_demo.sh
           └─2053318 sleep 5s
```

可以看到显示我们的单元是活跃的 `active (running)`，也能看到我们给此单元写的描述 "A systemd demo"。



记得我们此脚本会每隔 5 秒打印一条日志到文件中，我们可以查看：

```shell
$ tail -f /home/gukaifeng/systemd_demo/systemd_demo_log.txt 
this is the log of systemd demo, current time is Mon Dec 5 00:50:49 CST 2022
this is the log of systemd demo, current time is Mon Dec 5 00:50:54 CST 2022
this is the log of systemd demo, current time is Mon Dec 5 00:50:59 CST 2022
this is the log of systemd demo, current time is Mon Dec 5 00:51:04 CST 2022
this is the log of systemd demo, current time is Mon Dec 5 00:51:09 CST 2022
...
```

可以看到我们的程序确实如期运行了！



我们可以尝试 `kill` 杀死此进程，此进程不会重启，因为我们设定的重启策略是 `on-failure`。但如果使用 `kill -9` 来杀死此进程，那么进程将在我们设定的 5 秒后重启。这里就不演示了。





## 4. 常用的 `systemctl` 子命令



我们前文已经演示过了 `systemctl start` 和 `systemctl status`。

这一小节给出几个我认为十分常用的 `systemctl` 子命令。



| `systemctl` 子命令 | 作用                                 |
| ------------------ | ------------------------------------ |
| `start`            | 启动指定单元。                       |
| `restart`          | 重启指定单元。                       |
| `stop`             | 停止指定单元。                       |
| `status`           | 查看指定单元状态信息。               |
| `enable`           | * 设定指定单元为开机自启。           |
| `daemon-reload`    | 重载指定单元的单元文件使其立即生效。 |

\* `systemctl enable` 命令会设定指定单元为开机自启。但前文我们说过，单元文件放在目录 `/etc/systemd/system/` 下的单元才会开机自启，也就是说，如果我们的单元文件已经放置在该目录中，那么我们无需 `enable`，其便会开机自启。此命令主要用于单元文件在目录 `/usr/lib/systemd/system` 中的单元，这些单元默认不会开机自启，`enable` 会在 `/etc/systemd/system/` 目录中创建一个指向目录 `/usr/lib/systemd/system` 中指定单元文件的符号连接，以此实现对应单元的开机自启。`enable` 后，相应的单元文件也会重载（等价 `daemon-reload`）。`disable ` 是与之相反的命令，并且会删除即便不是由 `enable` 创建的符号连接。





