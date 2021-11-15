---
title: 基于bs4库的HTML内容遍历方法
mathjax: false
date: 2020-03-07 09:47:51
updated: 2020-03-07 09:47:51
tags: [Python,bs4,BeautifulSoup,爬虫]
categories: [爬虫]
toc: true
---

## 1. HTML基本格式

如果我们把HTML代码做一个结构化的设计，可以发现HTML代码是一个具有树形结构的文本信息。

事实上任何一个HTML或XML文件都是这样标签树形结构，这样的树形结构形成了三种遍历方法。

1. 下行遍历：从根节点开始向叶结点遍历。
2. 上行遍历：从叶结点开始向根节点遍历。
3. 平行遍历：在平级结点之间遍历。
<!--more-->
注意，一个标签结点的孩子结点不仅包括标签结点，也包括字符串结点，如`'\n'`等。

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E5%9F%BA%E4%BA%8Ebs4%E5%BA%93%E7%9A%84HTML%E5%86%85%E5%AE%B9%E9%81%8D%E5%8E%86%E6%96%B9%E6%B3%95_1.png)

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E5%9F%BA%E4%BA%8Ebs4%E5%BA%93%E7%9A%84HTML%E5%86%85%E5%AE%B9%E9%81%8D%E5%8E%86%E6%96%B9%E6%B3%95_2.png)



## 2. 标签树的三种遍历方法

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E5%9F%BA%E4%BA%8Ebs4%E5%BA%93%E7%9A%84HTML%E5%86%85%E5%AE%B9%E9%81%8D%E5%8E%86%E6%96%B9%E6%B3%95_3.png)

### 2.1. 下行遍历

标签树的下行遍历一共包含三个属性，如下。

| 属性         | 说明                                                        |
| ------------ | ----------------------------------------------------------- |
| .contents    | 子节点的列表，将`<tag>`所有儿子结点存入列表。               |
| .children    | 子节点的迭代类型，与`.contents`类似，用于循环遍历儿子结点。 |
| .descendants | 子孙结点的迭代类型，包含所有子孙结点，用于循环遍历。        |

```python
>>> soup.head.contents
['\n', <meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>, '\n', <title>Beautiful Soup: We called him Tortoise because he taught us.</title>, '\n', <link href="mailto:leonardr@segfault.org" rev="made"/>, '\n', <link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>, '\n', <meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>, '\n', <meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>, '\n', <meta content="Leonard Richardson" name="author"/>, '\n']
>>> type(soup.head.contents)
<class 'list'>
>>> len(soup.head.contents)
15
>>> soup.head.contents[1]
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
```

上述代码演示了`.contents`属性的一些内容，它的返回结果是一个链表，故我们可以用索引的方式取值。

如果想要对儿子结点标签进行遍历，用`.contents`或`.children`都是可以的。

对全部子孙结点标签进行遍历要用`.descendants`，如下，一些解析见注释。

```python
>>> import bs4 # 过滤结点代码用到了bs4.element.Tag，前面没有import
>>> for child in soup.head.children:
...     if(isinstance(child, bs4.element.Tag)): # 过滤字符串类型孩子结点
...         print(repr(child))
... 
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
<title>Beautiful Soup: We called him Tortoise because he taught us.</title>
<link href="mailto:leonardr@segfault.org" rev="made"/>
<link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>
<meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>
<meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>
<meta content="Leonard Richardson" name="author"/>
```

```python
>>> for child in soup.head.descendants:
...     if(child.name != None):
...         print(child)
... 
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
<title>Beautiful Soup: We called him Tortoise because he taught us.</title>
<link href="mailto:leonardr@segfault.org" rev="made"/>
<link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>
<meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>
<meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>
<meta content="Leonard Richardson" name="author"/>
```

另外注意，这里的孩子结点中既包括了标签，也包括了标签的属性、内容等信息。

### 2.2. 上行遍历

标签树的下行遍历一共包含两个属性，如下。

| 属性     | 说明                                           |
| -------- | ---------------------------------------------- |
| .parent  | 结点的父亲标签。                               |
| .parents | 结点先辈标签的迭代类型，用于循环遍历先辈结点。 |

```python
>>> soup.title.parent
<head>
<meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
<title>Beautiful Soup: We called him Tortoise because he taught us.</title>
<link href="mailto:leonardr@segfault.org" rev="made"/>
<link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>
<meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>
<meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>
<meta content="Leonard Richardson" name="author"/>
</head>
>>> soup.title.parents
<generator object PageElement.parents at 0x10bfe75e8>
>>> type(soup.title.parents)
<class 'generator'>
```

```python
>>> for parent in soup.head.parents: # 遍历所有先辈结点
...     print(parent.name) # 先辈结点中有包括了整个文本的<html></html>标签，内容太多，这里只输出其name属性
... 
html
[document]
```

另外注意，这里的父亲和先辈结点中既包括了标签，也包括了标签的属性、内容等信息。

### 2.3. 平行遍历

平行遍历发生在同一个父节点下的各节点之间。

![](https://raw.githubusercontent.com/gukaifeng/PicGo/master/img/%E5%9F%BA%E4%BA%8Ebs4%E5%BA%93%E7%9A%84HTML%E5%86%85%E5%AE%B9%E9%81%8D%E5%8E%86%E6%96%B9%E6%B3%95_4.png)

标签树的平行遍历一共包含四个属性，如下。

| 属性               | 说明                                                     |
| ------------------ | -------------------------------------------------------- |
| .next_sibling      | 返回按照HTML文本顺序的下一个平行结点标签。               |
| .previous_sibling  | 返回按照HTML文本顺序的上一个平行结点标签。               |
| .next_siblings     | 迭代类型，返回按照HTML文本书顺序的后续所有平行结点标签。 |
| .previous_siblings | 迭代类型，返回按照HTML文本书顺序的前续所有平行结点标签。 |

遍历前续平行结点和遍历后续平行结点的操作是完全一样的，这里以后续为例。

```python
>>> soup.a.next_sibling # 获取soup中第一个<a>...</a>标签的下一个标签
' | '
>>> soup.a.next_sibling.next_sibling
<a href="bs4/doc/">Documentation</a>
```

```python
>>> import bs4 # 过滤结点代码用到了bs4.element.Tag，前面没有import
>>> for sibling in soup.a.next_siblings: # 遍历后续所有平行结点标签
...     if(isinstance(sibling, bs4.element.Tag)):
...         print(sibling)
... 
<a href="bs4/doc/">Documentation</a>
<a href="#HallOfFame">Hall of Fame</a>
<a href="enterprise.html">For enterprise</a>
<a href="https://code.launchpad.net/beautifulsoup">Source</a>
<a href="https://bazaar.launchpad.net/%7Eleonardr/beautifulsoup/bs4/view/head:/CHANGELOG">Changelog</a>
<a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">Discussion group</a>
<a href="zine/">Zine</a>
```

