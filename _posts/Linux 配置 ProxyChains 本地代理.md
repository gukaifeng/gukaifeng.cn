---
title: Linux 配置 ProxyChains 本地代理
date: 2022-11-29 23:33:20
updated: 2022-11-29 23:33:20
categories: [技术杂谈]
tags: [Linux]
---





为了在 Linux 中使用命令行代理，本文介绍如何安装并配置 ProxyChains **本地代理**。

ProxyChains 在 GitHub 上开源，项目地址：[proxychains-ng](https://github.com/rofl0r/proxychains-ng)。

配置成功后，只需要在需要走代理的命令前加上 `proxychains` 即可，例如：

```shell
$ proxychains curl www.google.com
$ proxychains git clone git@github.com:gukaifeng/hexo.git
```

简单好用，下面说用法。

## 1. 安装 ProxyChains

首先克隆项目到本地，并进入目录：

```shell
git clone git@github.com:rofl0r/proxychains-ng.git
cd proxychains-ng
```

配置：

```shell
./configure --prefix=/usr --sysconfdir=/etc
```

编译安装（需要系统内有 C 编译器，最好是 gcc）：

```shell
make
sudo make install
```



安装完成后，在命令行输入 `proxychains4` （应当已经可以自动补全了）可以看到以下内容：

```shell
$ proxychains4

Usage:	proxychains4 -q -f config_file program_name [arguments]
	-q makes proxychains quiet - this overrides the config setting
	-f allows one to manually specify a configfile to use
	for example : proxychains telnet somehost.com
More help in README file
```

到这里就安装成功了。





## 2. 编辑配置文件

ProxyChains 会按以下顺序以此寻找配置文件，即下列配置文件优先级由高到低：

1. 由环境变量 `PROXYCHAINS_CONF_FILE` 指定的配置文件 或 执行 `proxychains4` 命令时由 `-f` 参数指定的配置文件。
2. `./proxychains.conf`，即运行命令 `proxychains4` 时所在目录下的 `proxychains.conf`（如果有的话）。
3. `$(HOME)/.proxychains/proxychains.conf`。
4. `$(sysconfdir)/proxychains.conf`，通常是 `/etc/proxychains.conf`。



可以看出来，优先级越低的是越通用的。

我这里选择编辑 `/etc/proxychains.conf`，这样用起来比较方便，并且每个用户都能用。



在我们克隆的仓库里的 `src` 目录中，有一个配置文件模板，我们将其拷贝一份到 `/etc` 下，并打开编辑：

创建并编辑 ``/etc/proxychains.conf`：

```shell
sudo cp ./src/proxychains.conf /etc/
vim /etc/proxychains.conf
```

其默认内容如下：

```conf
# proxychains.conf  VER 4.x
#
#        HTTP, SOCKS4a, SOCKS5 tunneling proxifier with DNS.


# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
#dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#round_robin_chain
#
# Round Robin - Each connection will be done via chained proxies
# of chain_len length
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped).
# the start of the current proxy chain is the proxy after the last
# proxy in the previously invoked proxy chain.
# if the end of the proxy chain is reached while looking for proxies
# start at the beginning again.
# otherwise EINTR is returned to the app
# These semantics are not guaranteed in a multithreaded environment.
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain or round_robin_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

## Proxy DNS requests - no leak for DNS data
# (disable all of the 3 items below to not proxy your DNS requests)

# method 1. this uses the proxychains4 style method to do remote dns:
# a thread is spawned that serves DNS requests and hands down an ip
# assigned from an internal list (via remote_dns_subnet).
# this is the easiest (setup-wise) and fastest method, however on
# systems with buggy libcs and very complex software like webbrowsers
# this might not work and/or cause crashes.
proxy_dns

# method 2. use the old proxyresolv script to proxy DNS requests
# in proxychains 3.1 style. requires `proxyresolv` in $PATH
# plus a dynamically linked `dig` binary.
# this is a lot slower than `proxy_dns`, doesn't support .onion URLs,
# but might be more compatible with complex software like webbrowsers.
#proxy_dns_old

