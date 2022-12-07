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



+ 第一种情况：Receiver 的任期等于 Sender 的任期，并且 Receiver 给 Sender 投过票，并且 Receiver 的 Log 小于等于 Sender 的 Log。
+ 第二种情况：Receiver 的任期小于 Sender 的任期，并且 Receiver 在此任期没有投过票，并且  Receiver 的 Log 小于等于 Sender 的 Log。



日志判断大小的依据：



+ Log 的 Term 大，日志大
+ Log Term 相同的情况下，Index 越高越大



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



虽然任期和日志都是 和时间无关的，但是这个系统想要稳定的持续下去，不可避免的依赖真实的系统时间。



**broadcastTime << electionTimeout << MTBF**



+ **<<** 表示：表达式右值要至少大于表达式左值一个数量级。
+ **MTBF** 表示：Mean Time Between Failures。



其中 **broadcastTime** 其实指的是 RPC 的平均传播时间，其中包括请求在网络上的传播以及数据持久化的时间。**broadcastTime** 和 **MTBF** 是底层系统的时间，Raft 需要自己设置 **electionTimeout**。



具体含义如下：



+ **broadcast time << electionTimeout**



选举的超时时间要至少比传播时间大一个数量级，不然传播还没结束，又开始新的选举了。



+ **electionTimeout << MTBF**



系统崩溃的时间至少要比选举的超时时间大一个数量级，不然选举还没完成，系统又崩溃了。



### Leader Transefer 全过程



Leader Server 不可避免的可能需要将 Leader 权限主动转移到其他 Server。所以在协议中涉及 Leader Transefer    。具体步骤如下：



+ Leader Server 接受到 Leader Transefer 以后，不再接受后面的 Client 的请求。
+ 向目标 Server 发送日志，直到目标 Server 和自己的日志一样新。
+ 给目标 Server 发送 Timeout 的请求。



这样目标 Server 就会立即选举超时，立即成为 Candidate，拥有比 Leader Server 更高的 Term。Leader Server 接受到目标 Server 的 RequestVote 的时候就会退化成 Follower。这样就完成了 Leader Transefer。



当然目标 Server 也可能挂掉，所以为了保持可用性（不可用的时间 <= 选举时间），当 Leader Server 在一个选举时间内，发现没有完成 Leader Transefer，会停掉 Leader Transefer，再次接受 Client 的请求。



### Cluster Membership Changes 全过程



分布式系统中成员的变更是不可避免的。如何在成员变更的时候保证顺序一致性，又是一个新的挑战。在 Raft 中，推荐使用**单步变更**。



**单步变更**指的是每次只添加或者删除一个 Server。使用**单步变更**的原因在于：**不论是添加还是删除，旧集群的 majority 和新集群的 majority 一定会发生重叠。**这样就保证了 Cluster Membership Changes 的安全性。



+ Leader Server 收到 Cluster Membership Changes Request
+ 等待上一次  Cluster Membership Changes 结束
+ 将 Cluster Membership Changes Append 到日志中
+ **更改当前 Leader 的 Membership（不需要 commit）**
+ 将日志同步给其他 Server
+ 当其他 Server Append 到 log 后，修改自己的 Membership
+ 当这个 Cluster Membership Changes 提交以后，本次 Membership Changes 结束



### 为什么加入新的 Node 会有可用性的问题并如何解决



以下两种情况都会引入集群可用性问题：



![image-20221205134746973](/image-20221205134746973.png)



+ 当加入新的 Node S4 以后，由于 S4 没有任何日志，如果 S3 挂掉了，集群中的请求没办法很快达到大多数，这样就新增了可用性的问题



![image-20221205135407856](/image-20221205135407856.png)



+ 当快速加入新的 Server 以后，由于 S4，S5，S6 没有日志，集群中的请求没办法很快达到大多数，这样同样新增了可用性的问题



解决办法如下：



加入新 Node 的时候不直接参与投票，而是以一种 **Leaner** 的身份加入集群，然后在**合适的时候**（日志同步的差不多了）再转换身份可以参与投票。



选择**合适的时候**的方法如下：



![image-20221205161635925](/image-20221205161635925.png)



如图所示，日志同步其实是多个 round 同步的过程。原因是在同步的过程中也会产生新的日志。只要上一个 round 的同步时间小于选举时间就是一个把新 Node 加入集群参与投票的合适时间。



### 删除的 Node 是 Leader 怎么办



由于删除的 Node 是 Leader，所以会导致不可用一段时间。Raft 中提出了这样的删除 Leader 的一种方法：



+ Leader 收到删除自己的 Request 以后，还做 Leader 同步一段时间
+ 并且在计算 commit 的时候不算自己的票
+ 一旦 commit，自己 step down



举一个极端的例子来分析这样是可行的。假设集群中只存在两个 Server：S1 和 S2，并且 S1 是 Leader。现在要删除 S1。流程如下：



+ S1 收到删除自己的 Request 以后，从成员中将自己删除，但是还做 Leader 给 S2 同步
+ S2 收到 S1 的请求以后，从成员中将 S1 删除，发送消息给 S1
+ S1 收到 S2 的 Response 以后，此时 1 为大多数，满足大多数，提交，S1 step down
+ S2 没收到 S1 的 Heartbeat，S2 超时，自己做 Leader



### 为什么引入 Pre-Vote



当删除的 Server 不是 Leader 的时候，那个 Server 将不再收到来自 Leader 的心跳。所以会自增 Term 给其他的 Server 发送 RequestVote。导致 Leader 收到比自己大的 Term，变成 Follower。这样会影响可用性。



解决这种情况有一个朴素的方案：Pre-Vote。



就是在发起 RequestVote 之前，先预投票一轮，看自己能不能当上 Leader（任期大不大，日志够不够新），如果不能也就没有必要发送 RequestVote 了。



### 什么情况下 Pre-Vote 没有作用



### 为什么需要 Snapshot



### Snapshot 要持久化哪些状态



### 什么时候 Snapshot



### 如何减轻 Snapshot 对 Client Performance 的影响



1. 使用多设备
2. the minority of snapshotting servers







### 有哪些性能可以优化的地方





## MultiRaft 方案总结





## ETCD/Raft 优化总结



