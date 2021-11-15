---
title: Requests库和Scrapy框架的比较
mathjax: false
date: 2020-03-15 11:08:27
updated: 2020-03-15 11:08:27
tags: [Requests,Scrapy,爬虫]
categories: [网络爬虫]
toc: true
---

Requests库和Scrapy框架都是Python中非常重要的第三方爬虫库（框架）。

## 1. Requests和Scrapy的相同点

1. 两者都可以进行页面请求和爬取，是Python爬虫的两个重要技术路线。
2. 两者都可用性很好，文档丰富，入门简单。
3. 两者都没有处理js、提交表单、应对验证码等功能（可扩展）。



## 2. Requests和Scrapy的不同点

| Requests                 | Scrapy                     |
| ------------------------ | -------------------------- |
| 页面级爬虫               | 网站级爬虫                 |
| 功能库                   | 框架                       |
| 并发性考虑不足，性能较差 | 并发性好，性能较高         |
| 重点在于页面下载         | 重点在于爬虫结构           |
| 定制灵活                 | 一般定制灵活，深度定制困难 |
| 上手十分简单             | 入门稍难                   |



## 3. 如何选择Requests和Scrapy？

* 非常小的需求：Requests库。

* 不太小的需求：Scrapy框架。
* 定制程度很高的需求（不考虑规模），自搭框架：Requests > Scrapy。因为Scrapy的“5+2”结构在自搭框架时反而会成为限制，不如直接用Requests库来编写。

