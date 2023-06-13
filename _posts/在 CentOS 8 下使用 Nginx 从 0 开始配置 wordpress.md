




本文 Linux 系统环境为 CentOS 8.2。

搭建 Typecho开发环境需要安装并配置以下这些

1. nginx
2. MySQL
3. php & php-fpm
4. wordpress

开始前的几个建议，避免踩坑：  
1\. 在系统中单独创建一个具有 root 权限的用户来配置 wordpress 环境；  
2\. 在 mysql 中创建一个用作 wordpress 数据库的独立的 database；  
3\. 在 mysql 中创建一个新用户，并赋予操作 wordpress 那个数据库的全部权限；  
4\. 安装 php 的时候要格外注意，要安装一些依赖，以及一些扩展，搞错了的话会比较麻烦。

这里仅指导新手第一次成功配置 wordpress，具体某些配置项的作用大家后面再自行了解。

下面逐个解决上面列出的，这里假定你已经以一个具有 root 权限的用户登录到 CentOS。

<!--more-->

### 1. Nginx

#### 1.1 安装 nginx

Nginx 安装比较简单，直接输入命令安装即可：

```shell
sudo yum install nginx
```



#### 1.2 配置 nginx 使其支持 php

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
location / {
    root   html;
    index  index.php index.html index.htm;
}

location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```

其他配置不需要改动。

#### 1.3 启动 nginx

输入命令 `sudo nginx` 即可启动 nginx。

另：  
重启 nginx: `sudo nginx -s reload`  
停止 nginx: `sudo nginx -s stop`

### 2. Mysql

#### 2.1 安装 mysql-server

```shell
sudo yum install mysql-server
```

#### 2.2 启动并连接 mysql

##### 2.2.1 启动 mysql

```shell
service mysqld start
```

##### 2.2.2 连接 mysql

先以 root 用户连接 mysql

```shell
mysql -uroot -p
```

这里 root 默认是没有密码的，所以提示输入密码的时候，直接按回车就可以了。

#### 2.3 新建 wordpress 数据库

```sql
CREATE DATABASE wordpress;
```

这里新建了一个名为 `wordpress` 的新 database，专门给 wordpress 用的。

#### 2.4 新建 mysql 用户并赋权

```sql
CREATE USER 'gukaifeng'@'localhost' IDENTIFIED BY 'gukaifeng';
GRANT ALL ON wordpress.* TO 'gukaifeng'@'localhost';
```

这里我们创建了一个名为 `gukaifeng` 的用户，密码也是 `gukaifeng`（`IDENTIFIED BY` 后面的是密码），  
然后给这个用户赋予了操作我们刚刚新建的名为 `wordpress` 数据库的全部权限。

#### 2.5 关闭 mysql 连接

```sql
EXIT
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
tar -zxvf php-7.4.23.tar.gz
cd php-7.4.23/
```



#### 3.2 安装 Php & php-fpm

在这里输入

```shell
./configure --enable-fpm --with-mysqli --with-zlib --with-curl
```

注意这里是容易踩坑的，这 4 个选项一个都不能少，具体用处这里先不解释了。

**\* 常见问题**

1. 错误信息：`Package 'libxml-2.0', required by 'virtual:world', not found`

   解决方案：`sudo yum install libxml2-devel`

2. 错误信息：`Package 'sqlite3', required by 'virtual:world', not found`

   解决方案：`sudo yum install sqlite-devel`

3. 错误信息：`Package 'libcurl', required by 'virtual:world', not found`

   解决方案：`sudo yum install libcurl-devel`

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

找到 `user` 字段，值改为 `wordpress`，即`user = wordpress`， 这个用户就是 linux 专门操作 wordpress 的那个用户，我的是 `wordpress`，你就是你的。  
`group` 字段不用改， 就默认的 `nobody` 就行，没什么影响。

##### 3.3.2 编辑 php-fpm.conf

```shell
sudo vim /usr/local/etc/php-fpm.conf
```

找到 include 字段，一般是最后一行，默认是 `include=NONE/etc/php-fpm.d/*.conf`，把这个路径改为我们上面配置的 www.conf 的正确路径。  
我这里是 `include=/usr/local/etc/php-fpm.d/*.conf`，改完后保存退出。

##### 3.3.3 可选/建议

如果文件不存在，我们应该阻止 nginx 将请求传递到 php-fpm 后端，这很重要，可以防止任意脚本注入。

```shell
sudo vim /usr/local/php/php.ini
```

找到 `cgi.fix_pathinfo` 字段，删除其前面的 `;` 注释，将其值改为 0，即 `cgi.fix_pathinfo=0`。



#### 3.4 启动 php-fpm

```shell
sudo /usr/local/bin/php-fpm
```

如果没有输出任何内容，就说明启动成功了。

### 4. Wordpress

#### 4.1 下载并解压 wordpress 的 tar.gz 压缩包

这是 wordpress 中文官网的下载页面：https://cn.wordpress.org/download/#download-install

复制下来你要下载的 wordpress 版本的 tar.gz 的链接（非最新版在 https://cn.wordpress.org/download/releases/）。  
我这里下载的版本是最新的 5.8，链接是 https://cn.wordpress.org/latest-zh_CN.tar.gz。

然后命令行输入

```shell
wget https://cn.wordpress.org/latest-zh_CN.tar.gz
```

这样 wordpress 就下载到服务器本地了。同样的，如果你下载太慢的话，也可以用其他方式，比如先下载到自己的电脑，再上传到服务器，这里就不讲述这个操作了。

解压下载下来的 `latest-zh_CN.tar.gz` 

```shell
tar -zxvf latest-zh_CN.tar.gz
```

然后会解压出来一个名为 `wordpress` 的文件夹，进入此目录

```shell
cd wordpress/
```

#### 4.2 修改 wordpress 配置文件

```shell
vim wp-config-sample.php
```

找到下面的字段，并填入我们在步骤 2 中在 mysql 中配置的信息，其他地方暂时不需要改。

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );  // 这里是你的 database 名字，我的是 wordpress

/** MySQL database username */
define( 'DB_USER', 'gukaifeng' );  // 这里是你刚创建的 mysql 新用户的名字，我的是 gukaifeng

/** MySQL database password */
define( 'DB_PASSWORD', 'gukaifeng' );  // 这里是你刚创建的 mysql 新用户的密码，我的也是 gukaifeng

/** MySQL hostname */
define( 'DB_HOST', '127.0.0.1' );  // 这里改成 127.0.0.1，填 localhost 我这里无效，但不知道为什么
```

然后修改配置文件名字

```shell
mv wp-config-sample.php wp-config.php
```



#### 4.3 将 wordpress 目录中的内容移动到 nginx 的 root 目录中

root 目录是 nginx 配置文件里那个

先删除 root 目录下原来的内容

```shell
sudo rm -rf /usr/share/nginx/html/*
```

然后把我们的 wordpress 内容移动过去

```shell
sudo mv ./* /usr/share/nginx/html/
```

**\* 修改 root 目录所有者**，要把这个 root 目录的所有者改成和 php 用户一样的，不然每次安装插件等一些时候，都要输入 ftp 信息。

我们上面 php-fpm 的 user 已经设置为了 `wordpress`，这里也一样

```shell
sudo chown -R wordpress /usr/share/nginx/html
```



### 5. 开始使用 wordpress

在浏览器中输入你的服务器 ip 或者域名，即可进入 wordpress 开始页面。

根据提示依次配置好 站点标题、用户名、密码、电子邮箱、对搜索引擎可见性 即可，这里的用户名和密码就是 wordpress 后台管理页面的用户与密码。

到这里 wordpress 配置就全部完成了！





