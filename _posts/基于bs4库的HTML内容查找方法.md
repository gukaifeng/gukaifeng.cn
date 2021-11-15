---
title: 基于bs4库的HTML内容查找方法
mathjax: false
date: 2020-03-08 00:50:46
updated: 2020-03-08 00:50:46
tags: [bs4, BeautifulSoup, 爬虫]
categories: [网络爬虫]
toc: true
---

我们有一份由`Requests`库`request.get()`方法获得的`demo.html`文本如下。

`demo.html`的文本信息存储于`demo`变量中，且有`soup = BeautifulSoup(demo, "html.parser")`。



![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E5%9F%BA%E4%BA%8Ebs4%E5%BA%93%E7%9A%84HTML%E5%86%85%E5%AE%B9%E6%9F%A5%E6%89%BE%E6%96%B9%E6%B3%95_1.png)

BeautifulSoup库提供了一个方法`find_all()`，这个方法可以在soup的变量中去查找一些信息。

`find_all()`有5个参数，返回一个列表类型，存储查找的结果。

`<>.find_all(name, attrs, recursive, string, **kwargs)`

下面我们对`find_all()`方法的每个参数进行说明，每次调用的说明写在注释中。

<!--more-->

* **`name`：对标签名称的检索字符串。**

    ```python
    >>> soup.find_all('a') # 查找<a>
    [<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
    ```

    ```python
    >>> soup.find_all(['a', 'b']) # 查找<a>和<b>
    [<b>The demo python introduces several python courses.</b>, <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
    ```

    ```python
    >>> soup.find_all(True) # 如果name参数值为True，那么将查找所有标签。
    [<html><head><title>This is a python demo page</title></head>
    <body>
    <p class="title"><b>The demo python introduces several python courses.</b></p>
    <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>
    </body></html>, <head><title>This is a python demo page</title></head>, <title>This is a python demo page</title>, <body>
    <p class="title"><b>The demo python introduces several python courses.</b></p>
    <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>
    </body>, <p class="title"><b>The demo python introduces several python courses.</b></p>, <b>The demo python introduces several python courses.</b>, <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>, <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
    ```

    ```python
    >>> import re
    >>> for tag in soup.find_all(re.compile('b')): # 查找所有的以'b'开头的标签
    ...     print(tag.name)
    ... 
    body
    b
    ```

* **`attrs`：对标签class属性值的检索字符串，可标注属性检索。**

    ```python
    >>> soup.find_all('p') # 未使用attrs字段，查找所有<p>标签
    [<p class="title"><b>The demo python introduces several python courses.</b></p>, <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>]
    ```

    ```python
    >>> soup.find_all('p', 'course')  # 查找有class属性'course'的<p>
    [<p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>]
    ```

    ```python
    >>> kv = {'class': 'course'}  # attrs可以设置很多个匹配的属性值
    >>> soup('p', attrs=kv)
    [<p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
    <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>]
    ```

    ```python
    >>> soup.find_all(id='link1') # 查找有id=‘link1’属性的所有标签
    [<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
    >>> soup.find_all(id='link') # 查找有id=‘link’属性的所有标签
    []
    >>> soup.find_all('a',id='link1') # 查找有id=‘link’属性的所有<a>标签
    [<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
    >>> soup.find_all(id=re.compile('link')) # 查找所有包含id='link*'的标签
    [<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
    ```

* **`recursive：是否对子孙全部检索，默认为True。`**

    ```python
    >>> soup.find_all('a')
    [<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>, <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>]
    >>> soup.find_all('a', recursive=False) # 结果[]说明，soup结点的儿子结点中无<a>。<a>在子孙中
    []
    ```

* **`string`：`<>...</>`中字符串区域的检索字符串。**

    ```python
    >>> soup.find_all(string = "Basic Python") # 检索"Basic Python"字符串，一字不能差，必须是<>...</>内的完整内容
    ['Basic Python']
    >>> soup.find_all(string = re.compile('Python')) # 检索包含"Python"字符串的<>...</>内的完整内容
    ['Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:\r\n', 'Basic Python', 'Advanced Python']
    ```

    



**由于bs4中的`find_all()`方法极其常用，所以bs4提供了简写形式。**

`<tag>()`等价于`<tag>.find_all()` <br/>`soup()`等价于`soup.find_all()`

```python
>>> soup(string = "Basic Python", id = "link1") # find_all() 完整写法
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
>>> soup.find_all(string = "Basic Python", id = "link1") # find_all() 简写
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
```

另外`find_all()`方法还有7个扩展的常用方法，这些方法的参数都与`find_all()`的参数完全一样。

| 方法                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| <>.find()                   | 搜索结果中只返回一个结果，字符串类型。参数同 `find_all()`。  |
| <>.find_parents()           | 在先辈结点中搜索，返回列表类型。参数同 `find_all()`。        |
| <>.find_parent()            | 在先辈结点中返回一个结果，字符串类型。参数同 `find_all()`。  |
| <>.find_next_siblings()     | 在后续平行结点中搜索，返回列表类型。参数同 `find_all()`。    |
| <>.find_next_sibling()      | 在后续平行结点中返回一个结果，字符串类型。参数同 `find_all()`。 |
| <>.find_previous_siblings() | 在前续平行结点中搜索，返回列表类型。参数同 `find_all()`。    |
| <>.find_previous_sibling()  | 在前续平行结点中返回一个结果，字符串类型。参数同 `find_all()`。 |

