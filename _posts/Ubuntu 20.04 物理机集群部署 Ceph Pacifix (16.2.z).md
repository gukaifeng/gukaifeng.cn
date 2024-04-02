本文将部署的 Ceph 版本为 Pacific (16.2.15)。

本文环境有三台机器，信息如下：

| 序号 | Hostname     | IP          | Linux 发行版                     | Linux 内核 |
| ---- | ------------ | ----------- | -------------------------------- | ---------- |
| 1    | hostname1.cn | 10.38.214.1 | Ubuntu 20.04.2 LTS (Focal Fossa) | 5.4.54-2   |
| 2    | hostname2.cn | 10.38.214.2 | Ubuntu 20.04.2 LTS (Focal Fossa) | 5.4.54-2   |
| 3    | hostname3.cn | 10.38.214.3 | Ubuntu 20.04.2 LTS (Focal Fossa) | 5.4.54-2   |

注：写这篇文章时我是使用公司机器操作的，所以对 hostname 和 ip 进行了替换，但对文章内容的理解应该没有影响。

参考文档：https://docs.ceph.com/en/pacific/install/



## 1. Ceph 管理工具 Cephadm



### 1.1. Cephadm 简介



安装 Ceph 的方法很多，本文使用官方最推荐的 Cephadm 工具进行。



官方是这样描述 Cephadm 的：cephadm 用于安装和管理使用容器和 systemd 的 Ceph 集群，并与 CLI 和仪表板 GUI 紧密集成。

* Cephadm 仅支持 Octopus (15.2.z) 和更新版本。
* Cephadm 与编排 API 完全集成，并完全支持用于管理集群部署的 CLI 和仪表板功能。
* Cephadm 需要容器支持（以 Podman 或 Docker 的形式）和 Python 3。



简单来说，就是从 Ceph 15 以后，官方摒弃了过往非常复杂的 Ceph 集群管理方法（旧方法仍然可用，但本文不做介绍），使用 Cephadm 进行了大一统。



