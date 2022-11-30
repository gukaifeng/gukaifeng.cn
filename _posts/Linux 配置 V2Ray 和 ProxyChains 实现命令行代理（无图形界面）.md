---
title: Linux 配置 V2Ray 和 ProxyChains 实现命令行代理（无图形界面）
date: 2022-11-30 11:27:20
updated: 2022-11-30 11:27:20
categories: [技术杂谈]
tags: [Linux]
---



我在国内的服务器上进行某些操作的时候常常会被因为网络问题困扰，于是决定配置一下代理。

注意下本文是在没有图形界面的 Linux 上进行的，如果你用的是带图形界面的 Linux，网络上有更多更合适的文章。

**本文适合已经有 V2Ray 结点的同学（无论是自建的还是别人现成的）。**





## 1. 安装并配置 V2Ray



### 1.1. 安装 V2Ray

执行下面的命令安装（更新也可）V2Ray：

```shell
sudo bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

这个执行的脚本里面会有在 GitHub 下载 v2ray-core 的 release 版的操作，所以可能会比较慢。



另附卸载 V2Ray 的命令：

```shell
sudo bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

### 1.2. 修改配置文件

V2Ray 的配置文件挺长的，我的建议是从其他 V2Ray 客户端导出来比较好，不容易出错。

这里以 Windows 下的 V2ray 客户端 V2RayN 客户端为例（若你使用其他客户端，应当也有类似的导出选项）：

![V2RayN 截图 - 导出所选服务器为客户端配置](D:\Profession\hexo\_posts\Linux 配置 V2Ray 和 ProxyChains 实现命令行代理（无图形界面）\1.png)



选择 **导出所选服务器为客户端配置** 导出后是一个 json 配置文件，我们将其内容拷贝出来。

V2Ray 默认的配置文件在 `/usr/local/etc/v2ray/config.json`，我们将其打开：

```shell
sudo vim /usr/local/etc/v2ray/config.json
```

将我们刚刚在其他客户端导出的 json 内的配置，拷贝替换掉其中的内容，退出并保存即可。



### 1.3. 启动 V2Ray





最后启动 V2Ray：

```shell
systemctl start v2ray
```

我们可以再检查一下 v2ray 是否正确启动了：

```shell
$ $ systemctl status v2ray
● v2ray.service - V2Ray Service
   Loaded: loaded (/etc/systemd/system/v2ray.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/v2ray.service.d
           └─10-donot_touch_single_conf.conf
   Active: active (running) since Wed 2022-11-30 12:12:07 CST; 6s ago
     Docs: https://www.v2fly.org/
 Main PID: 2028733 (v2ray)
    Tasks: 8 (limit: 49489)
   Memory: 10.5M
   CGroup: /system.slice/v2ray.service
           └─2028733 /usr/local/bin/v2ray run -config /usr/local/etc/v2ray/config.json
```

有类似上述输出，`active (running)` 即表明 v2ray 正在运行。到这里有关 V2Ray 的工作就完成了。









## 2. 安装并配置 ProxyChains



因为我之前写过关于 ProxyChains 的文章，重写也是原样搬过来，不如直接看了。见 [Linux 配置 ProxyChains 使用代理](https://gukaifeng.cn/posts/linux-pei-zhi-proxychains-shi-yong-dai-li/)，**原样照做所有步骤**即可。

唯一需要我们关注的就是上面文章第 2 小节最后配置 `[ProxyList]` 的部分，很多同学不知道这里该写什么，我们回头看一下刚刚配置的 V2Ray 的 `/usr/local/etc/v2ray/config.json`，以我自己的为例，其中有下面一段需要关注：

```json
"inbounds": [
  {
    "tag": "socks",
    "port": 10808,
    "listen": "127.0.0.1",
    "protocol": "socks",
    "sniffing": {
      "enabled": true,
      "destOverride": [
        "http",
        "tls"
      ]
    },
    "settings": {
      "auth": "noauth",
      "udp": true,
      "allowTransparent": false
    }
  },
  {
    "tag": "http",
    "port": 10809,
    "listen": "127.0.0.1",
    "protocol": "http",
    "sniffing": {
      "enabled": true,
      "destOverride": [
        "http",
        "tls"
      ]
    },
    "settings": {
      "udp": false,
      "allowTransparent": false
    }
  }
],
```



`inbounds` 中有两个子字段，我们以第一个子字段最需要关注的一部分举例说明：

```json
"tag": "socks",
"port": 10808,
"listen": "127.0.0.1",
"protocol": "socks",
```

* `tag`：是个标签，我们不需要关注，这个名字并不重要。
* `port`：表示端口，即 V2Ray 的本地监听端口，这个端口自己可以随便改。
* `listen`：V2Ray 的监听地址，这里的值是环回地址。
* `protocol`：即协议。



我们回忆一下 proxychains 的配置中最后 `[ProxyList]` 下条目的格式：

```
type  ip  port [user pass]
```

其中 `type` 对应我们上面的 `protocol`，`ip` 对应 `listen`，`port` 对应 `port`，最后的 `user` 和 `pass` 在本例中不需要写。



所以本例中，依据我自己的配置，此处可以填：

```
[ProxyList]
socks5 127.0.0.1 10808

or

[ProxyList]
socks4 127.0.0.1 10808

or

[ProxyList]
http 127.0.0.1 10809
```



任意写哪个都可以，看你心情。到这里 ProxyChains 就配置好了。



## 3. 测试代理



配置好后的代理测试在文章  [Linux 配置 ProxyChains 使用代理](https://gukaifeng.cn/posts/linux-pei-zhi-proxychains-shi-yong-dai-li/) 内也有，不过本来也比较简单，这里再演示一遍。

这里就以国内通常无法访问的谷歌 [www.google.com](http://www.google.com/) 为例：

在不使用 proxychains 时，我们 `curl` 是无法访问谷歌的，经过漫长的等待后会提示超时：

```shell
$ curl www.google.com
curl: (7) Failed to connect to www.google.com port 80: Connection timed out
```

然后我们在 `curl` 前面加上 `proxychains4`：

```shell
$ proxychains4 curl www.google.com
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/libproxychains4.so
[proxychains] DLL init: proxychains-ng 4.16-git-9-g060801d
[proxychains] Strict chain  ...  127.0.0.1:10808  ...  www.google.com:80  ...  OK
<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="en">...</html>
```

可以看到我们成功访问了谷歌网站（输出太长了，`<html></html>` 标签内的内容我用 `...` 代替了）。