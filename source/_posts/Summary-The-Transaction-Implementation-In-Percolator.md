---
title: '[Percolator] Summary: The Transaction Implementation In Percolator'
date: 2022-12-18 15:59:41
tags: [percolator, transaction]
---



**TL; DR**



This is a summary blog about **Percolator**. I only write down the **Transaction Implementation** part in this blog. Because I only understand this part of that paper.



According to that paper, I will introduce how Percolator implements **Snapshot-Isolation transactions** via BigTable. Following is my roadmap:



+ Background: Introduce BigTable and Snapshot Isolation.
+ Implementation: Use a c++ code from that paper to explain how Percolator implements Snapshot Isolation via BigTable.
+ Example: Use a concrete example from that paper to introduce this algorithm's workflow.
+ Safety Argument: Discuss what will happen if a Percolator client crashes and how to recover.
+ Interview Questions: Summarize the frequent interview questions about Percolator Transaction.





The Paper Address:



+ https://research.google/pubs/pub36726/



<!--more-->



## Background



### BigTable



**BigTable** presents a multi-dimensional sorted map to users: keys are (row, column, timestamp) tuples. For Percolator, BigTable offered two essential functions:



+ Reliable KV Storage
+ Single Row Transaction



### Snapshot Isolation



**Snapshot Isolation** is a database isolation level, which came up in the paper: A Critique of ANSI SQL Isolation Levels. Its core idea is straightforward: each transaction reads data from a snapshot of the committed data as of the time the transaction started, called its Start-Timestamp. In other words, each **transaction cannot read the data committed after its Start Timestamp**. With this method, a transaction cannot read dirty data. 



In addition, Snapshot Isolation also proposes **a way to prevent lost updates**. For example, when transaction T1 is ready to commit, it gets a **Commit Timestamp** larger than any existing Start Timestamp or Commit Timestamp. The transaction successfully commits only if no other transaction T2 with a Commit Timestamp in T1's execution interval [Start Timestamp, Commit Timestamp] wrote data that T1 also wrote. Otherwise, T1 will abort. This feature, called **First-committer-wins**, prevents lost updates.



However, Snapshot Isolation is not a serializable isolation level. It cannot solve a **Write Skew** issue that never happens in serializable transactions. For example, if we have a table in which Key a equals six and Key b equals five, and we have a constraint: a >= b.



| Key  | Value |
| ---- | ----- |
| a    | 6     |
| b    | 5     |



There are two concurrent transactions, T1 and T2. They both read Key a equals six, and Key b equals five. T1 reads Key a equals six, subtracting one from a, and gets five. T1 still meets the constraint: a >= b, and T1 commits. At the same time, T2 reads Key b equals five, adding one to b, and gets six. T2 also meets the constraint: a >= b, and T2 commits. But now, this table is changed and not meets the constraint: a >= b, which never happens in serializable transactions(T1 or T2 will abort because it doesn't meet constraint: a >= b). 



| Key  | Value |
| ---- | ----- |
| a    | 5     |
| b    | 6     |






Summarize Snapshot Isolation:



+ Each transaction has an execution interval: [Start Timestamp, Commit Timestamp] and only reads the data committed before its Start Timestamp.
+ All timestamps grow monotonously.
+ Transaction T1 will successfully commit only if no other transaction T2 with a Commit Timestamp in T1's execution interval wrote data that T1 also wrote.
+ Snapshot Isolation can't solve a **Write Skew** issue.



More details:



+ https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf





The next part will use a c++ code to explain how Percolator implements Snapshot Isolation via BigTable.



## Implementation






## Example



## Safety Argument



## Interview Questions



