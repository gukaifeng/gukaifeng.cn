---
title: 解决使用 Nginx 的 Wordpress 修改固定链接后 404 的问题
date: 2021-09-03 20:45
categories: [技术杂谈]
tags: [WordPress,Nginx]
toc: true
---





### 1. 问题场景

在固定链接中选择结构为文章名后，再打开文章页面提示 404。

<!--more-->

### 2. 解决方案

编辑 nginx 配置文件，我这里 nginx 安装在 /etc/nginx/ 目录下

```shell
sudo vim /etc/nginx/nginx.conf
```

找到 server{} 字段，在其中加入

```
rewrite /wp-admin$ $scheme://$host$uri/ permanent;
```

然后再在 server{} 中的 location / {} 字段最后，加入

```
if (-f $request_filename/index.html){  
    rewrite (.*) $1/index.html break;  
}  
if (-f $request_filename/index.php){  
    rewrite (.*) $1/index.php;  
}  
if (!-f $request_filename){  
    rewrite (.*) /index.php;  
}  
```

最后重启 nginx 即可

```shell
sudo nginx -s reload
```



最后附上修改后的部分示例代码，仅供参考。

```
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  gukaifeng.cn;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        root   html;
        index  index.php index.html index.htm;

        if (-f $request_filename/index.html){
            rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
    }
    
    rewrite /wp-admin$ $scheme://$host$uri/ permanent;

    location ~* \.php$ {
    ... more code
```

