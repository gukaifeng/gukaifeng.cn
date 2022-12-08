---
title: Linux 中 curl 命令的常用方法
date: 2021-12-09 18:02:33
updated: 2022-12-09 00:08:33
categories: [技术杂谈]
tags: [Linux,CentOS,curl]
toc: true
---







## 1. 什么是 `curl`？



`curl` 是一个使用 URL 语法传输数据的命令行工具和库。



`curl` 是开源的，项目地址为 https://github.com/curl/curl ，官网为 https://curl.se/ 。

0

`curl` 其支持的协议有 `DICT`、`FILE`、`FTP`、`FTPS`、`GOPHER`、`HTTP`、`HTTPS`、`IMAP`、`IMAPS`、`LDAP`、`LDAPS`、`MQTT`、`POP3`、`POP3S`、`RTMP`、`RTMPS`、`RTSP`、`SCP`、`SFTP`、`SMB`、`SMBS`、`SMTP`、`SMTPS`、`TELNET` 和 `TFTP`。



`curl` 命令被设计为无需用户交互即可工作。



`curl` 提供了大量有用的技巧，例如代理支持、用户身份验证、FTP 上传、HTTP post、SSL 连接、cookie、文件传输恢复等。`curl` 提供的功能的数量可能会让我们眼花缭乱头晕炫目。





## 2. `curl` 基本用法



>`curl` 的功能非常强大，非常多，本文只会介绍笔者认为比较常用的用法。详细文档请参阅 [man curl](https://man7.org/linux/man-pages/man1/curl.1.html) 和 [man curl-config](https://man7.org/linux/man-pages/man1/curl-config.1.html)。
>
>同样因为 `curl` 的功能实在是太多了，所以本文大概会偶尔追加更新一些我自己逐渐新用到的功能。



先说 `curl` 命令的基本格式很简单：

```shell
curl [options] URLs
```

不适用选项的时候，会直接打印出页面源码，例如：

```html
$ curl www.baidu.com
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

注意这里的 `www.baidu.com` 等价于 `http://www.baidu.com`。`curl` 支持很多协议，所以在需要查看 URL 中的确切内容时，建议写完整协议，因有些网站 `https` 的内容可能和 `http` 的是不一样的。







## 3. `curl` 常用选项



`curl` 选项非常多，这里结合场景，介绍几个我认为比较常用的。



这里再写一遍 curl 的基本用法：

```shell
curl [options] URLs
```



### 3.1. 保存页面或页面上的资源



`-o` 参数表示将资源保存到本地，后接存储保存资源的文件路径。



````
curl -o filename_to_save URL
````

如果 URL 是一个网页，那么将保存网页源码；如果 URL 是某个具体资源（如图片、视频文件）则将此资源存在本地。

例如：

```shell
# 将 https://gukaifeng.cn/ 的网页源码存在 ~/Downloads/gukaifeng.cn 文件中

curl -o ~/Downloads/gukaifeng.cn  https://gukaifeng.cn/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 48414  100 48414    0     0  1074k      0 --:--:-- --:--:-- --:--:-- 1099k
```



```shell
# 将 https://gukaifeng.cn/ 网站中的 avatar.png 保存到本地的 ~/Downloads/avatar.png

$ curl -o ~/Downloads/avatar.png  https://gukaifeng.cn/img/avatar.png
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  167k  100  167k    0     0  3155k      0 --:--:-- --:--:-- --:--:-- 3155k
```

这里的下载的文件名支持正则哦！可以批量下载。

### 3.2. 跟随页面跳转

如果我们 `curl` 的页面设置了自动跳转，那么默认情况 curl 得到的还是跳转前的网页。

要想支持页面跳转，需加 `-L` 参数。



### 3.3. 指定代理

指定代理选项为 `-x`。

```shell
curl -x proxy_host:port URL
```

例如：

```shell
curl -x 127.0.0.1:10808 https://www.google.com
```



不过你需要确保自己的代理可用。

我自己不是很喜欢这样使用代理，我的方法可以见 [Linux 配置 V2Ray 和 ProxyChains 实现命令行代理](https://gukaifeng.cn/posts/linux-pei-zhi-v2ray-he-proxychains-shi-xian-ming-ling-xing-dai-li-wu-tu-xing-jie-mian/)。当然了，每个人喜好不同，反正都能实现目标就好了。

