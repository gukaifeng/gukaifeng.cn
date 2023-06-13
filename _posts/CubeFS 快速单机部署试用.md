


**注意：**

* CubeFS 是一个分布式文件系统，是应当部署在集群上的。

* 本文仅针对纯小白第一次上手，目的是让纯小白可以快速部署好 CubeFS 并使用。

* 本文不会对 CubeFS 的各种配置项做说明解释，只是给出一个固定的操作过程，使 CubeFS 快速运转起来。
* 真实的集群中的 CubeFS 的配置要复杂一些，但不会在本文中介绍。
* 本文也是我作为纯小白，第一次试用 CubeFS 的过程记录。





## 1. 编译 CubeFS



### 1.1. 安装依赖



```shell
yum install -y gcc-c++ \
               go \
               cmake \
               zlib-devel \
               bzip2-devel \
               maven
```





这些依赖 yum 中的版本已经足够了，所以就直接这样装了，自己手动装更高版本也可以。







### 1.2. 克隆源码仓库

克隆 CubeFS 源码仓库：

```shell
git clone https://github.com/cubefs/cubefs.git
```



进入 CubeFS 源码目录：

```shell
cd cubefs
```







### 1.3. 编译



编译需要 root 权限（使用 root 用户或其他用户的 sudo 权限来执行）：

```shell
make
```



这个编译里会有 maven 下载各种依赖 jar 包的过程（主要是为了编译给 Java 用的 `libcubefs`），如果网络有障碍的话可能会比较慢，可以考虑换个 maven 源或者开个代理什么的，这里就不说了。



然后 CubeFS 的这个 `make` 如果想多线程的话，比如 `make -j8`，可能会有一点错误（应该是由于编译的个别处有顺序要求），不过问题不大，多执行一两次就可以规避这个问题。



编译完成以后，会在 `./build/bin/` 目录下有一些产出：

```shell
[gukaifeng@localhost cubefs]$ ll ./build/bin/
total 238220
-rwxr-xr-x 1 root root  7444284 May  2 10:54 cfs-authtool
-rwxr-xr-x 1 root root  8678032 May  2 10:54 cfs-bcache
-rwxr-xr-x 1 root root 62079104 May  2 11:06 cfs-cli
-rwxr-xr-x 1 root root 17905312 May  2 10:56 cfs-client
-rwxr-xr-x 1 root root 13159632 May  2 10:56 cfs-fsck
-rwxr-xr-x 1 root root 16468456 May  2 11:20 cfs-preload
-rwxr-xr-x 1 root root 83364264 May  2 11:05 cfs-server
-rw-r--r-- 1 root root     4021 May  2 11:21 libcfs.h
-rw-r--r-- 1 root root 17174088 May  2 11:21 libcfs.so
-rw-r--r-- 1 root root  8067257 May  2 11:25 libcubefs-1.0-SNAPSHOT.jar
-rw-r--r-- 1 root root  9574854 May  2 11:25 libcubefs-1.0-SNAPSHOT-jar-with-dependencies.jar
```







## 2. 快速单机部署

### 2.1. 使用单机部署脚本



下面的命令仍然在 CubeFS 源码的根目录位置执行。



官方提供了一个一键部署的脚本 `./shell/deploy.sh`，其用法如下：



```shell
sh ./shell/depoly.sh </path/to/data> <network_interface_name>
```

* `/path/to/data`：数据目录，用于保存集群运行日志、数据及配置文件。
* `network_interface_name`：本机网卡的名字。



数据目录可以任意设置，比如我这里设定为 `/home/gukaifeng/cubefs-test`。

网卡名字可以通过 `ifconfig` 命令查看，例如我这里网卡名字就是 `ens33`：

```shell
$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.17.129  netmask 255.255.255.0  broadcast 192.168.17.255
        inet6 fe80::20c:29ff:fe7a:60d9  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7a:60:d9  txqueuelen 1000  (Ethernet)
        RX packets 1416  bytes 114619 (111.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1034  bytes 83744 (81.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```





所以我这里要执行的命令为（需要 root 权限，普通用户需要加 `sudo`）：

