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



这篇文章以及随附的 [Instructors' Guide to Raft]() 文章记录了我们使用 Raft 的旅程，希望对 Raft 协议的实现者和试图更好地理解 Raft 内部结构的学生有用。如果您正在寻找 Paxos 与 Raft 的比较，或者想要对 Raft 进行更多的教学分析，您应该阅读 Instructors' Guide。这篇文章的底部包含 6.824 名学生常见的问题列表，以及这些问题的答案。如果您遇到本文主要内容中未列出的问题，请查看 [Q&A](https://thesquareplanet.com/blog/raft-qa/)。这篇文章很长，但它提出的所有观点都是很多 6.824 名学生（和助教）遇到的实际问题。这篇文章值得一读。





