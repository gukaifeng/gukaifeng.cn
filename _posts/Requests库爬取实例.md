---
title: Requests库爬取实例
mathjax: false
date: 2020-03-05 22:46:45
updated: 2020-03-05 22:46:45
tags: [Python,Requests,爬虫]
categories: [爬虫]
toc: true
---

## 1. 京东商品页面的爬取

找到一个京东商品页面，我这里选的是[**Apple 2019新品 MacBook Pro 16【带触控栏】九代八核i9 16G 1TB 深空灰 Radeon Pro 5500M显卡 **](https://item.jd.com/100010079900.html)，链接是**https://item.jd.com/100010079900.html**。链接可能会失效，不过其他商品页面链接完全同理。

本例仅直接使用最后的完整代码，我们应该在此之前，先对我们爬取的页面做一些前置工作。

前置工作参照后面几个例子，一般是先在交互式页面做前置工作，确认没有问题后，再使用完整代码爬取。

```python
import requests


def getHTMLText(url):
    try:
        r = requests.get(url)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text[:1000] # 页面中只有前面小部分是商品信息，故只取前面一小部分
    except:
        return "Exception"


if __name__ == "__main__":
    url = "https://item.jd.com/100010079900.html"
    print(getHTMLText(url))
```

<!--more-->

```
<!DOCTYPE HTML>
<html lang="zh-CN">
<head>
    <!-- shouji -->
    <meta http-equiv="Content-Type" content="text/html; charset=gbk" />
    <title>【AppleMacBook Pro 16】Apple 2019新品 MacBook Pro 16【带触控栏】九代八核i9 16G 1TB 深空灰 Radeon Pro 5500M显卡 笔记本电脑 轻薄本 MVVK2CH/A【行情 报价 价格 评测】-京东</title>
    <meta name="keywords" content="AppleMacBook Pro 16,AppleMacBook Pro 16,AppleMacBook Pro 16报价,AppleMacBook Pro 16报价"/>
    <meta name="description" content="【AppleMacBook Pro 16】京东JD.COM提供AppleMacBook Pro 16正品行货，并包括AppleMacBook Pro 16网购指南，以及AppleMacBook Pro 16图片、MacBook Pro 16参数、MacBook Pro 16评论、MacBook Pro 16心得、MacBook Pro 16技巧等信息，网购AppleMacBook Pro 16上京东,放心又轻松" />
    <meta name="format-detection" content="telephone=no">
    <meta http-equiv="mobile-agent" content="format=xhtml; url=//item.m.jd.com/product/100010079900.html">
    <meta http-equiv="mobile-agent" content="format=html5; url=//item.m.jd.com/product/100010079900.html">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    <link rel="c
```

## 2. 亚马逊商品页面的爬取

与前一个实例类似，亚马逊商品页面的爬取方法与京东商品页面的爬取一样。

我们可以使用与上面一样的代码，仅修改url链接。

我这里选用的是https://www.amazon.cn/dp/B07746N2J9。

这里先说一个问题，在编写完整的爬虫代码前，我们应该先检查对页面的访问情况。

```shell
>>> import requests
>>> url = "https://www.amazon.cn/dp/B07746N2J9"
>>> r = requests.get(url)
>>> r.status_code
503
>>> r.encoding
'ISO-8859-1'
>>> r.encoding = r.apparent_encoding
>>> r.text
'<!DOCTYPE html>\n<!--[if lt IE 7]> <html lang="zh-CN" class="a-no-js a-lt-ie9 a-lt-ie8 a-lt-ie7"> <![endif]-->\n<!--[if IE 7]>    <html lang="zh-CN" class="a-no-js a-lt-ie9 a-lt-ie8"> <![endif]-->\n<!--[if IE 8]>    <html lang="zh-CN" class="a-no-js a-lt-ie9"> <![endif]-->\n<!--[if gt IE 8]><!-->\n<html class="a-no-js" lang="zh-CN"><!--<![endif]--><head>\n<meta http-equiv="content-type" content="text/html; charset=UTF-8">\n<meta charset="utf-8">\n<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">\n<title dir="ltr">Amazon CAPTCHA</title>\n<meta name="viewport" content="width=device-width">\n<link rel="stylesheet" href="https://images-na.ssl-images-amazon.com/images/G/01/AUIClients/AmazonUI-3c913031596ca78a3768f4e934b1cc02ce238101.secure.min._V1_.css">\n<script>\n\nif (true === true) {\n    var ue_t0 = (+ new Date()),\n        ue_csm = window,\n        ue = { t0: ue_t0, d: function() { return (+new Date() - ue_t0); } },\n        ue_furl = "fls-cn.amazon.cn",\n        ue_mid = "AAHKV2X7AFYLW",\n        ue_sid = (document.cookie.match(/session-id=([0-9-]+)/) || [])[1],\n        ue_sn = "opfcaptcha.amazon.cn",\n        ue_id = \'AAGRX9PGTXKK9NJ47MDA\';\n}\n</script>\n</head>\n<body>\n\n<!--\n        To discuss automated access to Amazon data please contact api-services-support@amazon.com.\n        For information about migrating to our APIs refer to our Marketplace APIs at https://developer.amazonservices.com.cn/index.html/ref=rm_c_sv, or our Product Advertising API at https://associates.amazon.cn/gp/advertising/api/detail/main.html/ref=rm_c_ac for advertising use cases.\n-->\n\n<!--\nCorreios.DoNotSend\n-->\n\n<div class="a-container a-padding-double-large" style="min-width:350px;padding:44px 0 !important">\n\n    <div class="a-row a-spacing-double-large" style="width: 350px; margin: 0 auto">\n\n        <div class="a-row a-spacing-medium a-text-center"><i class="a-icon a-logo"></i></div>\n\n        <div class="a-box a-alert a-alert-info a-spacing-base">\n            <div class="a-box-inner">\n                <i class="a-icon a-icon-alert"></i>\n                <h4>请输入您在下方看到的字符</h4>\n                <p class="a-last">抱歉，我们只是想确认一下当前访问者并非自动程序。为了达到最佳效果，请确保您浏览器上的 Cookie 已启用。</p>\n                </div>\n            </div>\n\n            <div class="a-section">\n\n                <div class="a-box a-color-offset-background">\n                    <div class="a-box-inner a-padding-extra-large">\n\n                        <form method="get" action="/errors/validateCaptcha" name="">\n                            <input type=hidden name="amzn" value="qbzrA1PB1x9LlmVVTpPjDg==" /><input type=hidden name="amzn-r" value="&#047;dp&#047;B07746N2J9" />\n                            <div class="a-row a-spacing-large">\n                                <div class="a-box">\n                                    <div class="a-box-inner">\n                                        <h4>请输入您在这个图片中看到的字符：</h4>\n                                        <div class="a-row a-text-center">\n                                            <img src="https://images-na.ssl-images-amazon.com/captcha/sargzmyv/Captcha_idpwqxlotw.jpg">\n                                        </div>\n                                        <div class="a-row a-spacing-base">\n                                            <div class="a-row">\n                                                <div class="a-column a-span6">\n                                                    <label for="captchacharacters">输入字符</label>\n                                                </div>\n                                                <div class="a-column a-span6 a-span-last a-text-right">\n                                                    <a onclick="window.location.reload()">换一张图</a>\n                                                </div>\n                                            </div>\n                                            <input autocomplete="off" spellcheck="false" id="captchacharacters" name="field-keywords" class="a-span12" autocapitalize="off" autocorrect="off" type="text">\n                                        </div>\n                                    </div>\n                                </div>\n                            </div>\n\n                            <div class="a-section a-spacing-extra-large">\n\n                                <div class="a-row">\n                                    <span class="a-button a-button-primary a-span12">\n                                        <span class="a-button-inner">\n                                            <button type="submit" class="a-button-text">继续购物</button>\n                                        </span>\n                                    </span>\n                                </div>\n\n                            </div>\n                        </form>\n\n                    </div>\n                </div>\n\n            </div>\n\n        </div>\n\n        <div class="a-divider a-divider-section"><div class="a-divider-inner"></div></div>\n\n        <div class="a-text-center a-spacing-small a-size-mini">\n            <a href="https://www.amazon.cn/gp/help/customer/display.html/ref=footer_claim?ie=UTF8&nodeId=200347160">使用条件</a>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <a href="https://www.amazon.cn/gp/help/customer/display.html/ref=footer_privacy?ie=UTF8&nodeId=200347130">隐私声明</a>\n        </div>\n\n        <div class="a-text-center a-size-mini a-color-secondary">\n          &copy; 1996-2015, Amazon.com, Inc. or its affiliates\n          <script>\n           if (true === true) {\n             document.write(\'<img src="https://fls-cn.amaz\'+\'on.cn/\'+\'1/oc-csi/1/OP/requestId=AAGRX9PGTXKK9NJ47MDA&js=1" />\');\n           };\n          </script>\n          <noscript>\n            <img src="https://fls-cn.amazon.cn/1/oc-csi/1/OP/requestId=AAGRX9PGTXKK9NJ47MDA&js=0" />\n          </noscript>\n        </div>\n    </div>\n    <script>\n    if (true === true) {\n        var elem = document.createElement("script");\n        elem.src = "https://images-cn.ssl-images-amazon.com/images/G/01/csminstrumentation/csm-captcha-instrumentation.min._V" + (+ new Date()) + "_.js";\n        document.getElementsByTagName(\'head\')[0].appendChild(elem);\n    }\n    </script>\n</body></html>\n'
```

爬取出现了错误，但我们能收到来自服务器的信息，说明这不是网络错误。

这个错误是因为，亚马逊对爬虫通过查看HTTP头进行了限制，我们看代码下面：

```shell
>>> r.request.headers
{'User-Agent': 'python-requests/2.22.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive'}
```

可以看到`'User-Agent': 'python-requests/2.22.0'`，我们的程序诚实的告诉了亚马逊服务器，这次请求是由一个Python的Requests库产生的，而亚马逊不支持这样的访问。

下面，参照[Requests库入门]([https://gukaifeng.me/2020/03/05/Requests%E5%BA%93%E5%85%A5%E9%97%A8/#2-%E7%88%AC%E5%8F%96%E7%BD%91%E9%A1%B5%E7%9A%84%E9%80%9A%E7%94%A8%E4%BB%A3%E7%A0%81%E6%A1%86%E6%9E%B6](https://gukaifeng.me/2020/03/05/Requests库入门/#2-爬取网页的通用代码框架)中的方法，我们伪装一下。

```shell
>>> import requests
>>> url = "https://www.amazon.cn/dp/B07746N2J9"
>>> hd = {'user-agent': 'Mozilla/5.0'}
>>> r = requests.get(url, headers=hd)
>>> r.status_code
200
>>> r.encoding
'ISO-8859-1'
>>> r.encoding = r.apparent_encoding
>>> r.text
'<!DOCTYPE html>\n<!--[if lt IE 7]> <html lang="zh-CN" class="a-no-js a-lt-ie9 a-lt-ie8 a-lt-ie7"> <![endif]-->\n<!--[if IE 7]>    <html lang="zh-CN" class="a-no-js a-lt-ie9 a-lt-ie8"> <![endif]-->\n<!--[if IE 8]>    <html lang="zh-CN" class="a-no-js a-lt-ie9"> <![endif]-->\n<!--[if gt IE 8]><!-->\n<html class="a-no-js" lang="zh-CN"><!--<![endif]--><head>\n<meta http-equiv="content-type" content="text/html; charset=UTF-8">\n<meta charset="utf-8">\n<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">\n<title dir="ltr">Amazon CAPTCHA</title>\n<meta name="viewport" content="width=device-width">\n<link rel="stylesheet" href="https://images-na.ssl-images-amazon.com/images/G/01/AUIClients/AmazonUI-3c913031596ca78a3768f4e934b1cc02ce238101.secure.min._V1_.css">\n<script>\n\nif (true === true) {\n    var ue_t0 = (+ new Date()),\n        ue_csm = window,\n        ue = { t0: ue_t0, d: function() { return (+new Date() - ue_t0); } },\n        ue_furl = "fls-cn.amazon.cn",\n        ue_mid = "AAHKV2X7AFYLW",\n        ue_sid = (document.cookie.match(/session-id=([0-9-]+)/) || [])[1],\n        ue_sn = "opfcaptcha.amazon.cn",\n        ue_id = \'HDMFDWJW60YRSZF22WXB\';\n}\n</script>\n</head>\n<body>\n\n<!--\n        To discuss automated access to Amazon data please contact api-services-support@amazon.com.\n        For information about migrating to our APIs refer to our Marketplace APIs at https://developer.amazonservices.com.cn/index.html/ref=rm_c_sv, or our Product Advertising API at https://associates.amazon.cn/gp/advertising/api/detail/main.html/ref=rm_c_ac for advertising use cases.\n-->\n\n<!--\nCorreios.DoNotSend\n-->\n\n<div class="a-container a-padding-double-large" style="min-width:350px;padding:44px 0 !important">\n\n    <div class="a-row a-spacing-double-large" style="width: 350px; margin: 0 auto">\n\n        <div class="a-row a-spacing-medium a-text-center"><i class="a-icon a-logo"></i></div>\n\n        <div class="a-box a-alert a-alert-info a-spacing-base">\n            <div class="a-box-inner">\n                <i class="a-icon a-icon-alert"></i>\n                <h4>请输入您在下方看到的字符</h4>\n                <p class="a-last">抱歉，我们只是想确认一下当前访问者并非自动程序。为了达到最佳效果，请确保您浏览器上的 Cookie 已启用。</p>\n                </div>\n            </div>\n\n            <div class="a-section">\n\n                <div class="a-box a-color-offset-background">\n                    <div class="a-box-inner a-padding-extra-large">\n\n                        <form method="get" action="/errors/validateCaptcha" name="">\n                            <input type=hidden name="amzn" value="H/hCaolI3QagyDNK6cGgcA==" /><input type=hidden name="amzn-r" value="&#047;dp&#047;B07746N2J9" />\n                            <div class="a-row a-spacing-large">\n                                <div class="a-box">\n                                    <div class="a-box-inner">\n                                        <h4>请输入您在这个图片中看到的字符：</h4>\n                                        <div class="a-row a-text-center">\n                                            <img src="https://images-na.ssl-images-amazon.com/captcha/sargzmyv/Captcha_kzkgvarkbt.jpg">\n                                        </div>\n                                        <div class="a-row a-spacing-base">\n                                            <div class="a-row">\n                                                <div class="a-column a-span6">\n                                                    <label for="captchacharacters">输入字符</label>\n                                                </div>\n                                                <div class="a-column a-span6 a-span-last a-text-right">\n                                                    <a onclick="window.location.reload()">换一张图</a>\n                                                </div>\n                                            </div>\n                                            <input autocomplete="off" spellcheck="false" id="captchacharacters" name="field-keywords" class="a-span12" autocapitalize="off" autocorrect="off" type="text">\n                                        </div>\n                                    </div>\n                                </div>\n                            </div>\n\n                            <div class="a-section a-spacing-extra-large">\n\n                                <div class="a-row">\n                                    <span class="a-button a-button-primary a-span12">\n                                        <span class="a-button-inner">\n                                            <button type="submit" class="a-button-text">继续购物</button>\n                                        </span>\n                                    </span>\n                                </div>\n\n                            </div>\n                        </form>\n\n                    </div>\n                </div>\n\n            </div>\n\n        </div>\n\n        <div class="a-divider a-divider-section"><div class="a-divider-inner"></div></div>\n\n        <div class="a-text-center a-spacing-small a-size-mini">\n            <a href="https://www.amazon.cn/gp/help/customer/display.html/ref=footer_claim?ie=UTF8&nodeId=200347160">使用条件</a>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <span class="a-letter-space"></span>\n            <a href="https://www.amazon.cn/gp/help/customer/display.html/ref=footer_privacy?ie=UTF8&nodeId=200347130">隐私声明</a>\n        </div>\n\n        <div class="a-text-center a-size-mini a-color-secondary">\n          &copy; 1996-2015, Amazon.com, Inc. or its affiliates\n          <script>\n           if (true === true) {\n             document.write(\'<img src="https://fls-cn.amaz\'+\'on.cn/\'+\'1/oc-csi/1/OP/requestId=HDMFDWJW60YRSZF22WXB&js=1" />\');\n           };\n          </script>\n          <noscript>\n            <img src="https://fls-cn.amazon.cn/1/oc-csi/1/OP/requestId=HDMFDWJW60YRSZF22WXB&js=0" />\n          </noscript>\n        </div>\n    </div>\n    <script>\n    if (true === true) {\n        var elem = document.createElement("script");\n        elem.src = "https://images-cn.ssl-images-amazon.com/images/G/01/csminstrumentation/csm-captcha-instrumentation.min._V" + (+ new Date()) + "_.js";\n        document.getElementsByTagName(\'head\')[0].appendChild(elem);\n    }\n    </script>\n</body></html>\n'
```

注：`'user-agent': 'Mozilla/5.0'`说明这个时候的访问者可能是一个浏览器，这个浏览器可能是火狐，可能是Mozilla，甚至可能是IE10等等。`Mozilla/5.0`是一个很标准的浏览器身份标识字段。

然而，虽然状态码是200，但我们还是没有拿到想要的数据。

应该是亚马逊做了反爬虫，这次爬取宣告失败。

如果成功，那么完整代码和上一个例子，爬取京东商品页面一致，只需加一个修改headers的代码。



## 3. 百度360搜索关键词提交

我们如何实现用爬虫，在搜索引擎中输入关键字，并获取搜索结果呢？

搜索引擎有提供关键词提交接口。

百度的关键词接口：https://www.baidu.com/s?wd=keyword

360的关键词接口：http://www.so.com/s?q=keyword

故我们的爬虫只需要构造这样的url就可以实现向搜索引擎提交关键词、获取搜索结果信息。

参照[Requests库入门]([https://gukaifeng.me/2020/03/05/Requests%E5%BA%93%E5%85%A5%E9%97%A8/#2-%E7%88%AC%E5%8F%96%E7%BD%91%E9%A1%B5%E7%9A%84%E9%80%9A%E7%94%A8%E4%BB%A3%E7%A0%81%E6%A1%86%E6%9E%B6](https://gukaifeng.me/2020/03/05/Requests库入门/#2-爬取网页的通用代码框架)，了解如何在url后面添加参数。

### 3.1. 百度搜索关键词提交

我们先在交互式页面做前置工作。

```shell
>>> import requests
>>> url = "http://www.baidu.com/s"
>>> kv = {'wd': 'Python'}
>>> r = requests.get(url, params=kv)
>>> r.status_code
200
>>> r.request.url
'https://wappass.baidu.com/static/captcha/tuxing.html?&ak=c27bbc89afca0463650ac9bde68ebe06&backurl=https%3A%2F%2Fwww.baidu.com%2Fs%3Fwd%3DPython&logid=10659384054224124723&signature=23fe0bec15acc6971f6cefb77986ad8a&timestamp=1583503124'
>>> r.encoding = r.apparent_encoding
>>> r.text
'<!DOCTYPE html>\n<html lang="zh-CN">\n<head>\n    <meta charset="utf-8">\n    <title>百度安全验证</title>\n    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">\n    <meta name="apple-mobile-web-app-capable" content="yes">\n    <meta name="apple-mobile-web-app-status-bar-style" content="black">\n    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0">\n    <meta name="format-detection" content="telephone=no, email=no">\n    <link rel="shortcut icon" href="https://www.baidu.com/favicon.ico" type="image/x-icon">\n    <link rel="icon" sizes="any" mask href="https://www.baidu.com/img/baidu.svg">\n    <meta http-equiv="X-UA-Compatible" content="IE=Edge">\n    <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">\n    <link rel="stylesheet" href="https://wappass.bdimg.com/static/touch/css/api/mkdjump_8befa48.css" />\n</head>\n<body>\n    <div class="timeout hide">\n        <div class="timeout-img"></div>\n        <div class="timeout-title">网络不给力，请稍后重试</div>\n        <button type="button" class="timeout-button">返回首页</button>\n    </div>\n    <div class="timeout-feedback hide">\n        <div class="timeout-feedback-icon"></div>\n        <p class="timeout-feedback-title">问题反馈</p>\n    </div>\n\n<script src="https://wappass.baidu.com/static/machine/js/api/mkd.js"></script>\n<script src="https://wappass.bdimg.com/static/touch/js/mkdjump_2e06726.js"></script>\n</body>\n</html>'
```

我们发现，虽然状态码是200，但是url进行了跳转。百度也进行了反爬虫，要求验证。

我修改`headers`中的`user-agent`字段后，还是没有成功，对百度搜索的爬虫失败。

### 3.2. 360搜索关键词提交

同样，我们先在交互式页面做前置工作。

```shell
>>> import requests
>>> url = "http://www.so.com/s"
>>> kv = {'q': 'Python'}
>>> r = requests.get(url, params=kv)
>>> r.request.url
'https://www.so.com/s?q=Python'
>>> r.status_code
200
>>> r.encoding
'utf-8'
>>> len(r.text)
356960
```

我们可以看到，爬取应该是成功了，我上面的代码没有给出`r.text`的结果，而是给出的长度，太长了，所以没有把结果放在这里面，读者可以自己写代码查看下。我们现在只要知道爬取成功了就可以了。

对360搜索的爬虫成功，360搜索爬虫全代码在下面给出。

```python
import requests


def getHTMLText(url, keyword):
    try:
        kv = {'p': keyword}
        r = requests.get(url, params=kv)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "Exception"


if __name__ == "__main__":
    url = "https://www.so.com/s"
    keyword = "Python"
    print(getHTMLText(url, keyword))
```

在几十万长度的结果中，我们如何获取我们想要的信息呢？参照[未完待续](/)。

## 4. 网络图片的爬取和存储

网络图片链接的格式：http://www.example.com/picture.jpg

以国家地理网站 http://www.ngchina.com.cn/ 为例，我们随便找一张图，如下。

![http://image.ngchina.com.cn/2019/0523/20190523103156143.jpg](http://image.ngchina.com.cn/2019/0523/20190523103156143.jpg)

这张图片的地址为 http://image.ngchina.com.cn/2019/0523/20190523103156143.jpg

老规矩，我们先在交互式环境下尝试下载图片。

```shell
>>> import requests
>>> path = "/Users/gukaifeng/Downloads/1.jpg"
>>> url = "http://image.ngchina.com.cn/2019/0523/20190523103156143.jpg"
>>> r = requests.get(url)
>>> r.status_code
200
>>> with open(path, 'wb') as f:
...     f.write(r.content)
... 
1750635
```

打开我们的下载目录，可以看到图片已经下载好了，下面对上述代码做一点点解释。

图片是以二进制形式存储的，我们在[Requests库入门](https://gukaifeng.me/2020/03/05/Requests%E5%BA%93%E5%85%A5%E9%97%A8/#1-1-1-Response%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%B1%9E%E6%80%A7)有讲过，`r.content`得到的就是页面信息的二进制形式。

爬虫爬取并存储图片的完整代码如下：

```python
import os
import requests


'''
url: 图片链接地址
dir: 存储图片的本地目录
'''
def downloadPicture(url, dir):
    path = dir + url.split('/')[-1]
    if not os.path.exists(dir):
        os.mkdir(dir)
    if not os.path.exists(path):
        try:
            r = requests.get(url)
            r.raise_for_status()
            with open(path, 'wb') as f:
                f.write(r.content)
                f.close()
                print("OK")
        except:
            print("Exception.")
    else:
        print("File already exists.")



if __name__ == "__main__":
    url = "http://image.ngchina.com.cn/2019/0523/20190523103156143.jpg"
    dir = "/Users/gukaifeng/Downloads/"
    downloadPicture(url, dir)
```

在交互式页面中，我们仅仅尝试了单一的存储图片是否可行。

而在完整代码里，我们添加了一些使得程序具有一定健壮性的代码，健壮性是非常重要的。

在网络中，类似图片这样存储方式的资源有很多种类，比如视频，Flash，动画等。

我们可以修改上述代码，获取网上上各种不同的资源。



## 5. IP地址归属地的自动查询

查询IP地址归属地，手机号码归属地等，我们需要有一个库，然后从库中查询。

我们本身的程序里是没有这样的库的，但是网络上有相关的网站提供这样的功能。

我们使用[ip.cn](http://ip.cn)网站进行查询。查看页面发现，待查询的IP，拼接在该地址后面，例如 https://www.ip.cn/?ip=185.199.109.153 。我们可以想到，类似百度360搜索关键词提交，做一个这样的url，就可以实现IP地址查询了。

同样的，我们要先在交互式页面测试一下我们的思路，因为已经写过多次，这里不再重复。

这里直接给出IP地址归属地自动查询的完整代码。

```python
import requests


def IPBelongingTo(ip):
    kv = {'ip': ip}
    url = "http://ip.cn/"
    try:
        r = requests.get(url, params=kv)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        print(r.text)
    except:
        print("Exception.")


if __name__ == "__main__":
    IPBelongingTo("185.199.109.153")
```

本例相对来说十分简单，但是我在实践中发现，各个IP地址查询网站，大多对爬虫做了限制。

例如[IP138](http://www.ip138.com/)网站限制爬虫（我这里的体现是无限超时），额外提供收费API用于程序查询IP地址。

上述代码中使用[ip.cn](https://www.ip.cn/)页面进行的查询，我报了403错误。403状态码意味着服务器收到了我们的请求但拒绝为我们服务，这同样也是因为此网站对爬虫进行了限制。

本例也是只作学习之用，学会这些方法才是最重要的。