```shell
sh ./shell/depoly.sh /home/gukaifeng/cubefs-test ens33
```





执行完以后，如果正常，控制台应当是没有报错信息的。



### 2.2. 验证“集群”状态



我们可以使用下面的命令查看集群的运转状态：

```shell
./build/bin/cfs-cli cluster info
```

例如我这里：

```shell
$ ./build/bin/cfs-cli cluster info
[Cluster]
  Cluster name       : cfs_dev
  Master leader      : 172.16.1.101:17010
  Auto allocate      : Enabled
  MetaNode count     : 4
  MetaNode used      : 0 GB
  MetaNode total     : 0 GB
  DataNode count     : 4
  DataNode used      : 0 GB
  DataNode total     : 0 GB
  Volume count       : 0
  EbsAddr            : 
  LoadFactor         : 0
  BatchCount         : 0
  MarkDeleteRate     : 0
  DeleteWorkerSleepMs: 0
  AutoRepairRate     : 0
  MaxDpCntLimit      : 0
```

有这样的输出就说明集群启动成功了。

\-



我们也可以看下脚本具体启动了哪些进程：



```shell
$ ps -ef | grep cfs-server | grep master
root        2194       1  0 12:18 ?        00:00:02 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/master1.conf
root        2232       1  0 12:18 ?        00:00:00 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/master2.conf
root        2270       1  0 12:18 ?        00:00:02 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/master3.conf

$ ps -ef | grep cfs-server | grep meta
root        2310       1  0 12:18 ?        00:00:01 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/meta1.conf
root        2332       1  0 12:18 ?        00:00:00 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/meta2.conf
root        2352       1  0 12:18 ?        00:00:02 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/meta3.conf
root        2371       1  0 12:19 ?        00:00:01 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/meta4.conf

$ ps -ef | grep cfs-server | grep data
root        2392       1  0 12:19 ?        00:00:01 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/data1.conf
root        2412       1  0 12:19 ?        00:00:01 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/data2.conf
root        2432       1  0 12:19 ?        00:00:01 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/data3.conf
root        2452       1  0 12:19 ?        00:00:00 /home/gukaifeng/cubefs/build/bin/cfs-server -f -c /home/gukaifeng/cubefs-test/conf/data4.conf
```

可以看到，脚本启动了 3 个 master 结点，4 个meta 结点和 4 个 data 结点。



到这里，CubeFS 的服务就全部部署成功了，下面就是使用 CubeFS 了。





## 3. 创建并挂载卷



### 3.1. 创建卷



单机部署下，我们使用 http 模拟客户端像 master 请求的过程，创建一个卷：

```shell
curl -v "http://172.16.1.101:17010/admin/createVol?name=test&capacity=100&owner=cfs"
```

* `172.16.1.101:17010`：master1 的 IP 和监听端口（前面提过我们启动了 3 个 master，还有 master2 和 master3）。
* `admin`：表示管理员用户。
* `createVol`：创建卷的指令，`?` 后面是创建卷的相关参数。
  * `name`：卷名字。
  * `capacity`：卷大小，单位 GB。
  * `owner`：卷的持有者名字。



注意这里很多配置都是在 `./shell/deploy.sh` 脚本里设定的。

集群的配置还是比较复杂的，这里不解释任何东西，只是专注先跑起来。



有如下输出就说明卷创建成功了：



```shell
$ curl -v "http://172.16.1.101:17010/admin/createVol?name=test&capacity=100&owner=cfs"
*   Trying 172.16.1.101...
* TCP_NODELAY set
* Connected to 172.16.1.101 (172.16.1.101) port 17010 (#0)
> GET /admin/createVol?name=test&capacity=100&owner=cfs HTTP/1.1
> Host: 172.16.1.101:17010
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 100
< Content-Type: application/json
< Date: Tue, 02 May 2023 19:56:54 GMT
< 
* Connection #0 to host 172.16.1.101 left intact
{"code":0,"msg":"success","data":"create vol[test] successfully, has allocate [10] data partitions"}
```



### 3.2. 挂载卷



