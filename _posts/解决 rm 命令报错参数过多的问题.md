有些时候我们想删除一个目录内的全部内容，但是由于 rm 命令也是有参数数量上限的，所以直接删除会报错。

比如我想删除 `mnt/` 目录下的所有文件：

```shell
$ rm mnt/*
-bash: /bin/rm: Argument list too long
```

解决方法：

1. 使用 `find` 命令结合 `xargs`：

   ```
   find mnt/ -type f -exec rm {} +
   ```

   这个命令会使用 `find` 查找 `mnt/` 目录下的所有文件，并将它们作为参数传递给 `rm` 命令。`+` 表示将尽可能多的文件传递给 `rm` 以避免参数列表过长。

2. 使用 `find` 命令结合 `xargs` 和 `rm`：

   ```
   find mnt/ -type f | xargs rm
   ```

   这个命令会使用 `find` 查找 `mnt/` 目录下的所有文件，并将它们通过管道传递给 `xargs rm` 命令，以删除这些文件。


