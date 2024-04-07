

先说一个总结，彻底（尽可能的）清理干净 Linux 缓存：

```shell
sync ; echo 3 | sudo tee /proc/sys/vm/drop_caches
```

\-

下面是关于 Linux 清理系统缓存方法的介绍。

我们可以通过操作写入 `/proc/sys/vm/drop_caches` 文件来清理 Linux 的缓存。

1. 释放 Pagecache：

   ```shell
   echo 1 > /proc/sys/vm/drop_caches
   ```

2. 释放可回收的 Slab 对象（包括 Dentry 和 Inode）：

   ```shell
   echo 2 > /proc/sys/vm/drop_caches
   ```

3. 释放 Pagecache 和可回收的 Slab 对象：

   ```shell
   echo 3 > /proc/sys/vm/drop_caches
   ```



有一些点我们需要关注下：

* 这是一个非破坏性操作，不会释放任何脏对象。

* 要增加此操作释放的对象数量，我们应该在写入 `/proc/sys/vm/drop_caches` 之前进行 `sync`。这将最大限度地减少系统上脏对象的数量，并创建更多要删除的候选对象。

* 写入 `/proc/sys/vm/drop_caches` 文件不是控制各种内核缓存（Inodes、Dentries、Pagecache 等）增长的手段。当系统上的其他地方需要内存时，这些对象会由内核自动回收。

* 使用此文件可能会导致性能问题。 由于它会丢弃缓存的对象，因此可能会花费大量 I/O 和 CPU 来重新创建删除的对象，尤其是在大量使用这些对象的情况下。 因此，不建议在测试或调试环境之外使用。

* 写入 `/proc/sys/vm/drop_caches` 文件的操作会被记录到系统内核日志中（可以往里面写 `4` 禁用），比如：

  ```shell
  # echo 3 > /proc/sys/vm/drop_caches
  # cat /var/log/kern.log
  ...
  Apr  7 15:56:31 tj5-s1-v6-tj5-128473-2yqgrsr3 kernel: [878266.423329] bash (3636078): drop_caches: 3
  ...
  ```




参考资料：https://www.kernel.org/doc/Documentation/sysctl/vm.txt