> 如果想要将 Ceph 部署在 K8S 集群里面，官方建议使用工具 [Rook](https://rook.io/)，本文不做介绍。



### 1.2. Cephadm 下载

官方提供两个方法，二选一。**但注意两种方法互斥，不要同时使用，逻辑不同可能出错。**





#### 1.2.1. 手动下载（推荐）



手动下载可以精确指定与 Ceph 版本匹配的 Cephadm。

```shell
wget https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
```

URL 中间 `pacific` 可以换成你指定的 Ceph 版本名，下载的就是对应的 Cephadm。也可以换成版本号，比如 `v16.2.14`、`v16.2.15` 等，填 `pacific` 就是下载 v16.y.z 最新的。当前 Pacific 最新的小版本是 v16.2.15，即填 `pacific` 与填 `v16.2.15` 等价。

添加可执行权限：

```shell
chmod +x cephadm
```

将 Cephadm 可执行文件移动到 `$PATH` 中，例如我这里放到 /usr/sbin/ 目录下：

```shell
mv cephadm /usr/sbin/
```

然后可以验证一下：

```shell
$ which cephadm
/usr/sbin/cephadm
```



#### 1.2.2. 包管理器安装

```shell
apt install -y cephadm
```

包管理器安装的 Cephadm 可能与我们期望部署的 Ceph 版本不匹配，想要匹配又相对麻烦（比如改源），可用但不推荐。





### 1.3. Cephadm 依赖



Cephadm 其实是个 Python 脚本，这里的依赖指的时使用 Cephadm 时的，有以下：

* Python3
* Systemd
* 用于运行容器的 Podman 或 Docker（本文使用 Docker）
* 时间同步（例如 chrony 或 NTP）
* 用于配置存储设备的 LVM2



这里大部分不需要我们额外解决，看下 Docker 和 Python3 就 OK。



#### 1.3.1. Docker



卸载现有冲突包：

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

配置 Docker 官方 GPG key 和 apt 源：

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

安装：

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```





#### 1.3.2. Python3



Cephadm 对 Python3 的版本没有太高要求，Ubuntu 20.04 自带的 Python 3.8.10 已经足够了。





## 2. 部署一个新的 Ceph 集群



Cephadm 通过在单个主机上“引导(bootstrapping)”创建新的 Ceph 集群，再扩展集群以包含所有追加的主机，然后部署所需的服务。



### 2.1. 引导第一台机器



> 我这里操作第一台机器：
>
> hostname：hostname1.cn
>
> ip：10.38.214.1



创建新 Ceph 集群的第一步是，在 Ceph 集群的第一台主机上运行 `cephadm bootstrap` 命令。

在 Ceph 集群的第一台主机上运行 `cephadm bootstrap` 命令会创建 Ceph 集群的第一个“监控守护进程(monitor daemon)”，并且该监控守护进程需要一个 IP 地址。我们需要将 Ceph 集群第一台主机的 IP 地址传递给 `ceph bootstrap` 命令。

> 注意：如果主机有多个网络或接口，需要使用可以和集群其他主机互通的那个 IP。



```shell
cephadm bootstrap --mon-ip *<mon-ip>*
```



在不加任何额外参数时，这个命令将进行以下操作：

* 在本地主机上为新集群创建监视器和管理器守护程序。
* 为 Ceph 集群生成新的 SSH 密钥并将其添加到 root 用户的 `/root/.ssh/authorized_keys` 文件中。
* 将公钥的副本写入 `/etc/ceph/ceph.pub`。
* 将最小配置文件写入 `/etc/ceph/ceph.conf`。需要此文件与新集群进行通信。
* 将 `client.admin` 管理（特权！）密钥的副本写入 `/etc/ceph/ceph.client.admin.keyring`。
* 将 `_admin` 标签添加到引导主机。默认情况下，任何具有此标签的主机都将（也）获得 `/etc/ceph/ceph.conf` 和 `/etc/ceph/ceph.client.admin.keyring` 的副本。



成功后，会有类似这样的输出：

```shell
Ceph Dashboard is now available at:

             URL: https://hostname1.cn:8443/
            User: admin
        Password: wqr5zz9mf3

Enabling client.admin keyring and conf on hosts with "admin" label
Enabling autotune for osd_memory_target
You can access the Ceph CLI as following in case of multi-cluster or non-default config:

        sudo /usr/sbin/cephadm shell --fsid c5e3da58-ee3f-11ee-a744-e3cf29ca2a71 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Or, if you are only running a single cluster on this host:

        sudo /usr/sbin/cephadm shell 

Please consider enabling telemetry to help improve Ceph:

        ceph telemetry on

For more information see:

        https://docs.ceph.com/en/pacific/mgr/telemetry/

Bootstrap complete.
```

且 `/etc/ceph/` 目录下会有对应的文件：

```shell
# ll /etc/ceph/
total 28
drwxr-xr-x   2 root root  4096 Mar 29 10:56 ./
drwxr-xr-x 128 root root 12288 Mar 29 10:28 ../
-rw-------   1 root root   151 Mar 29 10:56 ceph.client.admin.keyring
-rw-r--r--   1 root root   177 Mar 29 10:56 ceph.conf
-rw-r--r--   1 root root   595 Mar 29 10:54 ceph.pub
```



这里有一些生产环境比较常用的参数：

* `cephadm` 参数（接在命令 `cephadm` 后面，子命令 `bootstrap` 前面）：

  * `--docker`：表示容器使用 Docker。如果不加此参数，在系统里同时存在 Docker 和 Podman 时，Cephadm 会使用 Podman。

  * `--image`：指定 Ceph 镜像版本。例如指定 Ceph 镜像版本为 v16.2.15：

    ```shell
    cephadm --image quay.io/ceph/ceph:v16.2.15 bootstrap --mon-ip *<mon-ip>*
    ```

* `cephadm bootstrap` 参数（接在子命令 `bootstrap` 后面）：

  * `--output-keyring`：输出 Ceph 集群 key 的位置，默认为上面提到 `/etc/ceph/ceph.client.admin.keyring`。

  * `--output-config`：输出 Ceph 配置文件的位置，默认为上面提到的 `/etc/ceph/ceph.conf`。

  * `--output-pub-ssh-key`：输出 Ceph 集群 SSH 公钥的文件，默认为 `/etc/ceph/ceph.pub`。

  * `--output-dir`：输出上面三个配置文件的目录，默认为 `/etc/ceph/`。这个参数和上面几个同时设定时，会进行拼接。

  * `--allow-fqdn-hostname`：允许 hostname 为域名。生产环境机器的 hostname 很多都是域名，但 Cephadm 默认不接受域名形式的 hostname 的。

  * `--cluster-network`：指定集群子网，用于集群复制、恢复和心跳。使用 CIDR，格式为网络/掩码，如 `172.26.13.0/255.255.255.0`。不指定此参数 Cephadm 会使用公共网络引导，即 Ceph 集群内的复制、恢复和心跳等消息会和公共流量混在一起，性能会差一些。



### 2.2. 安装 Ceph Cli



Cephadm 不要求安装任何 Ceph 组件，也可以执行 `ceph`、`rbd`、`mount.ceph` 等常用命令。

执行 `cephadm shell`，可以进到容器中，然后执行相关命令。

也可以直接在容器外执行容器内的命令，例如：

```shell
# cephadm shell -- ceph -v
Inferring fsid b686c856-ed9d-11ee-a6b4-b568275d3c48
Using recent ceph image quay.io/ceph/ceph@sha256:fca03613a9adb7f53f497420e5bb97c2066766fae2447ba11179b916eafb38ba
ceph version 16.2.15 (618f440892089921c3e944a991122ddc44e60516) pacific (stable)
```

但官方建议我们安装 `ceph-common`，不依赖 `cephadm shell`，在容器外操作更快速（实测快很多，主要是 `cephadm shell` 太慢了）。

安装 `ceph-common`：

```shell
cephadm add-repo --release pacific
cephadm install ceph-common
```

安装好以后，我们分别使用命令 `cephadm shell -- ceph status` 和 `ceph status` 验证一下，他们的输出内容应该是一样的，比如我这里：

```shell
# cephadm shell -- ceph status
Inferring fsid b686c856-ed9d-11ee-a6b4-b568275d3c48
Using recent ceph image quay.io/ceph/ceph@sha256:fca03613a9adb7f53f497420e5bb97c2066766fae2447ba11179b916eafb38ba
  cluster:
    id:     b686c856-ed9d-11ee-a6b4-b568275d3c48
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 1 daemons, quorum hostname1.cn (age 89m)
    mgr: hostname1.cn.lcrzre(active, since 87m)
    osd: 0 osds: 0 up, 0 i
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

```shell
# ceph status
  cluster:
    id:     b686c856-ed9d-11ee-a6b4-b568275d3c48
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 1 daemons, quorum hostname1.cn (age 89m)
    mgr: hostname1.cn.lcrzre(active, since 87m)
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

到这里 Ceph Cli 就安装完成了。

官方后面的文档也都是使用了 Ceph Cli 的，所以本文后面也是。





我们继续在第一台机器上操作。

注意所有的机器都需要满足 1.3 中提到的依赖。



### 2.3. 分发 SSH 公钥



我们在 2.1 引导第一台机器的时候，SSH 生成了一个公钥文件（默认位置是 `/etc/ceph/ceph.pub`）。

Cephadm 通过 SSH 免密实现对集群中各个机器的操作（私钥藏起来了），所以我们得把这个 SSH 公钥，分发到每个集群的主机上。

```
ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>
```

这个命令会把引导机上的 SSH 公钥副本 `/etc/ceph/ceph.pub` 的内容，写入 `root@<new-host>` 的 `/root/.ssh/authorized_keys` 中，从而实现免密（引导机的 `/root/.ssh/authorized_keys` 在引导阶段就已经写入了）。

比如我这里要加两台机器，命令就是：

```shell
ssh-copy-id -f -i /etc/ceph/ceph.pub root@hostname2.cn
ssh-copy-id -f -i /etc/ceph/ceph.pub root@hostname3.cn
```

~~不过我实际上是手动粘过去的，因为公司有自己的另一套鉴权系统，我没有这几个机器的密码。~~



### 2.4. 添加机器



```shell
ceph orch host add <newhost> [<ip>] [<labels> ...]
```

* `newhost`：待加入机器的 hostname。
* `ip`：待加入机器的 IP。可选，但官方强烈建议提供 IP，如果没提供，Ceph 会通过 DNS 解析找 IP。
* `label`：标签，可以加很多个。如果没有指定 IP 的话，添加标签要使用 `--labels` 选项。



> 标签这里主要关注一个 `_admin`：
>
> 第一台引导机会自动加上这个标签。
>
> 集群的配置文件 `ceph.conf` 和 `client.admin` 的 key 文件会被同步到所有有 `_admin` 标签的机器上。
>
> 有 `_admin`  标签的结点可以用来提供 Ceph Cli 服务，即其他节点可以通过此节点使用 `cephadm shell` 或 `ceph` 命令操作集群。单个 `_admin` 节点也没什么问题，只是如果很多人同时操作集群的话压力会大一些。



我给我的另外两台机器加到集群里，我这里先不加标签了：

```shell
ceph orch host add hostname2.cn 10.38.214.2
ceph orch host add hostname3.cn 10.38.214.3
```

然后，我们可以使用命令 `ceph orch host ls` 看到我们集群里全部的机器，包括刚加入的：

```shell
# ceph orch host ls
HOST                                ADDR           LABELS  STATUS  
hostname1.cn  10.38.214.1   _admin          
hostname3.cn  10.38.214.3                   
hostname2.cn  10.38.214.2                  
3 hosts in cluster
```





### 2.5. 添加额外的监视器



典型的 Ceph 集群具有三到五个分布在不同主机上的监控守护进程。如果集群中有五个或更多节点，官方建议部署五个监视器。

我这个集群目前只有三台机器，所以我们需要三个有监视器进程的节点。

如果你集群中的机器都在一个子网内，那么是不需要手动添加监视器的，Cephadm 会自动将合适数量的监视器部署到随机的机器上。 

我这三台机器刚好都在一个子网，所以我们看 `ceph status` 中的 mon 行：

```shell
# ceph status
...
  services:
    mon: 3 daemons, quorum hostname1.cn,hostname2,hostname3 (age 46s)
...
```

我这三台机器中已经部署了三个监视器节点，就不需要再管了。

如果你的 Ceph 集群中结点不在一个网段，Cephadm 就不会自动给你部署除了引导机以外的监视器节点了，需要手动更改下监视器的位置。

所以我这里暂时不需要其他操作了。





### 2.6. 添加存储空间（OSD）



#### 2.6.1. Ceph OSD 对存储设备的要求



Ceph 不会在不可用的设备上部署 OSD。符合以下要求的存储设备，才是可用的：

1. 该设备必须没有分区。
2. 设备不得具有任何 LVM 状态。
3. 该设备没有被挂载。
4. 该设备不得包含文件系统。
5. 设备不得包含 Ceph BlueStore OSD。
6. 该设备必须大于 5 GB。





#### 2.6.2. 配置 Ceph 自动管理 OSD



执行命令：

```shell
ceph orch apply osd --all-available-devices
```

这会让 Ceph 自动在可用的存储设备上部署 OSD。有以下几点需要注意：

**不过要注意：**Ceph Orch 应用的效果是持久的。这意味着 `ceph orch apply` 命令完成后添加到系统的设备将被自动找到并添加到集群中。这还意味着 `ceph orch apply` 命令完成后新变得可用（例如通过切换）的设备也将被自动找到并添加到集群中。

运行上述命令后：

* 如果我们向集群添加新磁盘，它们将自动用于创建新的 OSD。
* 如果我们删除了一个 OSD 并清理了 LVM 物理卷，Ceph 将很快自动创建新的 OSD。

简单的说，这个命令会开启 Ceph 自动托管 OSD。



本文暂时不会关注如何手动添加 OSD。如果你想禁用托管（比如你想要手动添加 OSD 等），使用命令：

```shell
ceph orch apply osd --all-available-devices --unmanaged=true
```





#### 2.6.3. 使现有存储设备满足要求



我这里在执行上述命令后，Ceph 集群没有新加 OSD：

```shell
# ceph status
...
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
...
```

使用命令查看 OSD 可用的设备是空的，没有输出：

```shell
# ceph orch device ls

```



---



我检查了下，其实是因为我的这些磁盘没有满足 2.6.1 中提到的要求。

例如我这里在第一台机器上看一下：

```shell
root@tj5-s1-v6-tj5-128473-2yqgrsr3:~# lvdisplay

root@tj5-s1-v6-tj5-128473-2yqgrsr3:~# lsblk -o PATH,NAME,FSTYPE,FSSIZE,FSAVAIL,FSUSE%,MOUNTPOINT
PATH       NAME   FSTYPE   FSSIZE FSAVAIL FSUSE% MOUNTPOINT
/dev/loop0 loop0  squashfs  91.9M       0   100% /snap/lxd/24061
/dev/loop1 loop1  squashfs  55.6M       0   100% /snap/core18/2409
/dev/loop2 loop2  squashfs                       
/dev/loop3 loop3  squashfs    62M       0   100% /snap/core20/1494
/dev/loop4 loop4  squashfs  67.9M       0   100% /snap/lxd/22753
/dev/loop5 loop5  squashfs    64M       0   100% /snap/core20/2182
/dev/loop6 loop6  squashfs  39.1M       0   100% /snap/snapd/21184
/dev/loop7 loop7  squashfs  55.8M       0   100% /snap/core18/2812
/dev/sda   sda                                   
/dev/sda1  ├─sda1                                
/dev/sda2  ├─sda2 ext4       3.7G    3.2G     8% /boot
/dev/sda3  └─sda3 ext4     545.7G    507G     2% /
/dev/sdb   sdb                                   
/dev/sdb1  └─sdb1 ext4       3.4T    3.2T     0% /home/work/hdd1
/dev/sdc   sdc                                   
/dev/sdc1  └─sdc1 ext4       3.4T    3.2T     0% /home/work/hdd2
/dev/sdd   sdd                                   
/dev/sdd1  └─sdd1 ext4       3.4T    3.2T     0% /home/work/hdd3
/dev/sde   sde                                   
/dev/sde1  └─sde1 ext4       3.4T    3.2T     0% /home/work/hdd4
/dev/sdf   sdf                                   
/dev/sdf1  └─sdf1 ext4       3.4T    3.2T     0% /home/work/hdd5
/dev/sdg   sdg                                   
/dev/sdg1  └─sdg1 ext4       3.4T    3.2T     0% /home/work/hdd6
/dev/sdh   sdh                                   
/dev/sdh1  └─sdh1 ext4       3.4T    3.2T     0% /home/work/hdd7
/dev/sdi   sdi                                   
/dev/sdi1  └─sdi1 ext4       3.4T    3.2T     0% /home/work/hdd8
/dev/sdj   sdj                                   
/dev/sdj1  └─sdj1 ext4       3.4T    3.2T     0% /home/work/hdd9
/dev/sdk   sdk                                   
/dev/sdk1  └─sdk1 ext4       3.4T    3.2T     0% /home/work/hdd10
/dev/sdl   sdl                                   
/dev/sdl1  └─sdl1 ext4       3.4T    3.2T     0% /home/work/hdd11
/dev/sdm   sdm                                   
/dev/sdm1  └─sdm1 ext4       3.4T    3.2T     0% /home/work/hdd12
```

虽然我这里单台机器有 12 块磁盘，但是都是分区了，有挂载，有文件系统 ext4。好在 `lvdisplay` 命令没有输出，没有逻辑卷。所以我这里整个集群的 36 块磁盘，都不满足上述的 1、3、4 条。

所以我需要对这 36 块磁盘进行卸载并删除分区（删除分区的同时文件系统自动就没了），以我上面的第一个磁盘设备 `/dev/sdb` 为例。

卸载设备 `/dev/sdb` 有挂载的分区 `/dev/sdb1`：

```shell
umount /dev/sdb1
```

删除 `/dev/sdb` 的分区，使用 `fdisk` 命令进入交互页面，安装下面的操作删除分区，注意你如果有多个分区，可能需要多次输入 `d` 指令：

```shell
# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): M
The size of this disk is 3.7 TiB (4000225165312 bytes). DOS partition table format cannot be used on drives for volumes larger than 2199023255040 bytes for 512-byte sectors. Use GUID partition table format (GPT).

Entering protective/hybrid MBR disklabel.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): w

