---
title: 在 CentOS 8 下使用 Nginx 从 0 开始配置 Typecho
date: 2021-09-05 20:55
updated: 2021-09-05 20:55
categories: [技术杂谈]
tags: [CentOS,Nginx,Typecho,SQLite,PHP,Blog]
toc: true
---



本文 Linux 系统环境为 CentOS 8.2。

搭建 typecho 开发环境需要安装并配置以下这些

1. nginx
2. SQLite
3. php & php-fpm
4. typecho

开始前的几个建议，避免踩坑：  
1\. 在系统中单独创建一个具有 root 权限的用户来配置 typecho 环境；  
\2. 安装 php 的时候要格外注意，要安装一些依赖，以及一些扩展，搞错了的话会比较麻烦。

这里仅指导新手第一次成功配置 typecho，具体某些配置项的作用大家后面再自行了解。

下面逐个解决上面列出的，这里假定你已经以一个具有 root 权限的用户登录到 CentOS，  
如果你直接在 root 用户下操作的话，下面的 `sudo` 就都不用加了。

<!--more-->

### 1. Nginx

#### 1.1 安装 nginx

Nginx 安装比较简单，直接输入命令安装即可：

```shell
sudo yum install nginx
```



#### 1.2 配置 nginx 使其支持 php 与 typecho

找到 nginx 的安装目录，里面有一个 `nginx.conf` 配置文件，修改它。  
我这里是默认安装目录 `/etc/nginx`，如果你的 nginx 安装在其他位置，就去其他位置找就可以。

```shell
sudo vim /etc/nginx/nginx.conf
```

找到 http 字段中的 server 字段，将其中的 

```
location / {
}
```

替换为

```
if (!-e $request_filename) {
    rewrite ^(.*)$ /index.php$1 last;
}

location / {
    root html;
    index index.php index.html index.htm;

    set $path_info "";
    set $real_script_name $fastcgi_script_name;
    if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
        set $real_script_name $1;
        set $path_info $2;
    }
    fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
    fastcgi_param SCRIPT_NAME $real_script_name;
    fastcgi_param PATH_INFO $path_info;
}

location ~ .*\.php(\/.*)*$ {
    include fastcgi.conf;
    fastcgi_pass  127.0.0.1:9000;
}
```

-

另外一个，如果你即将执行 nginx 启动命令的**不是 root 用户**，建议配置下面这些

还是 nginx.conf 文件，找到 `user` 字段，改成你的用户名，我这里是 `user typecho;`。

然后执行命令

```shell
sudo chown -R your_username /var/lib/nginx
```

我这里是 `sudo chown -R typecho /var/lib/nginx`。

不管怎样，一定要保证 nginx 的启动用户，和 `/var/lib.nginx` 这个目录的用户一致，不然后面写文章、传文件都可能会出现 500 错误。

其他配置不需要改动。

#### 1.3 启动 nginx

输入命令 `sudo nginx` 即可启动 nginx。

另：  
重启 nginx: `sudo nginx -s reload`  
停止 nginx: `sudo nginx -s stop`

### 2. SQLite

安装 SQLite

```shell
sudo yum install sqlite-devel
```



### 3. Php & php-fpm

#### 3.1 下载并解压 Php 的 tar.gz 压缩包

这是 php 官网的下载页面：https://www.php.net/downloads.php

复制下来你要下载的 php 版本的 tar.gz 的链接，我这里下载的版本是 7.4.23，链接是 https://www.php.net/distributions/php-7.4.23.tar.gz 。

然后命令行输入

```shell
wget https://www.php.net/distributions/php-7.4.23.tar.gz
```

这样 php 就下载到服务器本地了。如果你下载太慢的话，也可以用其他方式，比如先下载到自己的电脑，再上传到服务器，这里就不讲述这个操作了。

解压下载下来的 `php-7.4.23.tar.gz` 并进入此目录

```shell
tar -zxvf php-7.4.23.tar.gzcd php-7.4.23/
```



#### 3.2 安装 Php & php-fpm

在这里输入

```shell
./configure --enable-fpm --enable-mbstring --with-zlib --with-curl --with-openssl
```

