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

1\. RPC 的参数传输到接收方后变成了空指针，调了很久，最后发现是参数中不能传自己定义的结构体类型。

2\. 领导者完整性中，一个候选人需要包含全部已提交的条目，但不需要这些条目在候选人上是已提交的。

3\. RPC 中 args 和 reply 参数不能为空指针。

4\. 一个追随者收到 AppendEntries RPC 时，判定其是否为有效的 RPC，应该看 RPC 中的任期是否大于等于其自己最后一个条目的 RPC。因为如果这个追随者与其他服务器失联很久，其会不停的开始新选举，导致其自己的任期非常大，进而拒绝恢复连接后的领导者的有效 RPC。

5\. 在调试 lab2B 的时候，经常会多线程打印内容，已经不太容易了，而多个测试挨个进行还会加大难度，建议调试时使用 go test 测试单独某个函数。

6\. 主进程退出，协程也都会结束，所以要让主进程等待协程，可以用 WaitGroup。

7\. 理论上说，领导者应该把一个日志应用到其状态机后，才响应客户端。但是在 TestFailNoAgree2B 测试中，测试代码写的是，没有成功应用状态机，也要返回给客户端正确的响应，这里没太搞清楚。

8\. Start() 只是表示开始进行一致性算法，只要算法能正确开始，就可以返回了，而不是等待一致性算法完成才返回。

9\. 2B 第五个测试 concurrent start 遇到个问题，单独执行能过，和前面四个测试换位执行能过，但一起执行就过不了。做了点简单的 debug 后，我认为我的代码没有问题，肯定是可以通过的。问题大概应该是，测试受到了前面测试的影响。

10\. 我调了最久的一个问题是，2C 中的几个 unreliable 的 apply error 问题，最终发现我的问题可能存在两个方面，一个是一致性代码在大部分追随者上复制完成后就返回了，导致剩下追随者的 go 线程中断，最后使用 waitgroup 等待全部都完成再返回。另一个是没有正确使用 matchIndex，领导者的 commitIndex 应该只在比对 matchIndex 的时候更新，即 Figure2 最右下角说的那样。

11\. 一个偶尔会出现的问题 `config.go:475: one() failed to reach agreement`，我认为这个问题主要是因为，测试程序对达成一致的时间限制太短，而程序受各种因素影响可能导致速度变慢，我的机器配置也比较低（4C8G），但代码应该没啥问题。



```shell
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z raft]$ time go test
Test (2A): initial election ...
  ... Passed --   3.0  3   56   15274    0
Test (2A): election after network failure ...
  ... Passed --   4.5  3  118   18516    0
Test (2B): basic agreement ...
  ... Passed --   0.5  3   14    3766    3
Test (2B): RPC byte count ...
  ... Passed --   1.3  3   46  113154   11
Test (2B): agreement despite follower disconnection ...
  ... Passed --   5.6  3  149   36218    8
Test (2B): no agreement if too many followers disconnect ...
  ... Passed --   3.5  5  269   48910    4
Test (2B): concurrent Start()s ...
  ... Passed --   0.5  3   20    5532    6
Test (2B): rejoin of partitioned leader ...
  ... Passed --   2.1  3   70   14693    4
Test (2B): leader backs up quickly over incorrect follower logs ...
  ... Passed --  22.5  5 2669  500322  102
Test (2B): RPC counts aren't too high ...
  ... Passed --   2.2  3   52   14758   12
Test (2C): basic persistence ...
  ... Passed --   3.6  3  100   22990    6
Test (2C): more persistence ...
  ... Passed --  13.3  5  992  168247   17
Test (2C): partitioned leader and one follower crash, leader restarts ...
  ... Passed --   1.4  3   41    9578    4
Test (2C): Figure 8 ...
  ... Passed --  35.8  5 2085  397433   83
Test (2C): unreliable agreement ...
  ... Passed --   5.2  5  738  228682  252
Test (2C): Figure 8 (unreliable) ...
  ... Passed --  32.3  5 2716  350697   29
Test (2C): churn ...
  ... Passed --  16.3  5 1206  346655  316
Test (2C): unreliable churn ...
  ... Passed --  16.3  5 1344  377140  217
PASS
ok  	_/home/gukaifeng/projects/6.824/src/raft	169.711s

real	2m49.930s
user	0m3.908s
sys	0m0.936s
```



## Lab 3: Fault-tolerant Key/Value Service



## Lab 4: Sharded Key/Value Service

