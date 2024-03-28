今天使用 fio 3.9 测试文件系统的时候，遇到了一个报错：

```shell
smalloc: OOM. Consider using --alloc-size to increase the shared memory available.
fio: filesetup.c:1482: alloc_new_file: Assertion `0' failed.
```

并不复杂，smalloc 分配共享内存失败了，OOM 错误，但是我机器的内存是足够大的：

```shell
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           125G         15G         19G        4.7G         90G        103G
Swap:            0B          0B          0B
```

所以并不是物理机内存不足的问题，我们看一下 fio 提示的解决方案，参数 `--alloc-size` 的解释：

```shell
.. option:: --alloc-size=kb
	Allocate additional internal smalloc pools of size `kb` in KiB.  The
	``--alloc-size`` option increases shared memory set aside for use by fio.
	If running large jobs with randommap enabled, fio can run out of memory.
	Smalloc is an internal allocator for shared structures from a fixed size
	memory pool and can grow to 16 pools. The pool size defaults to 16MiB.

	NOTE: While running :file:`.fio_smalloc.*` backing store files are visible
	in :file:`/tmp`.
```

fio 文档中说明，在启用了 `randommap` 测试大工程的时候，fio 可能会耗尽内存。Smalloc 是一个用于从固定大小的内存池共享结构的内部分配器，可以增长到 16 个内存池。池大小默认为 16 MiB。

问题在于 Smalloc 是用于 fio 进程内自己分配自己使用内存的方法，所以在不修改其可分配大小的前提下，无论我们机器的内存多大，都还是会 OOM。所以我们一定要修改 `--alloc-size` 的值。



通过帮助信息可以看到，`--alloc-size` 的默认值是是 16384 KiB：

```shell
$ fio -h
...
--alloc-size=kb       Set smalloc pool to this size in kb (def 16384)
...
```

这个值看起来不是很大，我们可以通过调大这个值（注意单位是 KiB），来解决这个 OOM 问题。

具体调多大不太好知道，我的方法是逐步调大直到不再 OOM，或者直接调很大让他一定不会 OOM。