Command (m for help): q
```

然后重新加载下分区表就可以了：

```shell
partprobe
```

把所有磁盘都这样操作一遍就好了。



---



我写了个简单的脚本来做这个事，`disks` 换成你的磁盘列表：

```bash
#!/bin/bash

if [ "$(id -u)" != "0" ]; then
    echo "no permission"
    exit 1
fi

if [ -z "$BASH_VERSION" ]; then
    echo "this script must be run using bash"
    exit 1
fi

disks=(
    "/dev/sdb"
    "/dev/sdc"
    "/dev/sdd"
    "/dev/sde"
    "/dev/sdf"
    "/dev/sdg"
    "/dev/sdh"
    "/dev/sdi"
    "/dev/sdj"
    "/dev/sdk"
    "/dev/sdl"
    "/dev/sdm"
)

for disk in "${disks[@]}"
do
    sudo umount ${disk}*
done

for disk in "${disks[@]}"
do
    echo "d
    M
    d
    w
    q" | fdisk $disk
done

partprobe
```

然后在每个存在同样问题的机器上执行就好了。



#### 2.6.4. 检查 OSD 信息



配置完上述的所有操作后，**等几分钟后**，我们检查下 OSD 信息。

先用 `ceph -s` 看一下：

```shell
# ceph -s
  cluster:
    id:     c5e3da58-ee3f-11ee-a744-e3cf29ca2a71
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum hostname1.cn,hostname2,hostname3 (age 10h)
    mgr: hostname1.cn.nopizg(active, since 10h), standbys: hostname2.kncurq
    osd: 36 osds: 36 up (since 7m), 36 in (since 8m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   10 GiB used, 131 TiB / 131 TiB avail
    pgs:     1 active+clean
```

可以看到已经有了 36 个 OSD，可用存储空间也对得上。

再看下 Ceph 识别的设备：

```shell
# ceph orch device ls
HOST                                PATH      TYPE  DEVICE ID                                         SIZE  AVAILABLE  REFRESHED  REJECT REASONS                                                                 
hostname1.cn  /dev/sdb  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812401ca4d590  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdc  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812411cad387a  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdd  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812411cb598f5  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sde  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812421cbe4684  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdf  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812421cc767e2  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdg  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812431cd0b398  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdh  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812441cda2290  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdi  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812441ce4722d  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdj  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812451cee4fc2  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdk  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812461cf96559  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdl  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812461d039ab1  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname1.cn  /dev/sdm  hdd   PERC_H730P_Adp_64cd98f0a4fe92002d9812471d0ef7ce  3725G             13m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdb  hdd   INSPUR_66c92bf0009ec3eb2d9811ff1b1354c2          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdc  hdd   INSPUR_66c92bf0009ec3eb2d9812001b25f78d          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdd  hdd   INSPUR_66c92bf0009ec3eb2d9812011b38cbe8          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sde  hdd   INSPUR_66c92bf0009ec3eb2d9812021b4d8768          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdf  hdd   INSPUR_66c92bf0009ec3eb2d9812041b62310e          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdg  hdd   INSPUR_66c92bf0009ec3eb2d9812051b7746ef          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdh  hdd   INSPUR_66c92bf0009ec3eb2d9812061b8caf37          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdi  hdd   INSPUR_66c92bf0009ec3eb2d9812081ba28dbc          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdj  hdd   INSPUR_66c92bf0009ec3eb2d9812091bb84081          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdk  hdd   INSPUR_66c92bf0009ec3eb2d98120b1bce9925          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdl  hdd   INSPUR_66c92bf0009ec3eb2d98120c1be6c00c          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname3.cn  /dev/sdm  hdd   INSPUR_66c92bf0009ec3eb2d98120e1bfd630f          3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdb  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812131b8c06ee      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdc  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812151ba5248b      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdd  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812171bbf19a8      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sde  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812181bd9ada6      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdf  hdd   MR-SAS3316_6f80f41ffa0bf0002d98121a1bf376a0      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdg  hdd   MR-SAS3316_6f80f41ffa0bf0002d98121c1c10625c      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdh  hdd   MR-SAS3316_6f80f41ffa0bf0002d98121e1c2c99c2      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdi  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812201c4a1b2d      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdj  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812221c667a2d      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdk  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812231c83366c      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdl  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812251ca09452      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected        
hostname2.cn  /dev/sdm  hdd   MR-SAS3316_6f80f41ffa0bf0002d9812281cc1119d      3725G             12m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected
```

结果没问题，这里 `AVAILABLE` 列为空，`REJECT REASONS` 列显示 "Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected"，指的就是这个设备已经给 OSD 用上了。

到这里 OSD 的基本配置就完成了。



#### 2.6.5. 启用 libstoragemgmt （可选）



如果我们执行：

```shell
ceph orch device ls --wide
```

可能会看到这样的结果：

```shell
# ceph orch device ls --wide
HOST                                PATH      TYPE  TRANSPORT  RPM  DEVICE ID                                         SIZE  HEALTH  IDENT  FAULT  AVAILABLE  REFRESHED  REJECT REASONS                                                                 
...
tj5-s1-v6-tj5-128473-2yqgrsr3.kscn  /dev/sdg  hdd                   PERC_H730P_Adp_64cd98f0a4fe92002d9812431cd0b398  3725G          N/A    N/A               24m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected
...
```

其中，`HEALTH` 列为空，`IDENT` 和 `FAULT` 列为 `N/A`。这些字段由  [libstoragemgmt](https://github.com/libstorage/libstoragemgmt) 提供，在 Ceph 默认是禁用的（官方原因是与硬件并非 100% 兼容）。这几个字段在 Ceph DashBoard 中也会有。

在启用 libstoragemgmt 之前，我们需要先测试与我们硬件的兼容性，执行：

```shell
cephadm shell lsmcli ldl
```

如果结果形如（`Link Type` 和 `Health Status` 列可以正常显示值），则兼容：

```shell
Path     | SCSI VPD 0x83    | Link Type | Serial Number      | Health Status
----------------------------------------------------------------------------
/dev/sda | 50000396082ba631 | SAS       | 15P0A0R0FRD6       | Good
/dev/sdb | 50000396082bbbf9 | SAS       | 15P0A0YFFRD6       | Good
...
```

如果结果形如（`Link Type` 是 `No Support  `，`Health Status` 列是 `Unknown`），则不兼容：

```shell
Path     | SCSI VPD 0x83                    | Link Type  | Serial Number                    | Health Status
-----------------------------------------------------------------------------------------------------------
/dev/sda | 64cd98f0a4fe92002d9812391c3489f3 | No Support | 00f389341c3912982d0092fea4f098cd | Unknown      
/dev/sdb | 64cd98f0a4fe92002d9812401ca4d590 | No Support | 0090d5a41c4012982d0092fea4f098cd | Unknown      
...
```

如果兼容，我们可以启用 libstoragemgmt：

```shell
ceph config set mgr mgr/cephadm/device_enhanced_scan true
```







## 3. 使用 Ceph 简介



先看下官方对 **Ceph 存储集群(Ceph Storage Cluster)** 一词的描述：

>Ceph 存储集群是所有 Ceph 部署的基础，是以下几项的集合：
>
>* Ceph 监视器（Monitors）；
>* Ceph 管理器（Managers）；
>* 对象存储守护进程（OSDs）；
>* Ceph 元数据服务器（MDS，CephFS 专用，其他的不需要）。
>
>基于 RADOS，Ceph 存储集群由两种类型的守护进程组成：
>
>1. Ceph OSDs 将数据作为对象存储在存储节点上； 
>2. Ceph Monitor 维护集群映射的主副本。
>
>一个Ceph存储集群可能包含数千个存储节点。一个最小的系统将至少有一个 Ceph Monitor 和两个 Ceph OSD 守护进程用于数据复制。
>
>CephFS、Ceph RGW 和 Ceph RBD 都从 Ceph 存储集群读取数据并向其中写入数据。





也就是说，Ceph 底层的存储就是 RADOS，不管我们在上面跑 Ceph 文件系统、对象存储还是块存储，下面都是 RADOS。

Ceph 主要的使用方法有三种：

1. 使用 Ceph 文件系统，即 CephFS。
2. 使用 Ceph 对象存储，即 RGW（RADOS Gateway）。
3. 使用 Ceph 块存储，即 RBD（Rados Block Device）。



除了这三种方法以外，还有一种是我们自己编写程序，直接使用底下的 RADOS（毕竟上面三种底下也是 RADOS）。



## 4. 小结



这篇文章主要讲的是如何部署一个基本没有什么定制化的 Ceph 集群。

也没有深入讲具体怎么用 Ceph，跟着这篇文章，把 Ceph 集群部署起来就是目的，具体怎么用，我会写在其他文章里。

先就到这里，感谢 ~

