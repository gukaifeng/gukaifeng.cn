---
title: Unable to load authentication plugin 'caching_sha2_password'
date: 2022-12-05 18:52:33
updated: 2022-12-05 18:52:33
categories: [数据库]
tags: [Linux,MySQL]
---





## 1. 报错信息



使用某工具连接 MySQL 服务器时，报错：

```
Unable to load authentication plugin 'caching_sha2_password'.
```

此报错信息翻译过来是说，无法加载身份验证插件 `caching_sha2_password`。



## 2. 错误分析



此错误的**根本原因**是：  
MySQL 在 8 版本后使用了新的身份验证插件 `caching_sha2_password`，  
而在 8 以前的身份验证插件是 `mysql_native_password`。



我这里的 MySQL 版本为 8.0.28，我们可以查看下当前的默认身份验证插件的确是 `caching_sha2_password`：

```sql
mysql> show variables like "default_authentication_plugin";
+-------------------------------+-----------------------+
| Variable_name                 | Value                 |
+-------------------------------+-----------------------+
| default_authentication_plugin | caching_sha2_password |
+-------------------------------+-----------------------+
1 row in set (0.00 sec)
```



此错误的**直接原因**是：  
我们拿着 8 以前版本的 MySQL 客户端（或者使用 8 以前版本 MySQL 连接器的某工具）来连接 8 以后版本的 MySQL 服务器，我们自然拿不出 MySQL 服务器所需要的身份验证插件 `caching_sha2_password`。





## 3. 解决方案



我们已经知道了问题出在哪里，那么解决起来就比较容易了，有三个方案：

1. **（推荐）**升级我们的 MySQL 客户端版本（或某工具使用的 MySQL 驱动器）到 8 以后，以使用新版验证插件 `caching_sha2_password`。
2. （不推荐）将 MySQL 服务器全局的默认验证插件更改为旧的 `mysql_native_password`。
3. （不推荐）将 MySQL 服务器针对指定户登录时的验证插件更改为旧的 `mysql_native_password`。



\-

**方案 1（推荐）：**升级 MySQL 客户端版本（或某工具使用的 MySQL 连接器）到 8 以后



注意哦，这里升级的是 MySQL 客户端的版本（因为此错误是由于 MySQL 客户端拿不出 MySQL 服务器指定的身份验证插件导致的），不涉及 MySQL 服务器升级时的复杂操作（比如可能需要考虑的数据备份等）。升级 MySQL 客户端的方法多种多样，网上一搜一大把，这里就不说了。



如果你用的是使用 MySQL 连接器的某工具，则可以查看其内是否已有 8 以后版本的连接器可选，如果有则直接设置。如果没有，可自行在 [MySQL Community Downloads](https://dev.mysql.com/downloads/)下载所需连接器，记得选 8 以上版本的就行。





\-



**方案 2（不推荐）：**修改 MySQL 服务器全局的默认验证插件



**此方案应当仅用于测试环境，不建议用于生产环境。**



**此方案的修改生效后，仅对新创建的用户生效，原有用户仍需要手动修改（方案 3）。**



编辑 MySQL 服务器配置文件 `/etc/my.cnf`：

```shell
vim /etc/my.cnf
```

在其中的 `[mysqld]` 部分下添加以下内容：

```properties
[mysqld]
default_authentication_plugin=mysql_native_password
```

完成后，使用 `systemctl restart mysqld` 命令（或其他你使用的 MySQL 管理方式）重启 MySQL 服务器即可。

我们可以检查一下变量 `default_authentication_plugin` 的值以确定修改成功：

```sql
mysql> show variables like "default_authentication_plugin";
+-------------------------------+-----------------------+
| Variable_name                 | Value                 |
+-------------------------------+-----------------------+
| default_authentication_plugin | mysql_native_password |
+-------------------------------+-----------------------+
1 row in set (0.01 sec)
```





\-



**方案 3（不推荐）：**修改 MySQL 服务器中指定户登录时的验证插件



**此方案应当仅用于测试环境，不建议用于生产环境。**



修改用户 `'username'@'host'` 的验证插件为 `mysql_native_password`，密码为 `passwd`。命令如下：

```
ALTER USER 'username'@'host' IDENTIFIED WITH mysql_native_password BY 'passwd';
```

此命令修改立即生效。





