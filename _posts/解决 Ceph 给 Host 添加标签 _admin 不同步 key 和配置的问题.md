今天给一些 Ceph 集群一些 Host 添加 `_admin` 标签的时候，发现 `ceph.client.admin.keyring` 和 `ceph.conf` 没有同步。

排查了下发现，配置和标签的绑定列表是空的，即下面命令没有输出：

```shell
# ceph orch client-keyring ls
```

很好解决，在一台有 `ceph.client.admin.keyring` 和 `ceph.conf`  的机器上执行：

```shell
ceph orch client-keyring set client.admin label:_admin
```

然后再看一下配置和标签的绑定情况，像这样的输出就是成功了：

```shell
# ceph orch client-keyring ls
ENTITY        PLACEMENT     MODE       OWNER  PATH                                 
client.admin  label:_admin  rw-------  0:0    /etc/ceph/ceph.client.admin.keyring  
```

已有标签 `_admin` 的 Host 的不会被同步，需要删除标签再重新添加。

\-

经过我的排查，这个问题是在 `cephadm bootstarp` 时，添加了参数 `--no-minimize-config` 导致的。添加了这个参数后，引导过程没有给第一台机器添加 `_admin` 标签，也没有配置 key 和标签的绑定。

我尝试在测试集群重新部署测试后发现，Ceph 的使用最小化配置和在 `_admin` 标签机器上同步配置这两个操作是冲突的。只有使用了最小化配置，才能正确同步，即便是在引导阶段使用了参数 `--no-minimize-config`，然后再根据上面的解决方案重设 key 和标签的绑定，Ceph 的配置也仍然会被强制替换成最小化的。