注意这里是容易踩坑的，这几个选项一个都不能少，不然后面再改就比较麻烦，具体用处这里先不解释了。

**\* 常见问题**

1. 错误信息：`Package 'libxml-2.0', required by 'virtual:world', not found`

    解决方案：`sudo yum install libxml2-devel`

2. 错误信息：`Package 'libcurl', required by 'virtual:world', not found`

    解决方案：`sudo yum install libcurl-devel`

3. 错误信息：`Package 'openssl', required by 'virtual:world', not found`

    解决方案：`sudo yum install openssl-devel`

4. 错误信息：`Package 'oniguruma', required by 'virtual:world', not found`

    解决方案：https://www.cnblogs.com/architectforest/p/12433640.html

可能还会有其他类似问题，我暂时只遇到这三个，如果有其他问题，自己谷歌百度看看还需要安装哪些。

上面的命令完成后，会有提示 "Thank you for using PHP."，输入

```shell
make
```

```shell
sudo make install
```

php 安装完成。

#### 3.3 修改 Php & php-fpm 配置文件

先拷贝默认的配置文件到我们需要的目录下

```shell
sudo cp php.ini-development /usr/local/php/php.ini
sudo cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/www.conf
sudo cp sapi/fpm/php-fpm /usr/local/bin
sudo cp ~/php-7.4.23/sapi/fpm/php-fpm.conf /usr/local/etc/php-fpm.conf // 注意这里的 ~/php-7.4.23/ 换成你自己的 php 源码包的目录
```

##### 3.3.1 编辑 www.conf

```shell
vim /usr/local/etc/php-fpm.d/www.conf
```

找到 `user` 字段，值改为 `typecho`，即`user = typecho`， 这个用户就是 linux 专门操作 typecho 的那个用户，我的是 `typecho`，你就是你的。  
`group` 字段不用改， 就默认的 `nobody` 就行，没什么影响。

##### 3.3.2 编辑 php-fpm.conf

```shell
sudo vim /usr/local/etc/php-fpm.conf
```

找到 include 字段，一般是最后一行，默认是 `include=NONE/etc/php-fpm.d/*.conf`，把这个路径改为我们上面配置的 www.conf 的正确路径。  
我这里是 `include=/usr/local/etc/php-fpm.d/*.conf`，改完后保存退出。



#### 3.4 启动 php-fpm

```shell
sudo /usr/local/bin/php-fpm
```

如果没有输出任何内容，就说明启动成功了。

### 4. Typecho

#### 4.1 下载 typecho

建议下载 GitHub 上的最新版本，因为目前官网上的最新稳定版([1.1.17.10.30](https://github.com/typecho/typecho/releases/download/v1.1-17.10.30-release/1.1.17.10.30.-release.tar.gz))有个关于 cookie 的 bug。

```shell
git clone https://github.com/typecho/typecho.git
```

这样 typecho 就下载到服务器本地了。同样的，如果你下载太慢的话，也可以用其他方式，比如先下载到自己的电脑，再上传到服务器，这里就不讲述这个操作了。

clone 下来的是一个名为 `typecho-master` 的文件夹。

#### 4.2 将 typecho 目录中的内容移动到 nginx 的 root 目录中

root 目录是 nginx 配置文件里那个

先删除 root 目录下原来的内容

```shell
sudo rm -rf /usr/share/nginx/html/*
```

然后把我们的 typecho 中的内容移动过去

```shell
sudo mv ./* /usr/share/nginx/html/
```

**\* 修改 root 目录所有者**，要把这个 root 目录的所有者改成和 php-fpm 用户一样的，不然可能会出现权限问题。

我们上面 php-fpm 的 user 已经设置为了 `typecho`，这里也一样

```shell
sudo chown -R typecho /usr/share/nginx/html
```



### 5. 开始使用 typecho

在浏览器中输入你的服务器 ip 或者域名，即可进入 typecho开始页面。

根据提示依次配置好各项信息即可，这里的用户名和密码就是 typecho 后台管理页面的用户与密码。

到这里 typecho 配置就全部完成了！





