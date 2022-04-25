---
title: "MIT6.824 Students' Guide to Raft 翻译"
date: 2022-04-24 23:30:00
updated: 2022-04-24 23:30:00
categories: [技术杂谈]
tags: [MIT6.824,Raft,论文,分布式]
---



> <font color=red>请注意，此文章尚未完成。</font>  
> <font color=red>当此文章完结时，此声明将被删除。</font>



原文传送门：[Students' Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/)

> 此文章为原文翻译，非本人原创文章！此外，翻译内的人称均和原文保持一致！





在过去的几个月里，我一直是麻省理工学院 [6.824: Distributed Systems](https://pdos.csail.mit.edu/6.824/) 课程的助教。传统上，该课程有许多基于 Paxos 共识算法的实验，但今年，我们决定转向 [Raft](https://raft.github.io/)。Raft 的设计“易于理解”，我们希望这种改变可以让学生的生活更轻松。

这篇文章以及随附的 [Instructors' Guide to Raft](https://gukaifeng.cn/posts/mit6.824-raft-q-a-fan-yi/) 文章记录了我们使用 Raft 的旅程，希望对 Raft 协议的实现者和试图更好地理解 Raft 内部结构的学生有用。如果您正在寻找 Paxos 与 Raft 的比较，或者想要对 Raft 进行更多的教学分析，您应该阅读 Instructors' Guide。这篇文章的底部包含 6.824 名学生常见的问题列表，以及这些问题的答案。如果您遇到本文主要内容中未列出的问题，请查看 [Q&A](https://gukaifeng.cn/posts/mit6.824-instructors-guide-to-raft-fan-yi/)。这篇文章很长，但它提出的所有观点都是很多 6.824 名学生（和助教）遇到的实际问题。这篇文章值得一读。



## 1. 背景

在我们深入研究 Raft 之前，一些上下文可能很有用。6.824 曾经有一组[基于 Paxos 的实验](http://nil.csail.mit.edu/6.824/2015/labs/lab-3.html)，这些实验是用 [Go](https://go.dev/) 构建的；之所以选择 Go，是因为它对学生来说很容易学习，而且非常适合编写并发的分布式应用程序（goroutines 特别方便）。在四个实验的过程中，学生们构建了一个容错的分片键值存储。第一个实验让他们构建了一个基于共识的日志库，第二个实验室在此基础上添加了一个键值存储，第三个实验室在多个容错集群之间分片了键空间，并由一个容错分片主机处理配置更改。我们还有第四个实验，学生必须在其中处理机器的故障和恢复，无论磁盘是否完好，该实验室可作为学生的默认期末项目。

今年，我们决定使用 Raft 重写所有这些实验。前三个实验室都是一样的，但是第四个实验室被放弃了，因为持久性和故障恢复已经内置在 Raft 中。本文将主要讨论我们在第一个实验中的经验，因为它是与 Raft 最直接相关的一个，但我还将涉及在 Raft 之上构建应用程序（如在第二个实验中）。

对于那些刚刚了解 Raft 的人来说，Raft 最好由协议[网站](https://raft.github.io/)上的文本描述：

>Raft is a consensus algorithm that is designed to be easy to understand. It’s equivalent to Paxos in fault-tolerance and performance. The difference is that it’s decomposed into relatively independent subproblems, and it cleanly addresses all major pieces needed for practical systems. We hope Raft will make consensus available to a wider audience, and that this wider audience will be able to develop a variety of higher quality consensus-based systems than are available today.
>
>Raft 是一种旨在易于理解的共识算法。它在容错性和性能上与 Paxos 相当。不同之处在于它被分解为相对独立的子问题，并且清晰地解决了实际系统所需的所有主要部分。我们希望 Raft 能够将共识提供给更广泛的受众，并且这些更广泛的受众将能够开发出比现在可用的各种更高质量的基于共识的系统。

像[这样](http://thesecretlivesofdata.com/raft/)的可视化很好地概述了协议的主要组成部分，并且该论文对为什么需要各种部分提供了很好的直觉。如果你还没有阅读过 [extended Raft](https://raft.github.io/raft.pdf) 论文，那么在继续本文之前你应该先阅读它，因为我假设你对 Raft 相当熟悉。

与所有分布式共识协议一样，细节非常重要。在没有故障的稳定状态下，Raft 的行为很容易理解，并且可以用直观的方式来解释。例如，从可视化中很容易看出，假设没有失败，最终会选出一个领导者，最终发送给领导者的所有操作都将由跟随者以正确的顺序应用。但是，当引入延迟消息、网络分区和故障服务器时，每一个 if、but、和 and 都变得至关重要。特别是，由于阅读论文时的误解或疏忽，我们一遍又一遍地看到了许多错误。这个问题并不是 Raft 独有的，而是在所有提供正确性的复杂分布式系统中都会出现的问题。



## 2. 实现 Raft

Raft 的终极指南在 Raft 论文的图 2 中。该图指定了 Raft 服务器之间交换的每个 RPC 的行为，给出了服务器必须维护的各种不变量，并指定了何时应该发生某些操作。我们将在本文的其余部分大量讨论图 2。它需要信守承诺。

> 查看 Raft 中的图 2
>
> 中文翻译版：https://gukaifeng.cn/posts/raft-lun-wen-yue-du-bi-ji/Raft_Figure_2_Chinese.png  
> 英文原版：https://gukaifeng.cn/posts/raft-lun-wen-yue-du-bi-ji/Raft_Figure_2.png

图 2 定义了每个服务器在任何状态下对每个传入的 RPC 应该做什么，以及何时应该发生某些其他事情（例如何时可以安全地应用日志中的条目）。起初，您可能倾向于将图 2 视为一种非正式指南；你读过一次，然后开始编写一个大致遵循它所说的实现的实现。这样做，您将快速启动并运行大部分工作的 Raft 实现，然后问题就开始了。

事实上，图 2 非常精确，它所做的每一个陈述都应该按照规范的术语来处理，就像**必须**，而不是**应该**。例如，当您收到 `AppendEntries` 或 `RequestVote` RPC 时，您可能会合理地重置对等点的选举计时器，因为两者都表明其他对等点要么认为它是领导者，要么正试图成为领导者。直观地说，这意味着我们不应该干涉。但是，如果您仔细阅读图 2，它会说：

> If election timeout elapses without receiving `AppendEntries` RPC **`from current leader`** or **`granting`** vote to candidate: convert to candidate.
>
> 如果选举超时过去了，没有从当前领导者那里收到 `AppendEntries RPC`，也没有给候选人投票：转换为候选人。

事实证明，这种区别很重要，因为在某些情况下，前一种实现可能会导致活跃度显着降低。



### 2.1. 细节的重要性

为了使讨论更具体，让我们考虑一个让 6.824 名学生绊倒的例子。Raft 论文在很多地方都提到了心跳(**`heartbeat`**) RPCs。具体来说，领导者会偶尔（每个心跳间隔至少一次）向所有对等方发送 `AppendEntries` RPC，以防止它们开始新的选举。如果领导者没有新条目要发送给特定对等方，则 `AppendEntries` RPC 不包含任何条目，并被视为心跳。

我们的许多学生认为心跳在某种程度上是“特殊的”。当对等点收到心跳时，它应该与非心跳 `AppendEntries` RPC 区别对待。特别是，许多人会在收到心跳时简单地重置他们的选举计时器，然后返回成功，而不执行图 2 中指定的任何检查，这是**非常危险的**。通过接受 RPC，追随者隐含地告诉领导者它们的日志与领导者的日志匹配，直到并包括 `AppendEntries` 参数中包含的 `prevLogIndex`。在收到回复后，领导者可能会（错误地）决定某些条目已被复制到大多数服务器，并开始提交它。

许多人遇到的另一个问题（通常在解决上述问题后立即发生）是，在收到心跳后，他们会在 `prevLogIndex` 之后截断追随者的日志，然后附加任何包含在 `AppendEntries` 参数中的条目。 这也是不正确的。我们可以再次转向图 2：

> *If* an existing entry conflicts with a new one (same index but different terms), delete the existing entry and all that follow it.
>
> 如果现有条目与新条目冲突（索引相同但任期不同），删除现有条目及其后面的所有条目。

这里的如果（`if`）很关键。 如果追随者拥有领导者发送的所有条目，则追随者不得截断其日志。追随者必须保留领导者发送的条目之后的任何元素。这是因为我们可能会从领导者那里收到一个过时的 `AppendEntries` RPC，并且截断日志意味着“收回”我们可能已经告诉领导者我们在日志中的条目。



## 3. 调试 Raft