首先我们需要安装一下 fuse：

```shell
modprobe fuse
yum install -y fuse
```

\-





这里得回忆一下我在第 2 小节使用部署脚本时的命令：

```shell
sh ./shell/depoly.sh /home/gukaifeng/cubefs-test ens33
```

要说的是我这里的数据目录在 `/home/gukaifeng/cubefs-test`，脚本里生成的各种配置文件也存在这里了。



我们需要编辑一下客户端挂载的配置文件，在我这里的路径为  `/home/gukaifeng/cubefs-test/cubefs-test/conf/client.conf`，即数据目录下的 `conf/client.conf`（注意换成你自己的路径）：



```shell
vim /home/gukaifeng/cubefs-test/cubefs-test/conf/client.conf
```



我们需要修改里面的 `volName` 和 `owner` 两个字段，将这两个字段修改为和前面创建卷时的参数一致，就是前面 curl 请求 matser 创建卷时链接 `?` 后面的参数，`volName` 对应链接中的 `name`，`owner` 是一样的。我这里修改后如下：



```
{
  "masterAddr": "172.16.1.101:17010,172.16.1.102:17010,172.16.1.103:17010",
  "mountPoint": "/home/gukaifeng/cubefs-test/client/mnt",
  "volName": "test",
  "owner": "cfs",
  "logDir": "/home/gukaifeng/cubefs-test/client/log",
  "logLevel": "debug"
}
```

这里记一下 `client.conf` 里的 `mountPoint` 的挂载点参数值，我们一会挂载的时候这里就是挂载点。



然后就可以挂载了（如果不是 root 用户的话需要加 `sudo`，或者修改 fuse 的配置，否则会失败）：



```shell
./build/bin/cfs-client -c ~/cubefs-test/conf/client.conf
```



可能会有个提示，但不用管：

```
2023/05/03 04:12:25 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
```

挂载成功后，我们也可以查到挂载的进程：

```shell
$ ps -ef | grep cfs | grep client
root        5178       1  0 04:12 ?        00:00:55 /home/gukaifeng/cubefs/build/bin/cfs-client -f -c /home/gukaifeng/cubefs-test/conf/client.conf
```





### 3.3. 验证挂载卷



记得挂载点就是我们前面提到的 `client.conf` 的 `mountPoint` 字段写的路径，  
我这里是 `/home/gukaifeng/cubefs-test/client/mnt`，可以通过 `df` 命令查看：



```shell
$ df -Th
Filesystem                 Type         Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs     3.8G     0  3.8G   0% /dev
tmpfs                      tmpfs        3.8G     0  3.8G   0% /dev/shm
tmpfs                      tmpfs        3.8G  9.0M  3.8G   1% /run
tmpfs                      tmpfs        3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/almalinux-root xfs           57G  7.8G   50G  14% /
/dev/sda1                  xfs         1014M  168M  847M  17% /boot
tmpfs                      tmpfs        774M     0  774M   0% /run/user/1000
cubefs-test                fuse.cubefs  100G     0  100G   0% /home/gukaifeng/cubefs-test/client/mnt
```

最后一条就是我们刚刚挂载的，容量也是我们创建卷时指定的 `capacity=100` 即 100G，符合预期，挂载成功。







## 4. 使用文件系统



### 4.1. 我们的数据存在哪里



我们现在操作挂载目录（我这里是 `/home/gukaifeng/cubefs-test/client/mnt`），就是在操作挂载的 CubeFS 文件系统了。



但是数据存在哪呢？毕竟我们没有配置本地磁盘，或者对象存储等作为数据存储。



其实已经配置过了，因为我们使用的是部署脚本 `./shell/deploy.sh`，里面都帮我们做完了，毕竟我们只是想快速体验一下 CubeFS 罢了。



一键部署脚本将我们的数据存放在了本机目录下 `/home/gukaifeng/cubefs-test/data1/disk/ ` 目录下：



