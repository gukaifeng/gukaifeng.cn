---
title: "Linux 使用 crontab 设定定时任务"
date: 2023-04-30 02:09:00
updated: 2023-04-30 02:09:00
categories: [技术杂谈]
tags: [Linux]
---



## 1. `crontab` 命令的作用



Linux 中可以使用 `crontab` 命令来设定定时任务，例如每小时执行一次某个脚本。



具体来说，Linux 中有一个程序 `crond`（几乎所有发行版都自带），用于执行计划命令的守护程序。`crontab` 命令帮助我们快速设定这些任务，实际是修改 crontab 文件，`crond` 程序会定期检查这些 crontab 文件，执行其中需要执行的命令。



crontab 文件主要存储在三个地方：

* `/etc/crontab`: 是一个文件，存储主要的系统 crontab 文件。
* `/etc/cron.d/`: 是一个目录，也是存储了一些系统 crontab 文件。
* `/var/spool/cron/`: 是一个目录，里面存着各个用户定义的 crontab 文件，这个目录是我们自定义定时任务主要需要关注的。





需要注意的是，`crond` 的检查间隔是 1 分钟，也就是我们定时任务的执行最小间隔是 1 分钟。  
如果我们有更小间隔的定时任务需要执行，那么 `crontab` 命令是无法满足的，可能需要自己写一个专门的守护程序。



## 2. 如何设定 crontab 定时任务



前面说过，我们定义自己的定时任务，主要关注的是 `/var/spool/cron/` 目录下的 crontab 文件。



如果你的机器上没有任何用户使用 crontab 设定过定时任务，那么 `/var/spool/cron/` 目录应当是空的。如果有用户使用 crontab 定义过定时任务，那么这个目录下会有个与用户名相同的文件，即属于此用户的 crontab 文件。例如，我当前的用户名为 "gukaifeng"，那么当我设定过定时任务以后，这个目录下将会有一个 名为 "gukaifeng" 的 crontab 文件，即 `/var/spool/cron/gukaifeng` 。





编写 crontab 文件有几种方式：



1. （不推荐）直接使用文本编辑器编辑 crontab 文件，例如：

   ```shell
   vim /var/spool/cron/gukaifeng
   ```

   直接编辑 crontab 文件的话，crontab 程序不会对此文件进行检查，写错了也不容易被发现，导致定时任务按期望运行，所以不推荐此方法。

2. （不推荐）指定一个现有的 crontab 文件：

   ```shell
   crontab /path/to/your_crontab_file
   ```

   这个命令会直接拿你指定的 crontab 文件，替换掉 `/var/spool/cron/` 目录下属于你用户的那一个，同样不会进行正确性检查，而且原来的文件还直接丢了，风险很大，不推荐。

3. **（推荐）使用 `crontab` 命令设定：**

   ```shell
   crontab -e
   ```

   这个命令最安全的，其首先会将属于你用户的那一个 crontab 文件（如我这里的 `/var/spool/cron/gukaifeng`）的内容拷贝到临时文件中（如 `/tmp/crontab.JI8lbA`，如果用户没有 crontab 文件的话，就会创建一个空的此临时文件），并用文本编辑器打卡（默认是 `vi`）。我们编辑这个临时文件，保存并退出以后，`crontab` 程序会检查这个临时文件是否有错误，如果有错误会强制让你修改，如果没问题，才会将其中的内容应用到目录 `/var/spool/cron/` 下的 crontab 文件中。





下面我们说一下 crontab 文件具体的编写规范。





## 3. 一个简单 crontab 文件



我们首先使用命令编辑用户的 crontab 文件：

```shell
crontab -e
```

注意前文提过这里编辑的是一个临时文件，`crontab` 程序检查正确以后才会应用到真正的用户 crontab 文件。





这个命令会使得我们进入此临时文件 `vi` 的编辑页面，我们按规范编写后，退出保存即可，crontab 程序检查无误后就会生效。



这里通过一个简单的例子说明 crontab 文件的编写方法：



```shell
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

0 * * * * cd / && ls >> /home/gukaifeng/crontab_example
```

