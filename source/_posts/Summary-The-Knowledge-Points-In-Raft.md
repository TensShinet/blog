---
title: '[Raft] Summary: The Knowledge Points In Raft'
date: 2022-11-24 00:42:17
tags: [raft]
typora-root-url: ./Summary-The-Knowledge-Points-In-Raft
---



**TL; DR**



这是一篇关于 Raft 知识点总结的博客。



主要参考：



+ 论文：https://github.com/ongardie/dissertation/blob/master/stanford.pdf
+ ETCD 的文档：https://github.com/etcd-io/etcd/blob/main/raft/README.md



<!--more-->



## Knowledge Points



### 持久化的状态有哪些，以及为什么



+ log[]
+ currentTerm
+ votedFor



这三个属性都保证：一个 Term 只能给一个 Candiate 投票。



### 为什么不持久化 CommitIndex



如果持久化了上述三个属性，CommitIndex 就不是必须持久化的属性。重启后设置为 0 也没关系。



### 如何保证不 reapply log



持久化 AppliedIndex。



### 如果 Server 的持久化数据丢了怎么办



如果丢了的 Server 数目小于大多数，可以用新的身份重新加入集群。如果大于等于，就可能丢数据。所以必须承认还是有丢数据的可能。



### 被投票要满足哪些条件



### Leader 为什么不能提交之前任期的日志



当之前任期的日志就算被 replica 了大多数份了以后依然有可能被覆盖。



如下图所示，是论文中的具体例子。



![](1.png)

(a) 有 5 个 Server，S1 是 Term=2 的 Leader，现在 5 个 Server 都有 (Term=1, Index=1) 的日志。并且 S2 同步了 S1 的日志 (Term=2, Index=2)。

(b) S1 发生了崩溃，S5 当上了 Term=3 的 Leader（收到 S3，S4 的投票）。并且 S5 写入了 (Term=3, Index=2) 的日志。

(c) S5 还没来得及同步，发生了崩溃。S1 当上了 Term=4 的 Leader（收到 S2，S3 的投票），并且给 S2，S3 同步了 (Term=2, Index=2) 的日志。此时该日志已经被大多数 Replica 了。

(d1) S1 发生了崩溃，S5 当上了 Term=5 的 Leader（收到 S3，S4 的投票），在发生同步的时候把 (Term=2, Index=2) 日志覆盖了。**也就是说，被 Replica 了大多数份了以后依然有可能被覆盖**



也就是说，不能按日志被 Replica 的数目来设置 Commit Index。Leader 只能提交自己任期的日志。



### 系统中有哪些对时间的依赖



虽然任期和日志都是和时间无关的，但是这个系统想要稳定的持续下去，不可避免的依赖真实的系统时间。



**broadcastTime << electionTimeout << MTBF**



+ **<<** 表示：表达式右值要至少大于表达式左值一个数量级。
+ **MTBF** 表示：Mean Time Between Failures。



其中 **broadcastTime** 其实指的是 RPC 的平均传播时间，其中包括请求在网络上的传播以及数据持久化的时间。**broadcastTime** 和 **MTBF** 是底层系统的时间，Raft 需要自己设置 **electionTimeout**。





具体含义如下：



+ **broadcastTime << electionTimeout**



选举的超时时间要至少比传播时间大一个数量级，不然传播还没结束，又开始新的选举了。



+ **electionTimeout << MTBF**



系统崩溃的时间至少要比选举的超时时间大一个数量级，不然选举还没完成，系统又崩溃了。



### Leader Transefer 全过程



### Cluster Membership Changes 全过程



### 为什么单步



### 什么时候可以使用新的 Membership Config



### 为什么加入新的 Node 会有可用性的问题并如何解决



### 删除的 Node 是 Leader 怎么办



### 为什么引入 Pre-Vote



### 什么情况下 Pre-Vote 没有作用





### 如何实现 Replace Server

























