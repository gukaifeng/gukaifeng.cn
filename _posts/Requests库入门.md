
**Requests** is an elegant and simple HTTP library for Python, built for human beings.

关于Requests库更多信息，点击[这里](http://python-requests.org)查看Requests库官网。

## 1. Requsets库的7个主要方法

| 方法               | 说明                                             |
| ------------------ | ------------------------------------------------ |
| requests.request() | 构造一个请求，支撑以下各方法的基础方法。         |
| requests.get()     | 获取HTML网页的主要方法，对应于HTTP的GET。        |
| requests.head()    | 获取HTML网页头信息的方法，对应于HTTP的HEAD。     |
| requests.post()    | 向HTML网页提交POST请求的方法，对应于HTTP的POST。 |
| requests.put()     | 向HTML网页提交PUT请求的方法，对应于HTTP的PUT。   |
| requests.patch()   | 向HTML网页提交局部修改请求，对应于HTTP的PATCH。  |
| requests.delete()  | 向HTML页面提交删除请求，对应于HTTP的DELETE。     |

<!--more-->

Requests库中其实只有一个方法`request()`，如果查看源代码就会发现，其他的6个方法其实都是调用这个方法实现的，提供给我们这6个方法只是为了我们的编程更加方便。

### 1.1. Requests库的get()方法

* `r = requests.get(url)` 

    构造一个向服务器请求资源的Request对象，返回一个包含服务器资源的Response对象。

* requests.get()的完整写法为`requests.get(url, params=None, **kwargs)``

    `url`: 拟获取页面的url链接；

    `params`: url中的额外参数，字典或字节流格式；

    `**kwargs`: 12个控制访问的参数。

#### 1.1.1. Response对象的属性

Requests库的2个重要对象分别是Request对象和Response对象，其中Response对象包含了爬虫返回的内容。

Response对象的常用属性：

| 属性                | 说明                                             |
| ------------------- | ------------------------------------------------ |
| r.status_code       | HTTP请求的返回状态码。                           |
| r.text              | HTTP相应内容的字符串形式，即url对应的页面内容。  |
| r.encoding          | 从HTTP的header中猜测的相应内容编码格式。         |
| r.apparent_encoding | 从内容分析出的响应内容编码格式（备选编码格式）。 |
| r.content           | HTTP响应内容的二进制形式。                       |

理解`r.encoding`和`r.apparent_encoding`的区别：

* `r.encoding`：如果header中不存在charset，则认为编码是`IOS-8859-1`。

* `r.apparent_encoding`：根据网页内容分析出的编码方式。

`r.apparent_encoding`中的编码方式往往更加准确。

假设我们获取的网站信息编码为`utf-8`，而`r.encoding = 'IOS-8859-1'`，`r.apparent_encoding = 'utf-8'`，那么我们是无法正常查看页面信息的。此时，我们可以分别查看两个编码值，然后让 `r.encoding = 'utf-8' 就可以了。`

#### 1.1.2. 使用get()方法获取网络资源的基本流程

1. 使用Response对象的属性`r.status_code`获取状态码；
2. 如果状态码是200，则可以继续进行`r.text`，`r.encoding`，`r.apparent_encoding`，`r.content`等操作；如果状态码不是200，说明本次请求产生了异常。

### 1.2. Requests库的head()方法

```python
r = requests.head("https://gukaifeng.me")
```

```python
r.headers
```

```shell
{'Content-Type': 'text/html; charset=utf-8', 'Server': 'GitHub.com', 'Last-Modified': 'Thu, 05 Mar 2020 16:32:47 GMT', 'ETag': 'W/"5e6129af-8aa7"', 'Access-Control-Allow-Origin': '*', 'Expires': 'Fri, 06 Mar 2020 05:57:54 GMT', 'Cache-Control': 'max-age=600', 'Content-Encoding': 'gzip', 'X-Proxy-Cache': 'MISS', 'X-GitHub-Request-Id': '6E00:621A:13401B:14C109:5E61E40A', 'Content-Length': '8061', 'Accept-Ranges': 'bytes', 'Date': 'Fri, 06 Mar 2020 05:47:54 GMT', 'Via': '1.1 varnish', 'Age': '0', 'Connection': 'keep-alive', 'X-Served-By': 'cache-hnd18723-HND', 'X-Cache': 'MISS', 'X-Cache-Hits': '0', 'X-Timer': 'S1583473675.656988,VS0,VE174', 'Vary': 'Accept-Encoding', 'X-Fastly-Request-ID': '28b256b71ce0697115ec8824ffbd40d8a15c8800'}
```

```python
r.text
```

```shell
''
```

通过head()方法可以用很少的网络流量获取资源的概要信息。

### 1.3. Requests库的post()方法

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post("https://gukaifeng.me/post", data = payload) # 向URL POST一个字典，自动编码为form（表单）
print(r.text)
```

```shell
{
    ...
    "form": {
        "key2": "value2",
        "key1": "value1"
    },
}
```

```python
r = requests.post("https://gukaifeng.me/post", data = "ABC") # 向URL POST一个字符串，自动编码为data
print(r.text)
```

```shell
{
    ...
    "data": "ABC",
    "form": {},
}
```

### 1.4. Requests库的put()方法

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post("https://gukaifeng.me/post", data = payload) # 向URL POST一个字典，自动编码为form（表单）
print(r.text)
```

```shell
{
    ...
    "form": {
        "key2": "value2",
        "key1": "value1"
    },
}
```

注：put()方法与post()方法类似，只是会将原数据覆盖。



## 2. 爬取网页的通用代码框架

### 2.1. 理解Requests库的异常

| 异常                      | 说明                                          |
| :------------------------ | --------------------------------------------- |
| requests.ConnectionError  | 网络连接错误异常。如DNS查询失败、拒绝连接等。 |
| requests.HTTPError        | HTTP错误异常。                                |
| requests.URLRequired      | URL缺失异常。                                 |
| requests.TooManyRedirects | 超过最大重定向次数，产生重定向异常。          |
| requests.ConnectTimeout   | 连接远程服务器超时异常。                      |
| requests.Timeout          | 请求URL超时，产生超时异常。                   |

`requests.ConnectTimeout`和`requests.Timeout`的区别：

* `requests.ConnectTimeout`：仅指连接服务器的过程超时异常。
* `requests.Timeout`：指的是从发起url到获得内容的整个过程超时异常。

| 异常                 | 说明                                            |
| -------------------- | ----------------------------------------------- |
| r.raise_for_status() | 如果状态码不是200，产生异常requests.HTTPError。 |

### 2.2. 爬取网页的通用代码框架

```python
import requests

def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status() # 如果状态码不是200，引发HTTPErrpr异常
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "Exception"


if __name__ == "__main__":
    url = "https://gukaifeng.me"
    print(getHTMLText(url))
```

## 3. HTTP协议及Requests库方法

### 3.1. HTTP协议

* HTTP，Hypertext Transfer Protocal，超文本传输协议。

* HTTP是一个基于“请求与响应”模式的、无状态的应用层协议。

    无状态：指HTTP第一次请求和第二次请求之间没有关系。

* HTTP协议采用URL作为定位网络资源的标识。

    URL格式：`http://host[:port][path]`

    `host`：合法的Internet主机域名或IP地址。

    `port`：端口号。若缺省，端口号为80。

    `path`：请求资源的路径。

    URL是通过HTTP协议存取资源的Internet路径，一个URL对应一个数据资源。

#### 3.1.1. HTTP协议对资源的操作

| 方法   | 说明                                                  |
| ------ | ----------------------------------------------------- |
| GET    | 请求获取URL位置的资源。                               |
| HEAD   | 请求获取URL位置的响应报告，即获得该资源的头部信息。   |
| POST   | 请求向URL位置的资源后附加新的数据。                   |
| PUT    | 请求向URL位置存储一个资源，覆盖原URL位置的资源。      |
| PATCH  | 请求局部更新URL位置的资源，即改变该处资源的部分内容。 |
| DELETE | 请求删除URL位置存储的资源。                           |

理解`PATCH`和`PUT`的区别：

假设URL位置有一族数据UserInfo，包括UserID、UserName等20个字段。

需求：用户修改了UserName，其他不变。

* 采用PATCH，仅向URL提交UserName的局部更新请求。

* 采用PUT，必须将所有20个字段一并提交到URL，未提交字段将被删除。

    PATCH的最主要好处：节省网络带宽。

#### 3.1.2. HTTP协议与Requests库

| HTTP协议方法 | Request库方法     | 功能一致性 |
| ------------ | ----------------- | ---------- |
| GET          | requests.get()    | 一致       |
| HEAD         | requests.head()   | 一致       |
| POST         | requests.post()   | 一致       |
| PUT          | requests.put()    | 一致       |
| PATCH        | requests.patch()  | 一致       |
| DELETE       | requests.delete() | 一致       |

### 3.2. Request库方法

#### 3.2.1. requests.request()方法

requests.request()方法是所有其他方法的基础方法，该方法提供3个参数：

`requests.request(method, url, **kwargs)`

`method`：请求方式，对应GET/PUT/POST等7种方法（还有个不常用的OPTIONS）。

`url`：拟获取页面的url链接。

`**kwargs`：控制访问的参数，共13个，均为可选项。

`**keargs` 参数解析：

* `params`：字典或字节序列，作为参数增加到url中。

    ```python
    kv = {'key1': 'value1', 'key2': 'value2'}
    r = requests.request("GET", "https://gukaifeng.me/get", params=kv)
    print(r.url)
    ```

    ```shell
    https://gukaifeng.me/get?key1=value1&key2=value2
    ```

* `data`：字典、字节序列或文件对象，作为Request的内容。

    ```python
    kv = {'key1': 'value1', 'key2': 'value2'}
    r = requests.request("POST", "https://gukaifeng.me/post", data=kv)
    ```

    ```python
    body = "Contents"
    r = requests.request("POST", "https://gukaifeng.me/post", data=body)
    ```

* `json`：JSON格式的数据，作为Reuqest的内容。

    ```python
    kv = {'key1': 'value1'}
    r = requests.request("POST", "https://gukaifeng.me/post", json=kv)
    ```

* `headers`：字典，HTTP定制头。用于定制请求header的内容，也可以”伪装“。

    ```python
    hd = {'user-agent': 'Chrome/10'}
    r = requests.request("POST", "https://gukaifeng.me/post", headers=hd)
    ```

* `cookies`：字典或CookieJar，Request中的cookie。

* `auth`：元组，支持HTTP认证功能。

* `files`：字典类型，传输文件。

    ```python
    fs = {'file': open('data.xls', 'rb')}
    r = requests.request("POST", "https://gukaifeng.me/post", files=fs)
    ```

* `timeout`：设定超时时间，单位为秒。

    ```python
    r = requests.request("POST", "https://gukaifeng.me/post", timeout=10)
    ```

* `proxies`：字典类型，设定访问代理服务器，可以增加登录认证，可以隐藏爬虫用户的原IP信息，防止反爬虫逆追踪。

    ```python
    pxs = {'http': 'http://user:pass@10.10.10.1.1234', # 设置http代理
           'https': 'https://10.10.10.1:4321'} # 设置https代理
    r = requests.request("POST", "https://gukaifeng.me/post", proxies=pxs)
    ```

* `allow_redirects`：True/False，默认为True。重定向开关。

* `stream`：True/False，默认为True。获取内容立即下载开关。

* `verify`：True/False，默认为True，认证SSL证书开关。

* `cert`：本地SSL证书路径。

#### 3.2.2. requests.get()方法

`requests.get(url, params=None, **kwargs)`

`url`：拟获取页面的url链接。

`params`：url中的额外参数，字典或字节流格式，可选。

`**kwargs`：控制访问的参数，共12个，均为可选项。

注：这里的`params`和`**kwargs`就是`requests.request()`方法中的`**kwargs`，把`params`字段单独拿了出来，所以不再重复介绍。

#### 3.3.3. requests.head()方法

`requests.head(url, **kwargs)`

`url`：拟获取页面的url链接。

`**kwargs`：控制访问的参数，共13个，均为可选项。与 `requests.request()`方法中的`**kwargs`相同。

#### 3.3.4. requests.post()方法

`request.post(url, data=None, json=None, **kwargs)`

`**kwargs`：控制访问的参数，共11个，均为可选项。

与`requests.get()`类似，这里的`data`和`json`参数是对`requests.request()`参数`**kwargs`的拆分。

#### 3.3.5. request.put()方法

`requests.put(url, data=None, **kwargs)`

`**kwargs`：12个，与上面同理。

#### 3.3.6. request.patch()方法

`requests.patch(url, data=None, **kwargs)`

`**kwargs`：12个，与上面同理。

#### 3.3.7. request.delete()方法

`requests.delete(url, **kwargs)`

`**kwargs`：13个，与上面同理。



## 4. Requests库入门小结

受网络安全限制的影响，我们往往很难向url发起POST/PUT/PATCH/DELETE请求。

因此，作为爬虫用户，真正最常使用的是`requests.get()`方法。对于某些资源庞大的url，我们也可使用`request.head()`方法获得资源概要。故我们应重点掌握`requests.get()`方法和`request.head()`方法。

另外，网络连接有风险，异常处理很重要。

2.2中爬取网页的通用代码框架很重要，尤其是其中的`try/except`语句，我们一定要捕获异常。