* `SHELL`：设定执行设定任务的 shell 程序。
* `PATH`：与系统的 PATH 含义一样，设定环境变量。如果不设置的话，建议在使用命令的时候用绝对路径，不然可能出现找不到命令的情况。
* `MAILTO`：邮件的接收用户。`crontab` 并不会将定时任务的输出直接打印出来，而是将其输出以邮件的形式发送到指定用户。这里更建议将需要查看输出或错误的定时任务的输出重定向到一个指定的位置，这个邮件系统并不好用。





下面看最关键的部分，我这里的例子为：

```shell
0 * * * * cd / && ls >> /home/gukaifeng/crontab_example
```

这行的意思为：

* 该定时任务每小时执行一次（`0 * * * *` 代表每小时执行一次，后面会详细说）；

* 定时任务执行的命令为 `cd / && ls >> /home/gukaifeng/crontab_example`。

  这里的命令和我们直接在命令行执行是一样的，即先进入 `/` 目录，执行 `ls` 命令并将输出追加到文件 `/home/gukaifeng/crontab_example` 中。



这里我加上了 `cd` 操作来进入指定目录，是因为 crontab 中的命令默认是在 `~` 目录运行的，如果我们有在指定目录执行定时任务的需求，就有必要使用 cd 命令先进入到指定目录。







## 4. 命令执行周期的设定规则





不难看出，crontab 中设定定时任务的语法很简单，这里给出具体语法如下：

```shell
*  *  *  *  * [user-name] <command to be executed>
```

* `*  *  *  *  *`：设定命令的执行周期。
* `user-name`：指定执行设定命令的用户。这里需要你的用户有指定用户的权限，例如 root 用户可以指定普通用户。这个并不常用，上面的例子里我也没写。默认情况下，是哪个用户的 crontab 文件中的定时任务，就以哪个用户的身份执行。
* `command to be executed`：要执行的命令。





\-



显然，这里最复杂的就是命令的执行周期设定了，这里给出格式如下：

```
.---------------- minute (0 - 59)
|  .------------- hour (0 - 23)
|  |  .---------- day of month (1 - 31)
|  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
|  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
|  |  |  |  |
*  *  *  *  * [user-name]  <command to be executed>
```

* 第 1 个 `*`：分钟，取值为 0 - 59。
* 第 2 个 `*`：小时，取值为 0 - 23。
* 第 3 个 `*`：日，取值为 1 - 31。
* 第 4 个 `*`：月，取值为 1 - 12，也可以是英文简写，如 `jan`、`feb`、`mar`、`apr` 等等
* 第 5 个 `*`：星期几，取值为 0 - 7（1 - 6 分别为周一到周六，0 和 7 都表示周日），也可以是英文简写，周一到周日分别为 `mon`、`tue`、`wed`、`thu`、`fri`、`sat`、`sun`。



\-





