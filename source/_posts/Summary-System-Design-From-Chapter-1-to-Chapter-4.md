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
   2. Requirements that are needed indirectly.

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



When the procedure execution finishes, the results are returned to the calling environment, where the excursion restarts as a regular procedure call.



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

Updating an account's password requires linearizability consistency. For example, if I change my bank password, the attacker cannot use a stale password to log in to my account.



### What are the categories related to failure?



+ Fail-stop

+ Crash

+ Omission Failures

  In **omission failures**, the node fails to send or receive messages. There are two types of omission failures: **send omission failure** and **receive omission failure**.

+ Temporal Failures

  The node generates correct results in temporal failures, but it is too late to be useful.

+ Byzantine Failures

  In **byzantine failures**, the node exhibits **arbitrary** behavior, like transmitting random messages at arbitrary times.



### How do we measure the availability?



The availability in percent is `(Total Time - Amount of Time Service was Down/Total Time) * 100`.



| Availability Percentages versus Service Downtime |                       |                        |                       |
| ------------------------------------------------ | --------------------- | ---------------------- | --------------------- |
| **Availability %**                               | **Downtime per Year** | **Downtime per Month** | **Downtime per Week** |
| 90% (1 nine)                                     | 36.5 days             | 72 hours               | 16.8 hours            |
| 99% (2 nines)                                    | 3.65 days             | 7.20 hours             | 1.68 hours            |
| 99.5% (2 nines)                                  | 1.83 days             | 3.60 hours             | 50.4 minutes          |
| 99.9% (3 nines)                                  | 8.76 hours            | 43.8 minutes           | 10.1 minutes          |
| 99.99% (4 nines)                                 | 52.56 minutes         | 4.32 minutes           | 1.01 minutes          |
| 99.999% (5 nines)                                | 5.26 minutes          | 25.9 seconds           | 6.05 seconds          |
| 99.9999% (6 nines)                               | 31.5 seconds          | 2.59 seconds           | 0.605 seconds         |
| 99.99999% (7 nines)                              | 3.15 seconds          | 0.259 seconds          | 0.0605 seconds        |



### How do we measure the reliability?



We often use the **mean time between failures(MTBF)** and the **mean time to repair(MTTR)** to measure reliability.



+ MTBF: `(Total Elapsed Time - Sum of Downtime)/Total Number of Failures`.
+ MTTR: `Total Maintenance Time/Total Number of Repairs`.





### What are the scalability approaches?



+ Vertical scalability—scaling up
+ Horizontal scalability—scaling out



 

### How do we measure the maintainability?



We use the **mean time to repair(MTTR)** to measure maintainability.



+ MTTR: `Total Maintenance Time/Total Number of Repairs`











