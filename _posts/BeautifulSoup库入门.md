---
title: BeautifulSoup库入门
mathjax: false
date: 2020-03-07 00:33:44
updated: 2020-03-07 00:33:44
tags: [BeautifulSoup,爬虫]
categories: [爬虫]
toc: true
---

**[Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/)** is a Python library for pulling data out of HTML and XML files. It works with your favorite parser to provide idiomatic ways of navigating, searching, and modifying the parse tree. It commonly saves programmers hours or days of work.



## 1. BeautifulSoup库的安装

终端安装方法方法：`pip install beautifulsoup4`。<br/>我这里其实还是用了Anaconda软件，来帮助我管理各种Python包。

下面我们对BeautifulSoup库的安装做一个简单的小测。

<!--more-->

```python
>>> import requests
>>> from bs4 import BeautifulSoup
>>> r = requests.get("http://www.crummy.com/software/BeautifulSoup/")
>>> demo = r.text
>>> soup = BeautifulSoup(demo, "html.parser")
>>> print(soup.prettify)
<bound method Tag.prettify of <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN"
"http://www.w3.org/TR/REC-html40/transitional.dtd">

<html>
<head>
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
<title>Beautiful Soup: We called him Tortoise because he taught us.</title>
<link href="mailto:leonardr@segfault.org" rev="made"/>
<link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>
<meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>
<meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>
<meta content="Leonard Richardson" name="author"/>
</head>
<body alink="red" bgcolor="white" link="blue" text="black" vlink="660066">
<style>
#tidelift { }

#tidelift a {
 border: 1px solid #666666;
 margin-left: auto;
 padding: 10px;
 text-decoration: none;
}

#tidelift .cta {
 background: url("tidelift.svg") no-repeat;
 padding-left: 30px;
}
</style>
<img align="right" src="10.1.jpg" width="250"/><br/>
<p>[ <a href="#Download">Download</a> | <a href="bs4/doc/">Documentation</a> | <a href="#HallOfFame">Hall of Fame</a> | <a href="enterprise.html">For enterprise</a> | <a href="https://code.launchpad.net/beautifulsoup">Source</a> | <a href="https://bazaar.launchpad.net/%7Eleonardr/beautifulsoup/bs4/view/head:/CHANGELOG">Changelog</a> | <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">Discussion group</a>  | <a href="zine/">Zine</a> ]</p>
<div align="center">
<a href="bs4/download/"><h1>Beautiful Soup</h1></a>
</div>
<p>You didn't write that awful page. You're just trying to get some
data out of it. Beautiful Soup is here to help. Since 2004, it's been
saving programmers hours or days of work on quick-turnaround
screen scraping projects.</p>
<p>Beautiful Soup is a Python library designed for quick turnaround
projects like screen-scraping. Three features make it powerful:

<ol>
<li>Beautiful Soup provides a few simple methods and Pythonic idioms
for navigating, searching, and modifying a parse tree: a toolkit for
dissecting a document and extracting what you need. It doesn't take
much code to write an application

<li>Beautiful Soup automatically converts incoming documents to
Unicode and outgoing documents to UTF-8. You don't have to think
about encodings, unless the document doesn't specify an encoding and
Beautiful Soup can't detect one. Then you just have to specify the
original encoding.

<li>Beautiful Soup sits on top of popular Python parsers like <a href="http://lxml.de/">lxml</a> and <a href="http://code.google.com/p/html5lib/">html5lib</a>, allowing you
to try out different parsing strategies or trade speed for
flexibility.

</li></li></li></ol>
<p>Beautiful Soup parses anything you give it, and does the tree
traversal stuff for you. You can tell it "Find all the links", or
"Find all the links of class <tt>externalLink</tt>", or "Find all the
links whose urls match "foo.com", or "Find the table heading that's
got bold text, then give me that text."

<p>Valuable data that was once locked up in poorly-designed websites
is now within your reach. Projects that would have taken hours take
only minutes with Beautiful Soup.

<p>Interested? <a href="bs4/doc/">Read more.</a>
<h3>Getting and giving support</h3>
<div align="center" id="tidelift">
<a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=enterprise" target="_blank">
<span class="cta">
  Beautiful Soup for enterprise available via Tidelift
 </span>
</a>
</div>
<p>If you have questions, send them to <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">the discussion
group</a>. If you find a bug, <a href="https://bugs.launchpad.net/beautifulsoup/">file it on Launchpad</a>. If it's a security vulnerability, report it confidentially through <a href="https://tidelift.com/security">Tidelift</a>.</p>
<p>If you use Beautiful Soup as part of your work, please consider a <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=website">Tidelift subscription</a>. This will support many of the free software projects your organization depends on, not just Beautiful Soup.


<p>If Beautiful Soup is useful to you on a personal level, you might like to read <a href="zine/"><i>Tool Safety</i></a>, a short zine I wrote about what I learned about software development from working on Beautiful Soup. Thanks!</p>
</p></p></p></p></p></body></html>
<a name="Download"><h2>Download Beautiful Soup</h2></a>
<p>The current release is <a href="bs4/download/">Beautiful Soup
4.8.2</a> (December 24, 2019). You can install Beautiful Soup 4 with
<code>pip install beautifulsoup4</code>.

<p>In Debian and Ubuntu, Beautiful Soup is available as the
<code>python-bs4</code> package (for Python 2) or the
<code>python3-bs4</code> package (for Python 3). In Fedora it's
available as the <code>python-beautifulsoup4</code> package.

<p>Beautiful Soup is licensed under the MIT license, so you can also
download the tarball, drop the <code>bs4/</code> directory into almost
any Python application (or into your library path) and start using it
immediately. (If you want to do this under Python 3, you will need to
manually convert the code using <code>2to3</code>.)

<p>Beautiful Soup 4 works on both Python 2 (2.7+) and Python
3. Support for Python 2 will be discontinued on or after December 31,
2020—one year after the Python 2 sunsetting date.

<h3>Beautiful Soup 3</h3>
<p>Beautiful Soup 3 was the official release line of Beautiful Soup
from May 2006 to March 2012. It does not support Python 3 and it will
be discontinued on or after December 31, 2020—one year after the
Python 2 sunsetting date. If you have any active projects using
Beautiful Soup 3, you should migrate to Beautiful Soup 4 as part of
your Python 3 conversion.

<p><a href="http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html">Here's
the Beautiful Soup 3 documentation.</a>
<p>The current and hopefully final release of Beautiful Soup 3 is <a href="download/3.x/BeautifulSoup-3.2.2.tar.gz">3.2.2</a> (October 5,
2019). It's the <code>BeautifulSoup</code> package on pip. It's also
available as <code>python-beautifulsoup</code> in Debian and Ubuntu,
and as <code>python-BeautifulSoup</code> in Fedora.

<p>Once Beautiful Soup 3 is discontinued, these package names will be available for use by a more recent version of Beautiful Soup.

<p>Beautiful Soup 3, like Beautiful Soup 4, is <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup?utm_source=pypi-beautifulsoup&amp;utm_medium=referral&amp;utm_campaign=website">supported through Tidelift</a>.</p>
<a name="HallOfFame"><h2>Hall of Fame</h2></a>
<p>Over the years, Beautiful Soup has been used in hundreds of
different projects. There's no way I can list them all, but I want to
highlight a few high-profile projects. Beautiful Soup isn't what makes
these projects interesting, but it did make their completion easier:

<ul>
<li><a href="http://www.nytimes.com/2007/10/25/arts/design/25vide.html">"Movable
 Type"</a>, a work of digital art on display in the lobby of the New
 York Times building, uses Beautiful Soup to scrape news feeds.

<li>Reddit uses Beautiful Soup to <a href="https://github.com/reddit/reddit/blob/85f9cff3e2ab9bb8f19b96acd8da4ebacc079f04/r2/r2/lib/media.py">parse
a page that's been linked to and find a representative image</a>.

<li>Alexander Harrowell uses Beautiful Soup to <a href="http://www.harrowell.org.uk/viktormap.html">track the business
 activities</a> of an arms merchant.

<li>The developers of Python itself used Beautiful Soup to <a href="http://svn.python.org/view/tracker/importer/">migrate the Python
bug tracker from Sourceforge to Roundup</a>.

<li>The <a href="http://www2.ljworld.com/">Lawrence Journal-World</a>
uses Beautiful Soup to <a href="http://www.b-list.org/weblog/2010/nov/02/news-done-broke/">gather
statewide election results</a>.

<li>The <a href="http://esrl.noaa.gov/gsd/fab/">NOAA's Forecast
Applications Branch</a> uses Beautiful Soup in <a href="http://laps.noaa.gov/topograbber/">TopoGrabber</a>, a script for
downloading "high resolution USGS datasets."

</li></li></li></li></li></li></ul>
<p>If you've used Beautiful Soup in a project you'd like me to know
about, please do send email to me or <a href="http://groups.google.com/group/beautifulsoup/">the discussion
group</a>.

<h2>Development</h2>
<p>Development happens at <a href="https://launchpad.net/beautifulsoup">Launchpad</a>. You can <a href="https://code.launchpad.net/beautifulsoup/">get the source
code</a> or <a href="https://bugs.launchpad.net/beautifulsoup/">file
bugs</a>.<hr/><table><tr><td valign="top">
<p>This document (<a href="/source/software/BeautifulSoup/index.bhtml">source</a>) is part of Crummy, the webspace of <a href="/self/">Leonard Richardson</a> (<a href="/self/contact.html">contact information</a>). It was last modified on Friday, January 31 2020, 13:44:05 Nowhere Standard Time and last built on Friday, March 06 2020, 16:00:02 Nowhere Standard Time.</p><p><table class="licenseText"><tr><td><a href="http://creativecommons.org/licenses/by-sa/2.0/"><img border="0" src="/nb//resources/img/somerights20.jpg"/></a></td><td valign="top">Crummy is © 1996-2020 Leonard Richardson. Unless otherwise noted, all text licensed under a <a href="http://creativecommons.org/licenses/by-sa/2.0/">Creative Commons License</a>.</td></tr></table></p></td></tr></table></p></p></p></p></p></p></p></p></p></p></p><!--<rdf:RDF xmlns="http://web.resource.org/cc/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"><Work rdf:about="http://www.crummy.com/"><dc:title>Crummy: The Site</dc:title><dc:rights><Agent><dc:title>Crummy: the Site</dc:title></Agent></dc:rights><dc:format>text/html</dc:format><license rdf:resource=http://creativecommons.org/licenses/by-sa/2.0//></Work><License rdf:about="http://creativecommons.org/licenses/by-sa/2.0/"></License></rdf:RDF>--><td valign="top"><p><b>Document tree:</b>
<dl><dd><a href="http://www.crummy.com/">http://www.crummy.com/</a><dl><dd><a href="http://www.crummy.com/software/">software/</a><dl><dd><a href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup/</a></dd></dl>
</dd></dl>
</dd></dl>


Site Search:

<form action="/search/" method="get">
<input maxlength="255" name="q" type="text" value=""/>
</form>
</p></td>




>
```

