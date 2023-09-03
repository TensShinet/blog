---
title: '[SD] Summary: System Design From Chapter 1 to Chapter 4'
date: 2023-09-03 09:21:42
tags: [sd]
---



**TL; DR**



I started to focus on System Design recently. This blog series will record what I learn in a **Q&A format**. By the way, the material is from [Grokking Modern System Design Interview for Engineers & Managers](https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers). In this article, I will summarise what I have learned from Chapter 1 to Chapter 4.



<!--more-->



## Q&A



### How do you allocate time during a system design interview?



The system design interview is an iterative process. As a result, I will develop an overall design that takes about 80 percent of my time and a second iteration for improvements.



### How are SDIs different from other interviews?



Like any other interview, we need to approach the systems design interviews strategically. SDIs are different from the coding interview. There's rarely any coding required in this interview. An SDI takes place at a much higher level of abstraction. We must figure out the requirements and map them onto the computational components and the high-level communication protocols connecting these subsystems. The final answer doesn't matter. What matters is the process and journey a good applicant takes through the interview.



### How do we tackle a design question?



Here are the best practices:



1. Ask the **refining** questions to **solidify** the requirements until we find it can be solved within the limited time.

   We should ensure that we're solving the right problem. Often, it helps to divide the requirements into two groups:

   	1. Requirements that the clients need directly.
   	1. Requirements that are needed indirectly.

2. Handle data

   We must identify and understand data and its characteristics to look for appropriate data storage systems and processing components. Like, we can ask some questions:

   1. What's the size of the data right now?
   2. Is the data read-heavy or write-heavy?

3. Engage the interviewer to ensure that they understand our thought process.

   There are two main discussions with the interviewer: 

   1. Discuss the components

      An example could be the type of database—Will a conventional database work, or should we use a NoSQL database?

   2. Discuss the trade-offs

      We should point out weaknesses in our design to our interviewer and explain why we have yet to tack them. An example could be that our current design can't handle ten times more load, but we don't expect our system to reach that level anytime soon.



### What not to do in an interview?



1. Don't start building without a plan.
2. If we don’t know something, we don’t paper over it, and we don’t pretend to know it.



### How does RPC work?



**RPC** is an interprocess communication protocol widely used in distributed systems. When we make a remote procedure call, the calling environment is paused, and the procedure parameters are sent over the network to the environment where the procedure will be executed. 



When the procedure execution finishes, the results are returned to the calling environment where the excursion restarts as a regular procedure call.



### What are the categories related to consistency?



There are two ends of the consistency spectrum: 



+ Weakest consistency
+ Strongest consistency



In detail, they are **Eventual consistency**, **Casual consistency**, **Sequential consistency**, and **Linearizability consistency**.



### What is the eventual consistency?



Eventual consistency ensures that all replicas will eventually have the same value as the read request, but the returned value isn't meant to be the latest value. However, the value will finally reach its latest state.

The **domain name system** is a highly available system that enables name lookups to a hundred million devices across the Internet. It uses an eventual consistency model and doesn't necessarily return the latest value.



### What is the causal consistency?



Causal consistency works by categorizing operations into dependent and independent operations. Dependent operations are also called causally-related operations. Causal consistency preserves the order of the causally related operations.

The causal consistency model is used in a commenting system. For example, for the replies to a comment on a Facebook post, we want to display comments after it replies. This is because a cause-and-effect relationship exists between a comment and its replies.



### What is the linearizability consistency?



This model ensures that a read request from any replicas will get the latest write value. Once the client acknowledges that the write operation has been performed, other clients can see that value.

Updating an account's password requires linearizability consistency. For example, if I change my bank password and the attacker cannot use a stale password to login in my account.













 













