---
title: '[MySQL] A Simple Summary: REPEATED READ Isolation In MySQL'
date: 2025-03-30 22:16:06
tags: [mysql]
---

**TL; DR**

REPEATED READ is the default isolation level in MySQL. It solves the following:

+ **Dirty reads**: reads uncommitted data (**READ COMMITTED has solved this**)
+ **Nonrepeatable reads**: reads different data in the same transaction for the same row even UPDATEs are not executed
+ **Phantom reads**: reads additional rows or missing rows in the same transaction, even INSERTs or DELETEs are not executed

This blog post mainly focuses on how nonrepeatable reads and phantom reads are solved by REPEATED READ isolation level and the unsolvable anomalies in REPEATED READ.

<!--more-->

## Nonrepeatable Reads

Consistent nonlocking reads and locking reads are two types of reads in the REPEATED READ isolation level, and both solve nonrepeatable reads. 

### Consistent Nonlocking Reads

+ `SELECT`

It creates a consistent snapshot provided by the MVCC mechanism to read data. When a transaction begins and starts to read some rows, it will have an unchangeable version, which can be used to check if the rows are visible to the transaction. The details are as follows(It may not be accurate):

+ When a transaction starts, it will have a transaction ID.
+ When a transaction plans to read, it will keep all active transaction IDs.
+ When a transaction reads a row, it will check if the row's transaction ID is less than or equal to this transaction's and does not exist in the active transaction IDs. If it is, the row is visible to the transaction. Otherwise, the row is invisible to the transaction.



### Locking Reads

+ `SELECT ... LOCK IN SHARE MODE`

It sets a shared mode lock on **any rows that are read,** even if some rows cannot meet the WHERE condition. Other sessions can read the rows, but cannot modify them until your transaction commits. If any of these rows were changed by another transaction that has not yet committed, your query waits until that transaction ends and then uses the latest values[1].

+ `SELECT ... FOR UPDATE`

For index records the search encounters, locks the rows and any associated index entries, the same as if you issued an UPDATE statement for those rows. Other transactions are blocked from updating those rows, from doing SELECT ... FOR SHARE, or from reading the data in certain transaction isolation levels. **Consistent reads ignore any locks set on the records that exist in the read view. (Old versions of a record cannot be locked; they are reconstructed by applying undo logs on an in-memory copy of the record.)**[1]



### Pros and Cons

|                             | Pros | Cons |
| --------------------------- | ---- | ---- |
| Consistent Nonlocking Reads | Never blocks any transaction    | Lost Update(Use read data to update) |
| Locking Reads               | Deadlock | No Lost |



## Phantom Reads

Consistent nonlocking reads and locking reads also solve phantom reads.

### Consistent Nonlocking Reads

+ `SELECT`

It uses the same mechanism as nonrepeatable reads to solve phantom reads.

### Locking Reads

+ `SELECT ... LOCK IN SHARE MODE` and `SELECT ... FOR UPDATE`

It uses the **GAP LOCK** to solve phantom reads. When a transaction reads a range of rows, it locks the gap between the rows. If another transaction inserts a row in the gap, it will be blocked until the first transaction commits or rolls back.



## Other Confusing Anomalies

Even though the REPEATED READ isolation level solves nonrepeatable reads and phantom reads, it still has some confusing anomalies:



### Example

+ Create a table:

```sql
CREATE TABLE t1 (id INT PRIMARY KEY, c INT) ENGINE = InnoDB;
INSERT INTO t1 VALUES (1, 10), (2, 20), (3, 30);
```

+ Transaction 1:

```sql
START TRANSACTION;
SELECT * FROM t1;
+----+------+
| id | c    |
+----+------+
|  1 |   10 |
|  2 |   20 |
|  3 |   30 |
+----+------+
```

+ Transaction 2:

```sql
START TRANSACTION;
UPDATE t1 SET c = c + 1 WHERE id = 2;
```

+ Transaction 1:

```sql
SELECT * FROM t1;
+----+------+
| id | c    |
+----+------+
|  1 |   10 |
|  2 |   20 |
|  3 |   30 |
+----+------+
```

+ Transaction 2:

```sql
COMMIT;
```

+ Transaction 1:

```sql
UPDATE t1 SET c = c + 1 WHERE id = 2;
SELECT * FROM t1;
+----+------+
| id | c    |
+----+------+
|  1 |   10 |
|  2 |   22 |
|  3 |   30 |
+----+------+
```

**The anomaly is the c value of id 2 is 22, not 21, in Transaction 1.**



### Explanation

The root cause is the mixing of consistent nonlocking reads and locking reads. `SELECT * FROM t1;` is a consistent nonlocking read, and `UPDATE t1 SET c = c + 1 WHERE id = 2;` is a locking read. **The locking read updates the row via the committed version(21), not the uncommitted version(20). So the c value of id 2 is 22, not 21.**

Therefore, mixing consistent nonlocking and locking reads may cause some confusing anomalies. Users should be as consistent as possible in the same transaction.



## Useful SQLs

+ Check the locks:

```sql
SELECT * FROM performance_schema.data_locks;
```



## References

+ [1]: [17.7.2.4 Locking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking-reads.html)