如果不设定某个项（即保持为 ``*`）则表示取全部值，如：

* `* * * * *`：表示每分钟执行一次。
* `0 * * * *`：表示每小时整点执行一次。即我们没有限制小时、日、月和星期几，仅限制了在每个小时的第 0 分钟执行（如 08:00 会执行）。
* `3 * * * *`：与上一个一样，表示每小时的 03 分执行，如 09:03，12:03 等。
* `5 8 3 * *`：在每个月的 3 号 08:05 执行一次。





\-



这个周期设定也支持一些高级语法：

* `-`：表示一个区间。

* `,`：分隔多个值。
* `/`：表示每隔多久。



我们通过例子来理解比较简单：



* `0 3 * * 1-5`：每周一到周五的 03:00 执行一次。
* `0 0 1,15 * *`：每月 1 号和 15 号执行一次。
* `0 */2 * * *`：每隔 2 小时的 0 分时执行一次，具体来说，是每天的 00:00、02:00、04:00 ... 22:00 执行一次。
* `15 5-10/2 * * *`：在每天的 5 - 10 点钟，每隔 2 小时的 15 分时执行一次，具体来说，是每天的 05:15、07:15、09:15 等执行一次。





理解以后其实很简单，纸老虎~







## 5. 查看 crontab 的日志



我们可以通过查看 cron 日志来了解定时任务的执行情况，该日志存在 `/var/log/` 目录下，有多个：

```shell
$ ll /var/log/cron*
-rw------- 1 root root  2793 Apr 30 10:01 /var/log/cron
-rw------- 1 root root 57714 Apr  9 03:01 /var/log/cron-20230409
-rw------- 1 root root 57918 Apr 16 03:01 /var/log/cron-20230416
-rw------- 1 root root 58189 Apr 23 03:01 /var/log/cron-20230423
-rw------- 1 root root 73936 Apr 30 03:01 /var/log/cron-20230430
```

从名字就看的出来，后缀带日期的是旧的日志，是那个日期的 cron 日志。

`/var/log/cron` 是最新的，我们大部分情况下查看这个就可以了：

```shell
$ sudo cat /var/log/cron
Apr 30 03:39:01 iZ8vbf7xcuoq7ug1e7hjk5Z anacron[2612749]: Job `cron.daily' started
Apr 30 03:39:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2613043]: (/etc/cron.daily) starting logrotate
Apr 30 03:39:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2613043]: (/etc/cron.daily) finished logrotate
Apr 30 03:39:01 iZ8vbf7xcuoq7ug1e7hjk5Z anacron[2612749]: Job `cron.daily' terminated
Apr 30 03:39:01 iZ8vbf7xcuoq7ug1e7hjk5Z anacron[2612749]: Normal exit (1 job run)
Apr 30 04:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2613228]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 04:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2613228]: (/etc/cron.hourly) starting 0anacron
Apr 30 04:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2613228]: (/etc/cron.hourly) finished 0anacron
Apr 30 05:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2614312]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 05:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2614312]: (/etc/cron.hourly) starting 0anacron
Apr 30 05:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2614312]: (/etc/cron.hourly) finished 0anacron
Apr 30 05:15:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2614430]: (gukaifeng) CMD (date >> /home/gukaifeng/test123)
Apr 30 06:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2615106]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 06:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2615106]: (/etc/cron.hourly) starting 0anacron
Apr 30 06:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2615106]: (/etc/cron.hourly) finished 0anacron
Apr 30 07:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2615876]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 07:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2615876]: (/etc/cron.hourly) starting 0anacron
Apr 30 07:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2615876]: (/etc/cron.hourly) finished 0anacron
Apr 30 07:15:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2616436]: (gukaifeng) CMD (date >> /home/gukaifeng/test123)
Apr 30 08:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2617251]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 08:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2617251]: (/etc/cron.hourly) starting 0anacron
Apr 30 08:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2617251]: (/etc/cron.hourly) finished 0anacron
Apr 30 09:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2617748]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 09:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2617748]: (/etc/cron.hourly) starting 0anacron
Apr 30 09:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2617748]: (/etc/cron.hourly) finished 0anacron
Apr 30 09:15:02 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2617852]: (gukaifeng) CMD (date >> /home/gukaifeng/test123)
Apr 30 10:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2618215]: (root) CMD (run-parts /etc/cron.hourly)
Apr 30 10:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2618215]: (/etc/cron.hourly) starting 0anacron
Apr 30 10:01:01 iZ8vbf7xcuoq7ug1e7hjk5Z run-parts[2618215]: (/etc/cron.hourly) finished 0anacron
```

以我这里为例，我取其中一条：

```shell
Apr 30 05:15:01 iZ8vbf7xcuoq7ug1e7hjk5Z CROND[2614430]: (gukaifeng) CMD (date >> /home/gukaifeng/test123)
```

可以看到，4 月 30 日 05:15:01，在机器 iZ8vbf7xcuoq7ug1e7hjk5Z 上（这是我的机器名字，阿里云的机器默认名字就这样），以用户 gukaifeng 的身份执行了命令 `date >> /home/gukaifeng/test123`。



可以注意到日志里大部分的时间的秒都是 01 或者 02，主要是因为前文说过 crond 的检查间隔是 1 分钟，检查完就执行了，一般会在设定时间的第 1 秒或者第 2 秒开始执行并输出日志。

