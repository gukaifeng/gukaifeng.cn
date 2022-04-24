---
title: "MIT6.824 Lab1,2,3,4 经验与总结"
date: 2022-04-09 18:05:00
updated: 2022-04-24 23:27:00
categories: [技术杂谈]
tags: [MIT6.824,GFS,MapReduce,Raft,论文,分布式]
mathjax: true
---





> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>







课程主页传送门：<font size=4px>[6.824: Distributed Systems](http://nil.csail.mit.edu/6.824/2020/)</font>

4 个 lab 传送门：  
[Lab 1: MapReduce](http://nil.csail.mit.edu/6.824/2020/labs/lab-mr.html)  
[Lab 2: Raft](http://nil.csail.mit.edu/6.824/2020/labs/lab-raft.html)  
[Lab 3: Fault-tolerant Key/Value Service](http://nil.csail.mit.edu/6.824/2020/labs/lab-kvraft.html)  
[Lab 4: Sharded Key/Value Service](http://nil.csail.mit.edu/6.824/2020/labs/lab-shard.html)



## Lab 1: MapReduce



## Lab 2: Raft

RPC 的参数传输到接收方后变成了空指针，调了很久，最后发现是参数中不能传自己定义的结构体类型。

领导者完整性中，一个候选人需要包含全部已提交的条目，但不需要这些条目在候选人上是已提交的。

RPC 中 args 和 reply 参数不能为空指针。

一个追随者收到 AppendEntries RPC 时，判定其是否为有效的 RPC，应该看 RPC 中的任期是否大于等于其自己最后一个条目的 RPC。因为如果这个追随者与其他服务器失联很久，其会不停的开始新选举，导致其自己的任期非常大，进而拒绝恢复连接后的领导者的有效 RPC。

在调试 lab2B 的时候，经常会多线程打印内容，已经不太容易了，而多个测试挨个进行还会加大难度，建议在测试代码中注释掉其他的测试。

主进程退出，协程也都会结束，所以要让主进程等待协程，可以用 WaitGroup 方法。\

理论上说，领导者应该把一个日志应用到其状态机后，才响应客户端。但是在 TestFailNoAgree2B 测试中，测试代码写的是，没有成功应用状态机，也要返回给客户端正确的响应，这里没太搞清楚。

Start() 只是表示开始进行一致性算法，只要算法能正确开始，就可以返回了，而不是等待一致性算法完成才返回。

2B 第五个测试 concurrent start 遇到个问题，单独执行能过，和前面四个测试换位执行能过，但一起执行就过不了。做了点简单的 debug 后，我认为我的代码没有问题，肯定是可以通过的。问题大概应该是，测试受到了前面测试的影响。

## Lab 3: Fault-tolerant Key/Value Service



## Lab 4: Sharded Key/Value Service