# method 3. use proxychains4-daemon process to serve remote DNS requests.
# this is similar to the threaded `proxy_dns` method, however it requires
# that proxychains4-daemon is already running on the specified address.
# on the plus side it doesn't do malloc/threads so it should be quite
# compatible with complex, async-unsafe software.
# note that if you don't start proxychains4-daemon before using this,
# the process will simply hang.
#proxy_dns_daemon 127.0.0.1:1053

# set the class A subnet number to use for the internal remote DNS mapping
# we use the reserved 224.x.x.x range by default,
# if the proxified app does a DNS request, we will return an IP from that range.
# on further accesses to this ip we will send the saved DNS name to the proxy.
# in case some control-freak app checks the returned ip, and denies to 
# connect, you can use another subnet, e.g. 10.x.x.x or 127.x.x.x.
# of course you should make sure that the proxified app does not need
# *real* access to this subnet. 
# i.e. dont use the same subnet then in the localnet section
#remote_dns_subnet 127 
#remote_dns_subnet 10
remote_dns_subnet 224

# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000

### Examples for localnet exclusion
## localnet ranges will *not* use a proxy to connect.
## note that localnet works only when plain IP addresses are passed to the app,
## the hostname resolves via /etc/hosts, or proxy_dns is disabled or proxy_dns_old used.

## Exclude connections to 192.168.1.0/24 with port 80
# localnet 192.168.1.0:80/255.255.255.0

## Exclude connections to 192.168.100.0/24
# localnet 192.168.100.0/255.255.255.0

## Exclude connections to ANYwhere with port 80
# localnet 0.0.0.0:80/0.0.0.0
# localnet [::]:80/0

## RFC6890 Loopback address range
## if you enable this, you have to make sure remote_dns_subnet is not 127
## you'll need to enable it if you want to use an application that 
## connects to localhost.
# localnet 127.0.0.0/255.0.0.0
# localnet ::1/128

## RFC1918 Private Address Ranges
# localnet 10.0.0.0/255.0.0.0
# localnet 172.16.0.0/255.240.0.0
# localnet 192.168.0.0/255.255.0.0

### Examples for dnat
## Trying to proxy connections to destinations which are dnatted,
## will result in proxying connections to the new given destinations.
## Whenever I connect to 1.1.1.1 on port 1234 actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1:1234  1.1.1.2:443

## Whenever I connect to 1.1.1.1 on port 443 actually connect to 1.1.1.2 on port 443
## (no need to write :443 again)
# dnat 1.1.1.2:443  1.1.1.2

## No matter what port I connect to on 1.1.1.1 port actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1  1.1.1.2:443

## Always, instead of connecting to 1.1.1.1, connect to 1.1.1.2
# dnat 1.1.1.1  1.1.1.2

# ProxyList format
#       type  ip  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#       only numeric ipv4 addresses are valid
#
#
#        Examples:
#
#            	socks5	192.168.67.78	1080	lamer	secret
#		http	192.168.89.3	8080	justu	hidden
#	 	socks4	192.168.1.49	1080
#	        http	192.168.39.93	8080	
#		
#
#       proxy types: http, socks4, socks5, raw
#         * raw: The traffic is simply forwarded to the proxy without modification.
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

这里面需要我们改的其实只有最后的 `[ProxyList]` 字段（其他字段如有需求请自行了解）。在这个字段上面的注释中给出了该字段下每行的书写格式：

```shell
type  ip  port [user pass]
```

`type` 即协议，proxychains 支持 `http`, `socks4`, `socks5`, `raw`；  
然后是地址 `ip` 和端口 `port`，以及可选的用户名 `user` 和密码 `pass`。



在这里配置好你的本地代理信息就可以了。

> ProxyChains 只是本地代理，所以你还需要一个网络代理工具（比如 V2Ray）。
>
> 如果你刚好在使用某个 V2Ray 客户端（例如 Windows 上的 V2RrayN），那么这篇文章可能对你也有帮助：[Linux 配置 V2Ray 和 ProxyChains 实现命令行代理（无图形界面）](https://gukaifeng.cn/posts/linux-pei-zhi-v2ray-he-proxychains-shi-xian-ming-ling-xing-dai-li-wu-tu-xing-jie-mian/)。

## 3. 测试代理

这里就以国内通常无法访问的谷歌 www.google.com 为例：

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