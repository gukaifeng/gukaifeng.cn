
> 我刚刚踩了一个坑，就是 MySQL 数据库的大小远远超过了期望中的大小，导致了一些问题。排查后发现是 binlog 占用空间过高所致。所以在不关闭 binlog 的情况下，就需要修改 binlog 的保留时间。

MySQL 中以往都是使用变量 `expire_logs_days` 来控制 binlog 保留时间的，单位是天。

在 8.0 以后的版本中，又新增了变量 `binlog_expire_logs_seconds` 来更精确的以秒为单位的控制 binlog 保留时间。

我们可以看一下：

```sql
mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.26    |
+-----------+
1 row in set (0.00 sec)

mysql> show variables like "%expire_logs%";
+----------------------------+---------+
| Variable_name              | Value   |
+----------------------------+---------+
| binlog_expire_logs_seconds | 2592000 |
| expire_logs_days           | 0       |
+----------------------------+---------+
2 rows in set (0.00 sec)00 sec)
```

可以看到，我的 MySQL 版本为 8.0.26，关于 binlog 过期时间的设置有两个，就是我们刚刚提到过的。

在 8.0 以后的版本中，`binlog_expire_logs_seconds` 的优先级更高，其默认值 2592000 秒刚好是 30 天整。

查阅官方文档 [MySQL :: MySQL 8.0 Reference Manual :: 17.1.6.4 Binary Logging Options and Variables](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_expire_logs_seconds) 还可以得知以下内容：

1. `expire_logs_days` 已弃用（但仍可用），我们应当 `binlog_expire_logs_seconds` 用来控制 binlog 保留时间。

2. 如果在 MySQL 启动时 `binlog_expire_logs_seconds` 和已弃用的 `expire_logs_days` 都没有设置，那就是按 `binlog_expire_logs_seconds` 的默认值 2592000 即 30 天来保留 binlog。

3. 如果两者都设定了非零值，那按照 `binlog_expire_logs_seconds` 来，忽略 `expire_logs_days` 并给出一个警告信息。

4. 如果只设置了其中一个为非零值的话，那就那这个非零值的变量来。

5. 如果二者之一已经是非零值了，那在运行时我们不能将另一个设为非零值。比如，`binlog_expire_logs_seconds` 默认值是 2592000，非零，所以你要是想启用 `expire_logs_days` 的话，需要先把 `binlog_expire_logs_seconds` 设成 0。

其实总结下来，在 8.0 以后的 MySQL 版本，除非有特别需求，我们都应该只使用 `binlog_expire_logs_seconds`，放弃 `expire_logs_days`（保持其值为 0）。
