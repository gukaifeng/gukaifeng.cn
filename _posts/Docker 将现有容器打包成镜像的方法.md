


## 1. 场景

* 我们之前已经用 docker 创建好了一个容器。
* 我们对这个容器进行了修改，例如安装软件、修改文件、部署服务器等等。
* 我们现在要把修改后的这个容器制作成新的镜像，方便以后使用这个镜像直接创建与之相同的容器。



## 2. 命令说明

```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

关于 `OPTIONS`，有如下可选：

* `-a`：指定新镜像的作者。
* `-c`：使用 Dockerfile 指令来创建镜像。
* `-m`：提交生成镜像的说明信息。
* `-p`：在 commit 时，将容器暂停。



## 3. 实例演示

对于本文中的场景，关于上述的 `OPTIONS`，没有用到 `-c`，其他几个可以自己看情况选择。

下面看一个例子：

\-

首先在**宿主**机器上，执行 `docker ps -a`，查看本机上的所有容器：

```shell
[gukaifeng@dell2 ~]$ docker ps -a
CONTAINER ID   IMAGE                                              COMMAND                  CREATED         STATUS                       PORTS                                                                                                                                                                                                                                                                                NAMES
b02c76857587   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         21 hours ago    Up 21 hours                  0.0.0.0:42220->22/tcp, :::42220->22/tcp, 0.0.0.0:45331->10001/tcp, :::45331->10001/tcp, 0.0.0.0:45332->10002/tcp, :::45332->10002/tcp, 0.0.0.0:45333->10003/tcp, :::45333->10003/tcp, 0.0.0.0:45334->10004/tcp, :::45334->10004/tcp, 0.0.0.0:45335->10005/tcp, :::45335->10005/tcp   gkf-ycsb-client
b4b2173fcb2f   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         44 hours ago    Up 44 hours                  0.0.0.0:52220->22/tcp, :::52220->22/tcp, 0.0.0.0:55331->10001/tcp, :::55331->10001/tcp, 0.0.0.0:55332->10002/tcp, :::55332->10002/tcp, 0.0.0.0:55333->10003/tcp, :::55333->10003/tcp, 0.0.0.0:55334->10004/tcp, :::55334->10004/tcp, 0.0.0.0:55335->10005/tcp, :::55335->10005/tcp   lifuzhou_52220_mysql_dcompact
384c9eecf623   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         8 days ago      Up 8 days                    0.0.0.0:52210->22/tcp, :::52210->22/tcp, 0.0.0.0:55321->10001/tcp, :::55321->10001/tcp, 0.0.0.0:55322->10002/tcp, :::55322->10002/tcp, 0.0.0.0:55323->10003/tcp, :::55323->10003/tcp, 0.0.0.0:55324->10004/tcp, :::55324->10004/tcp, 0.0.0.0:55325->10005/tcp, :::55325->10005/tcp   lifuzhou_52210_mysql_slave
44ee4ec54ade   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         8 days ago      Up 8 days                    0.0.0.0:52200->22/tcp, :::52200->22/tcp, 0.0.0.0:55311->10001/tcp, :::55311->10001/tcp, 0.0.0.0:55312->10002/tcp, :::55312->10002/tcp, 0.0.0.0:55313->10003/tcp, :::55313->10003/tcp, 0.0.0.0:55314->10004/tcp, :::55314->10004/tcp, 0.0.0.0:55315->10005/tcp, :::55315->10005/tcp   lifuzhou_52200_mysql_master
38a67d4320be   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         6 weeks ago     Up 3 weeks                   0.0.0.0:52170->22/tcp, :::52170->22/tcp, 0.0.0.0:55301->10001/tcp, :::55301->10001/tcp, 0.0.0.0:55302->10002/tcp, :::55302->10002/tcp, 0.0.0.0:55303->10003/tcp, :::55303->10003/tcp, 0.0.0.0:55304->10004/tcp, :::55304->10004/tcp, 0.0.0.0:55305->10005/tcp, :::55305->10005/tcp   lifuzhou_52170_mysql
e67469d992f6   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         3 months ago    Exited (137) 4 weeks ago                                                                                                                                                                                                                                                                                          lifuzhou_52160_dcompact
4149104125df   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         3 months ago    Exited (137) 4 weeks ago                                                                                                                                                                                                                                                                                          lifuzhou_52150_todis
a352c270eecc   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         3 months ago    Exited (255) 8 weeks ago     0.0.0.0:52130->22/tcp, :::52130->22/tcp, 0.0.0.0:55081->10001/tcp, :::55081->10001/tcp, 0.0.0.0:55082->10002/tcp, :::55082->10002/tcp, 0.0.0.0:55083->10003/tcp, :::55083->10003/tcp, 0.0.0.0:55084->10004/tcp, :::55084->10004/tcp, 0.0.0.0:55085->10005/tcp, :::55085->10005/tcp   lifuzhou_52130_kvrocks
0e53dfb1a9a1   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         3 months ago    Exited (255) 4 weeks ago     0.0.0.0:39500->22/tcp, :::39500->22/tcp, 0.0.0.0:43961->10001/tcp, :::43961->10001/tcp, 0.0.0.0:43962->10002/tcp, :::43962->10002/tcp, 0.0.0.0:43963->10003/tcp, :::43963->10003/tcp, 0.0.0.0:43964->10004/tcp, :::43964->10004/tcp, 0.0.0.0:43965->10005/tcp, :::43965->10005/tcp   simoncao_39500_dcompact
e8bfa9ed104b   redis                                              "docker-entrypoint.s…"   6 months ago    Exited (0) 5 months ago                                                                                                                                                                                                                                                                                           redis-test
5d4f5056eb20   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         7 months ago    Exited (255) 7 months ago    0.0.0.0:52000->22/tcp, :::52000->22/tcp, 0.0.0.0:55001->10001/tcp, :::55001->10001/tcp, 0.0.0.0:55002->10002/tcp, :::55002->10002/tcp, 0.0.0.0:55003->10003/tcp, :::55003->10003/tcp, 0.0.0.0:55004->10004/tcp, :::55004->10004/tcp, 0.0.0.0:55005->10005/tcp, :::55005->10005/tcp   gukaifengserver_52000
b224a8564996   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         7 months ago    Exited (137) 7 months ago                                                                                                                                                                                                                                                                                         zengjingtao_52130_grafana
67eb3897ea0a   mysql/mysql-server:latest                          "/entrypoint.sh -p 2…"   11 months ago   Exited (1) 11 months ago                                                                                                                                                                                                                                                                                          zengjingtao-mysql
09a08cf18264   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         11 months ago   Exited (255) 10 months ago   0.0.0.0:2323->22/tcp, :::2323->22/tcp                                                                                                                                                                                                                                                temp-for-grpc
7328c67351ae   192.168.100.100:5000/terark/terark-centos:v0.9.4   "/usr/sbin/init"         13 months ago   Exited (137) 12 months ago                                                                                                                                                                                                                                                                                        zengjingtao-test
c56caddf5baf   mysql:8.0                                          "docker-entrypoint.s…"   14 months ago   Up 2 weeks                   33060/tcp, 0.0.0.0:13306->3306/tcp, :::13306->3306/tcp                                                                                                                                                                                                                               luantian-mysql8.0
524c622333d9   192.168.100.100:5000/terark/terark-centos:v0.9.1   "/usr/sbin/init"         14 months ago   Exited (137) 13 months ago                                                                                                                                                                                                                                                                                        gukaifeng
```

查看本机上的所有容器，我们要在这之中选择你要制作镜像的那一个。

我这里是选择了第一个：

```
b02c76857587   192.168.100.100:5000/terark/terark-centos:v0.9.5   "/usr/sbin/init"         21 hours ago    Up 21 hours                  0.0.0.0:42220->22/tcp, :::42220->22/tcp, 0.0.0.0:45331->10001/tcp, :::45331->10001/tcp, 0.0.0.0:45332->10002/tcp, :::45332->10002/tcp, 0.0.0.0:45333->10003/tcp, :::45333->10003/tcp, 0.0.0.0:45334->10004/tcp, :::45334->10004/tcp, 0.0.0.0:45335->10005/tcp, :::45335->10005/tcp   gkf-ycsb-client
```

记下你要制作镜像的容器的 `CONTAINER ID`，本例中为 `b02c76857587`，然后在命令行输入写好参数的 2. 中的命令：

```shell
docker commit -a "gukaifeng" -m "example: create image" b02c76857587 example:v0.1
```

现在，镜像就制作完成了。

我们可以通过 `docker images` 查看现有镜像，可以在下面看到我们新创建的镜像 `example:v0.1` 了（第一条就是）：

```shell
REPOSITORY                                  TAG       IMAGE ID       CREATED          SIZE
example                                     v0.1      22682b82e574   4 seconds ago    2.34GB
ycsb                                        v1.0      1004fe361fdb   33 minutes ago   2.34GB
redis                                       latest    7614ae9453d1   6 months ago     113MB
192.168.100.100:5000/terark/terark-centos   v0.9.5    4a910aeb4719   13 months ago    1.99GB
lifuzhoutendisclient                        latest    969dc9fbfd5f   13 months ago    5.86GB
mysql/mysql-server                          latest    1504607f1ce7   13 months ago    391MB
192.168.100.100:5000/terark/terark-centos   v0.9.4    880a96e96c88   14 months ago    1.73GB
192.168.100.100:5000/terark/terark-centos   <none>    74b3e33da4e6   14 months ago    1.73GB
192.168.100.100:5000/terark/terark-centos   <none>    af2f4b580fb6   14 months ago    1.72GB
192.168.100.100:5000/terark/terark-centos   v0.9.3    2fc6c88bbbaf   14 months ago    1.72GB
192.168.100.100:5000/terark/terark-centos   <none>    7e283a32979f   14 months ago    1.64GB
192.168.100.100:5000/terark/terark-centos   v0.9.2    b168f8056ce8   14 months ago    1.42GB
192.168.100.100:5000/terark/terark-centos   v0.9.1    a5f32b4e8406   14 months ago    1.34GB
terark/terark-centos                        v0.9.1    a5f32b4e8406   14 months ago    1.34GB
terark-centos-k8s-build                     latest    421cb7d2cf78   15 months ago    5.79GB
192.168.100.100:5000/terark/terark-centos   v0.9      4a3cb7404a46   15 months ago    1.27GB
terark-centos                               v0.9      4a3cb7404a46   15 months ago    1.27GB
terark/terark-centos                        v0.9      4a3cb7404a46   15 months ago    1.27GB
terark-centos                               v0.8      e1eb3678d4dc   15 months ago    867MB
mysql                                       8.0       e646c6533b0b   15 months ago    546MB
centos                                      centos8   300e315adb2f   18 months ago    209MB
centos                                      latest    300e315adb2f   18 months ago    209MB
```

