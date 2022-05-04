---
title: "MIT6.824 Lab1,2,3,4 经验与总结"
date: 2022-04-09 18:05:00
updated: 2022-05-03 01:49:00
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

Lab1 是实现自己的 MapReduce 模型，比较简单，在 6.824 这里我感觉就是给大家练手的。

主要适合我这种之前没用过 Go 语言的，这个 Lab 被我用来熟悉 Go 了。

没什么好讲的，只要按着论文里描述的写就好了。后面的几个 Lab 才是重头戏。

下面是正确实现后的输出，用作参考就好了。

```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z main]$ sh ./test-mr.sh
*** Starting wc test.
--- wc test: PASS
*** Starting indexer test.
--- indexer test: PASS
*** Starting map parallelism test.
--- map parallelism test: PASS
*** Starting reduce parallelism test.
--- reduce parallelism test: PASS
*** Starting crash test.
--- crash test: PASS
*** PASSED ALL TESTS
```

除了上面的这些外，你可能还会看到很多下面这样的输出，忽略就好。

```
2022/05/02 19:22:05 rpc.Register: method "Done" has 1 input parameters; needs exactly three
```

## Lab 2: Raft

写这个 lab 的时候，我的 Go 语言依然不熟练，所以下面列出的我遇到的问题，可能有些比较基础。



1\. RPC 的参数传输到接收方后变成了空指针，调了很久，最后发现是参数中不能传自己定义的结构体类型。

2\. 领导者完整性(Leader Completeness)属性中，要求一个候选人需要包含全部已提交的条目，但不需要这些条目在候选人上是已提交的。在实际操作时，只需要验证，候选人比追随者的日志更新就可以。[“更新”在论文中有精确定义](https://gukaifeng.cn/posts/raft-lun-wen-yue-du-bi-ji/#5-4-1-%E9%80%89%E4%B8%BE%E9%99%90%E5%88%B6)：Raft 通过比较日志中的最后一个条目的索引和任期来判断两个日志哪个是更新的。如果两个日志具有不同任期的最后条目，则具有较晚任期的日志是最新的。如果两个日志以相同的任期结束，那么认为较长的日志是最新的。

3\. RPC 中 args 和 reply 参数不能为空指针。

4\. 一个追随者收到 AppendEntries RPC 时，判定其是否为有效的 RPC，应该看 RPC 中的任期是否大于等于其自己最后一个条目的 RPC。因为如果这个追随者与其他服务器失联很久，其会不停的开始新选举，导致其自己的任期非常大，进而拒绝恢复连接后的领导者的有效 RPC。

5\. 在调试 lab2B 的时候，经常会多线程打印内容，已经不太容易了，而多个测试挨个进行还会加大难度，建议调试时使用 go test 测试单独某个函数。

6\. 主进程退出，协程也都会结束，所以要让主进程等待协程，可以用 WaitGroup。

7\. 理论上说，领导者应该把一个日志应用到其状态机后，才响应客户端。但是在 TestFailNoAgree2B 测试中，测试代码写的是，没有成功应用状态机，也要返回给客户端正确的响应，这里没太搞清楚。

8\. Start() 只是表示开始进行一致性算法，只要算法能正确开始，就可以返回了，而不是等待一致性算法完成才返回。

9\. 2B 第五个测试 concurrent start 遇到个问题，单独执行能过，和前面四个测试换位执行能过，但一起执行就过不了。做了点简单的 debug 后，我认为我的代码没有问题，肯定是可以通过的。问题大概应该是，测试受到了前面测试的影响。

10\. 我调了最久的一个问题是，2C 中的几个 unreliable 的 apply error 问题，最终发现我的问题可能存在两个方面，一个是一致性代码在大部分追随者上复制完成后就返回了，导致剩下追随者的 go 线程中断，最后使用 waitgroup 等待全部都完成再返回。另一个是没有正确使用 matchIndex，领导者的 commitIndex 应该只在比对 matchIndex 的时候更新，即 Figure2 最右下角说的那样。

11\. 一个偶尔会出现的问题 `config.go:475: one() failed to reach agreement`，我认为这个问题主要是因为，测试程序对达成一致的时间限制太短，而程序受各种因素影响可能导致速度变慢，我的机器配置也比较低（4C8G），但代码应该没啥问题。





这个 Lab2 在课程里要求使用  `time` 命令 计时时， `real` 小于 4 分钟，`user` 小于 1分钟。

本人的操作机器的配置不算太高，是 CentOS 8 4C8G 的。下面是我通过的输出。

```
[gukaifeng@iZ8vbf7xcuoq7ug1e7hjk5Z raft]$ time go test
Test (2A): initial election ...
  ... Passed --   3.1  3  110   30232    0
Test (2A): election after network failure ...
  ... Passed --   4.5  3  128   24809    0
Test (2B): basic agreement ...
  ... Passed --   0.6  3   14    3794    3
Test (2B): RPC byte count ...
  ... Passed --   0.9  3   46  113246   11
Test (2B): agreement despite follower disconnection ...
  ... Passed --   5.4  3  209   55562    8
Test (2B): no agreement if too many followers disconnect ...
  ... Passed --   3.5  5  350   77663    3
Test (2B): concurrent Start()s ...
  ... Passed --   0.6  3   24    6818    6
Test (2B): rejoin of partitioned leader ...
  ... Passed --   2.3  3   99   22613    4
Test (2B): leader backs up quickly over incorrect follower logs ...
  ... Passed --  12.8  5 1334  632664  102
Test (2B): RPC counts aren't too high ...
  ... Passed --   2.1  3   98   27906   12
Test (2C): basic persistence ...
  ... Passed --   3.5  3  128   30655    6
Test (2C): more persistence ...
  ... Passed --  11.0  5  798  153220   16
Test (2C): partitioned leader and one follower crash, leader restarts ...
  ... Passed --   1.6  3   47   11317    4
Test (2C): Figure 8 ...
  ... Passed --  29.1  5 1482  294102   53
Test (2C): unreliable agreement ...
  ... Passed --   3.0  5  708  346841  246
Test (2C): Figure 8 (unreliable) ...
  ... Passed --  31.5  5 1750  440162   73
Test (2C): churn ...
  ... Passed --  16.2  5 1452 1261721  662
Test (2C): unreliable churn ...
  ... Passed --  16.1  5 1965  891346  533
PASS
ok  	_/home/gukaifeng/projects/6.824/src/raft	147.686s

real	2m27.916s
user	0m4.425s
sys	0m0.900s
```



## Lab 3: Fault-tolerant Key/Value Service

1\. 遇到的第一个问题是，测试 `TestUnreliableOneKey3A` 耗时二十几秒才能完成，远超预期。

```
Test: concurrent append to same key, unreliable (3A)
```

最后发现原因是，因为我设置了客户端请求超时重试（即服务器太久没有返回 RPC 结果时重试），但超时时间设置的太短或太长。太短会导致有些请求服务器正在处理，没有故障，但还没处理完就被客户端放弃了。太长会导致遇到故障服务器时，客户端等的太久。我最后设置在 200-300 ms，是个比较合适的范围。

2\. 第二个问题是，3A 的最后 6 个测试，即带有 `restart` 标签的测试，耗时均是课程样例输出的 2 倍。

## Lab 4: Sharded Key/Value Service

