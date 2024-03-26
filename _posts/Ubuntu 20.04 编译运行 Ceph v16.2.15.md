



## 1. Linux 发行版本和 Ceph 版本的选择



下表是 Ceph 官方提供的 Ceph 版本与 Linux 系统版本的支持表格：

|               | Reef (18.2.z) | Quincy (17.2.z) | Pacific (16.2.z) | Octopus (15.2.z) |
| ------------- | ------------- | --------------- | ---------------- | ---------------- |
| Centos 7      |               |                 | A                | B                |
| Centos 8      | A             | A               | A                | A                |
| Centos 9      | A             |                 |                  |                  |
| Debian 10     | C             |                 | C                | C                |
| Debian 11     | C             | C               | C                |                  |
| OpenSUSE 15.2 | C             |                 | C                | C                |
| OpenSUSE 15.3 | C             | C               |                  |                  |
| Ubuntu 18.04  |               |                 | C                | C                |
| Ubuntu 20.04  | A             | A               | A                | A                |
| Ubuntu 22.04  | A             |                 |                  |                  |

其中 A、B、C 的含义如下：

- **A**: Ceph 提供软件包，并对其中的软件进行了全面的测试。
- **B**: Ceph 提供软件包，并对其中的软件进行了基本测试。
- **C**: Ceph 仅提供软件包，尚未对这些版本进行任何测试。



写这篇文章时，Ceph 19 刚刚发布，业内普遍使用的还是 16.2.z，16.2.15 是 16.2.z 当前的最新版本。

我这次安装 Ceph 主要是学习目的，而非公司的生产环境。所以参照上表，学习 v16.2.15 最合适的版本是 CentOS 7/8 和 Ubuntu 20.04。又因为 CentOS 已经不再维护了，且我个人对 Ubuntu 更感兴趣，所以就选择了 Ubuntu 20.04 了。



> 根据个人经验，强烈不建议选择上表之外的 Linux 系统发行版和 Ceph 版本的组合，也不建议选择非 A 的组合，可能会有各种意想不到的问题难以解决。
>
> 我踩过了很多的坑，Ceph 很复杂，对于官方没有提供软件包也没有测试过的环境中出现的那些问题，个人想要解决非常困难，白白给自己增加难度事小，Ceph 的稳定性不可控事大。





## 2. 注意事项



### 2.1. 磁盘容量问题



这里要提醒注意下磁盘容量，对于 Ceph v16.2.15 的编译，磁盘容量不能太小，否则会空间不足。

我这个虚拟机是新创建的、干净的，跑起来就只编 Ceph，100 GiB 的磁盘空间（因为实际给到 `/` 的空间会少一些）是不够的：

```shell
root@ubuntu:~# df -Th
Filesystem                        Type      Size  Used Avail Use% Mounted on
...
/dev/mapper/ubuntu--vg-ubuntu--lv ext4       96G   96G     0 100% /
...
```

最终由于磁盘空间不足导致编译安装失败。

扩容之后，编译安装操作全部完成以后，磁盘用量是这样的：

```shell
# df -Th
Filesystem                        Type      Size  Used Avail Use% Mounted on
...
/dev/mapper/ubuntu--vg-ubuntu--lv ext4      250G   99G  140G  42% /
...
```

我最后是扩容到了 256 GiB 的磁盘容量，因为也要留出空间去跑 Ceph 和其他程序。读者可以自行考虑合适的磁盘大小。



### 2.2. 用户权限问题



请使用 root 用户直接操作，不要使用其他用户 sudo 操作，也不要从其他用户切到 root 操作，有坑！

请使用 root 用户直接操作，不要使用其他用户 sudo 操作，也不要从其他用户切到 root 操作，有坑！

请使用 root 用户直接操作，不要使用其他用户 sudo 操作，也不要从其他用户切到 root 操作，有坑！

-

避免重点偏移，本文就不讲作者在非 root 操作时遇到的坑了。



### 2.3. 网络问题



Ceph 的编译安装过程涉及大量的网络下载，需要自行解决网络问题，我实际操作时使用了代理（但下面的操作过程没有写这个）。





## 3. 下载与编译



其实只要避开前面说过的版本、磁盘、权限和网络问题，Ceph 的编译安装就很简单了。上面几点看着没啥，其实都是泪。

不过仍然要说下，Ceph 的编译时间很长，建议使用配置较高的机器（或者像我一样等到地老天荒，我使用了 4 个核编译了一个下午）。



下面进入正题。



1. 克隆仓库，进入 Ceph 目录并切换到 v16.2.15 Tag：

   ```shell
   git clone git://github.com/ceph/ceph
   cd ceph
   git checkout v16.2.15
   ```

2. 克隆依赖仓库：

   ```shell
   git submodule update --init --recursive
   ```

3. 安装依赖，Ceph 的依赖巨多，好在有官方的脚本：

   ```shell
   ./install-deps.sh
   ```

4. 执行 `do_cmake.sh` 创建 build/ 目录：

   ```shell
   ./do_cmake.sh -DCMAKE_BUILD_TYPE=RelWithDebInfo
   ```
   注：`-DCMAKE_BUILD_TYPE=RelWithDebInfo` 表示编译 Release 版本，不加的话编译的是 Debug 版本。

5. 进入 build/ 目录并开始编译：

   ```shell
   cd build/
   make
   ```

   注：这里耗时较长，建议加参数  `-j` 并行编译。

6. 安装：

   ```shell
   make install
   ```



安装完成以后，可用的 Ceph 相关的命令有很多（我目测是 150 个左右）。











