---
title: "MIT6.824 Lab1,2,3,4 经验与总结"
date: 2022-04-09 18:05:00
updated: 2022-04-09 18:05:00
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

RPC 中 args 和 reply 参数不能为空指针。

一个追随者收到 AppendEntries RPC 时，判定其是否为有效的 RPC，应该看 RPC 中的任期是否大于等于其自己最后一个条目的 RPC。因为如果这个追随者与其他服务器失联很久，其会不停的开始新选举，导致其自己的任期非常大，进而拒绝恢复连接后的领导者的有效 RPC。
领导者完整性中，一个候选人需要包含全部已提交的条目，但不需要这些条目在候选人上是已提交的。

## Lab 3: Fault-tolerant Key/Value Service



## Lab 4: Sharded Key/Value Service