可以看到，这个页面正常输出了，说明BeautifulSoup成功解析了我们给出的页面。

BeautifuSoup的使用简单来说只有两行代码：

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup('<p>data</p>', 'html.parser')
# soup = BeautifulSoup(open("/User/gukaifeng/demo.html"), 'html.parser')
```

这里`BeautifulSoup`是一个类。

`BeautifulSoup`有两个参数，第一个是待解析的html/xml文本，第二个是解释器。

## 2. BeautifulSoup库的基本元素

### 2.1. BeautifulSoup库解析器

| 解析器           | 使用方法                         | 条件                 |
| ---------------- | -------------------------------- | -------------------- |
| bs4的HTML解析器  | BeautifulSoup(mk, 'html.parser') | 安装bs4库            |
| lxml的HTML解析器 | BeautifulSoup(mk, 'lxml')        | pip install lxml     |
| lxml的XML解析器  | BeautifulSoup(mk, 'xml')         | pip install lxml     |
| html5lib的解析器 | BeautifulSoup(mk, 'html5lib')    | pip install html5lib |

### 2.2. BeautifulSoup类的基本元素

| 基本元素        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Tag             | 标签。最基本的信息组织单元，分别用`<>`和`</>`标明开头和结尾。 |
| Name            | 标签的名字。如`<p>...</p>`的名字是`p`，格式为` <tag>.name`。 |
| Attributes      | 标签的属性，字典形式组织，格式为`<tag>.attrs`。              |
| NavigableString | 标签内非属性字符串，`<>...</>`中的字符串，格式为`<tag>.string`。 |
| Comment         | 标签内字符串的注释部分，一种特殊的Comment类型。              |

#### 2.2.1. 获取页面的标签

接上面安装章节的测试代码，我们获取一下页面的标签。

```python
>>> soup.title
<title>Beautiful Soup: We called him Tortoise because he taught us.</title>
>>> soup.a
<a href="#Download">Download</a>
```

我们知道页面中的`<a>...</a>`有很多个，当页面中同时存在多个`tag`标签时，`soup.tag`返回第一个。

#### 2.2.2. 获取标签的名字

我们以此获取`<a>...</a>`标签的名字，其父亲的名字，其父亲的父亲的名字。

注意这里的`soup.a`指的是页面中第一个`<a>...</a>`标签。

```python
>>> soup.a.name
'a'
>>> soup.a.parent.name
'p'
>>> soup.a.parent.parent.name
'body'
```

#### 2.2.3. 获取标签的属性

我们知道标签的属性是在标签中标明标签特点的区域，以字典形式存储。

```python
>>> soup.meta.attrs
{'http-equiv': 'Content-Type', 'content': 'text/html; charset=utf-8'}
>>> soup.meta.attrs['content']
'text/html; charset=utf-8'
>>> type(soup.meta.attrs)
<class 'dict'>
>>> type(soup.meta)
<class 'bs4.element.Tag'>
```

从上面的代码可以看到，标签的属性以字典形式存储，因此我们可以用访问字典的方式来访问标签的属性。

标签属性字典的类型就是`dict`，而标签本身的类型是`bs4.element.Tag`。

另外，如果标签没有属性，我们会获得一个空字典。<br/>也就是说无论怎样，我们调用标签的`attrs`属性总能获得一个字典。

#### 2.2.4. 获取标签的内容

```python
>>> soup.a
<a href="#Download">Download</a>
>>> soup.a.string
'Download'
>>> type(soup.a.string)
<class 'bs4.element.NavigableString'>
```

获取标签内容的格式为`<tag>.string`。标签内容的类型为`bs4.element.NavigableString`。

#### 2.2.5. 标签中注释内容的处理

```python
>>> newsoup = BeautifulSoup('<b><!--This is a comment.--></b><p>This is not a comment.</p>', 'html.parser')
>>> newsoup.b.string
'This is a comment.'
>>> type(newsoup.b.string)
<class 'bs4.element.Comment'>
>>> newsoup.p.string
'This is not a comment.'
>>> type(newsoup.p.string)
<class 'bs4.element.NavigableString'>
```

为了突出重点，我们新建一个BeautifulSoup类对象，其中只包含注释和非注释内容。

代码中可以看到，对于`<b>...</b>`标签和`<p>...</p>`标签，我们都获得了其中的文本内容，对于`<b>...</b>`标签更是忽略了其中的注释符号，直接获取到了文本。

查看类型可以发现，注释内容的类型为`bs4.element.Comment`，非注释内容类型为`bs4.element.NavigableString`。如果我们希望过滤掉注释内容，那么久可以通过判断类型的方式来实现。