* `/home/gukaifeng/cubefs-test/`：我们启动一键部署脚本时配置的数据目录。
* `data1/`：数据结点目录（我们在第 2.2 小节说过，这个脚本启动了 4 个 data 结点，所以还有 `data2/`、`data3/` 和 `data4/`，这里就不多解释了）。
* `data1/disk/`：数据结点 data1 下的磁盘。



这里简单解释一下：CubeFS 是本应当在分布式集群中部署的，所以 4 个 data 结点进程应当是部署在 4 台不同的机器上的，每个机器都有自己的磁盘。由于我们是单机部署的，所以 4 个 data 结点进程都在一台机器上，部署脚本用每个 data 结点目录下的 `disk/` 目录来模拟其磁盘。



我们可以随便看一个 data 结点的磁盘，大概内容是下面这样的：

```shell
$ ll /home/gukaifeng/cubefs-test/data2/disk/
total 32
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_10_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_1_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_2_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_4_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_5_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_6_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_8_128849018880
drwxr-xr-x 3 root root 4096 May  3 04:54 datapartition_9_128849018880
```



### 4.2. 读写 CubeFS



我这里简单使用 fio 这个工具进行读写测试，注意这里不会解释 fio 的用法，只是简单对比挂载点和 data 结点的磁盘目录在读写前后的状态，以验证 CubeFS 正常运转。我们先安装 fio：

```shell
yum install -y fio
```





\-





在测试开始前，我们先看一下挂载点 `mnt/` 目录和任意 data 结点的磁盘目录大小：

```shell
$ du -sh /home/gukaifeng/cubefs-test/client/mnt/
0       /home/gukaifeng/cubefs-test/client/mnt/

$ du -sh /home/gukaifeng/cubefs-test/data1/disk/
62M     /home/gukaifeng/cubefs-test/data1/disk/
```



然后我们执行测试：

```shell
fio -filename=/home/gukaifeng/cubefs-test/client/mnt/fio-test-file -direct=1 -iodepth 1 -thread -rw=rw -ioengine=psync -bs=128k -size=1G -numjobs=5 -group_reporting -name="IORWTest"
```

这里我们暂且只需要关注参数 `-filename`，其值应当为挂载点下的一个不存在的文件名，我们这个命令就是要写入并读取这个文件。



这个命令的大概意思就是并发 5 个线程写一个 1G 大小的文件。



我这里大概需要执行 1 分钟，然后会打印输出一个性能分析报告，我们暂且不关注，这里就不给出了。



我们看看挂载点与任意 data 结点的磁盘目录，在 fio 操作后的变化：



```shell
$ du -sh /home/gukaifeng/cubefs-test/client/mnt/
1.0G    /home/gukaifeng/cubefs-test/client/mnt/

$ du -sh /home/gukaifeng/cubefs-test/data1/disk/
1.3G    /home/gukaifeng/cubefs-test/data1/disk/
```

可以看到还是比较符合预期的，到这里我们就已经验证了，我们单机部署的 CubeFS，部署和使用都是没有问题的了。



\-

单机版的 CubeFS 快速部署、使用，就到这里了 ~

## 5. 停止单机“集群”



当我们不再需要这个集群，可以停止掉。



官方提供了一个停止集群的脚本（同样需要 root 或 `sudo` 权限）：



```shell
sh ./shell/stop.sh
```

执行完以后，可以再查看一下进程，可以发现所有相关进程都已经被停止了：

```shell
$ ps -ef | grep cfs
gukaife+    2558    1722  0 12:29 pts/0    00:00:00 grep --color=auto cfs
```



\-

卸载后，挂载点的内容自然也就没了：

```shell
$ ll /home/gukaifeng/cubefs-test/client/mnt/
total 0
```

我们可以再次启动集群并挂载（不需要重新创建卷），我们之前 fio 写入的文件 fio-test-file 就又可以看到了：



```shell
$ ll /home/gukaifeng/cubefs-test/client/mnt/
total 1048576
-rw-r--r-- 1 gukaifeng gukaifeng 1073741824 May  3 05:50 fio-test-file
```



这里就不演示过程了，不过得注意下我们改过 `client.conf`，一键部署脚本会重置这个配置文件，还得重新改一下。



