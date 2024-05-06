今天排查 Ceph 集群的 SLOW_OPS 警告的时候，发现 `ceph daemon` 命令查看不到我想要的信息。

例如，我想查看 osd.29 的 ops 信息，就会报错：

```shell
# ceph daemon osd.29 ops
Can't get admin socket path: unable to get conf option admin_socket for osd: b"error parsing 'osd': expected string of the form TYPE.ID, valid types are: auth, mon, osd, mds, mgr, client\n"
```

排查了一点时间后，结论是：**当 daemon 进程在 cephadm Orchestrator 管理的容器中运行时，套接字文件具有不同的位置，并且命令行工具 `ceph`（在容器外运行时）不会自动找到它。**

**所以我们需要指定套接字文件的具体位置，而非通过 daemon 的名字来试图让命令行工具 `ceph` 去查找。**

每个 daemon 的套接字文件，位于其所在宿主机上的 `/var/run/ceph/<fsid>/` 目录下，例如我这里：

```shell
# ll /var/run/ceph/1e2c2112-037f-11ef-b466-45c2ff826e1a/
total 0
drwxrwx--- 2  167  167 240 Apr 26 17:56 ./
drwxrwx--- 3 ceph ceph  60 Apr 26 11:43 ../
srwxr-xr-x 1  167  167   0 Apr 26 11:44 ceph-mgr.c3-hadoop-ceph-st38.bj.urnckz.asok=
srwxr-xr-x 1  167  167   0 Apr 26 11:43 ceph-mon.c3-hadoop-ceph-st38.bj.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:55 ceph-osd.11.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:55 ceph-osd.17.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:55 ceph-osd.2.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:56 ceph-osd.23.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:56 ceph-osd.29.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:56 ceph-osd.35.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:56 ceph-osd.41.asok=
srwxr-xr-x 1  167  167   0 Apr 26 17:56 ceph-osd.47.asok=
```

**不过要注意，我们无法直接在一台主机上使用其他主机上 daemon 的套接字。**

这时候，如果我们想看一个 daemon 的信息，使用对应的套接字就可以了。例如我这里想看我这台机器上 daemo osd.29 的 ops 信息：

```shell
# ceph daemon /var/run/ceph/1e2c2112-037f-11ef-b466-45c2ff826e1a/ceph-osd.29.asok ops
{
    "ops": [],
    "num_ops": 0
}
```

即使用套接字路径，替换 daemon 名字，即可~

