---
title: Database Transaction
categories:
 - Database
tags: [Database Transaction, MySQL, Cassandra]
---

## Introduction
In the context of databases and data storage systems, a transaction is any operation that is treated as a single unit of work, which either completes fully or does not complete at all, and leaves the storage system in a consistent state. 

## ACID 
ACID is an acronym that refers to the set of 4 key properties that define a transaction: Atomicity, Consistency, Isolation, and Durability. 
### Atomicity 
Each statement in a transaction (to read, write, update or delete data) is treated as a single unit. Either the entire statement is executed, or none of it is executed. This property prevents data loss and corruption from occurring if, for example, if your streaming data source fails mid-stream.
### Consistency
Ensures that transactions only make changes to tables in predefined, predictable ways. Transactional consistency ensures that corruption or errors in your data do not create unintended consequences for the integrity of your table.
### Isolation
When multiple users are reading and writing from the same table all at once, isolation of their transactions ensures that the concurrent transactions don't interfere with or affect one another. Each request can occur as though they were occurring one by one, even though they're actually occurring simultaneously.

| Read uncommitted | Read committed | Repeatable read | Serializable
| ------ | ------ | ------ | ------ |
| dirty read, unrepeatable read, phantom read | unrepeatable read, phantom read | phantom read | |

#### Dirty Read
Example: The boss wants to pay the programmer's salary, and the programmer's salary is 36,000 yuan/month. But the boss accidentally pressed the wrong number when the salary, according to 39,000 / month, the money has been hit to the programmer's account, but the affairs have not yet submitted, at this time, the programmer to check his salary this month, found that more than usual 3,000 yuan, thought that the salary was very happy. But the boss found the wrong in time, immediately rolled back the transaction that was almost submitted, changed the number to 36,000 and submitted again.
Analysis: The actual programmer's salary this month is still 36,000, but the programmer sees 39,000. He saw the data before the boss committed the transaction. This is dirty reading.

#### Non-repeatable Read
Example: The programmer takes the credit card to enjoy life (the card is only 36,000, of course), when he pays the bill (the programmer transaction opens), the charging system detects in advance that his card has 36,000, at this time!! The programmer's wife would transfer all the money out for household use and submit it. When the charging system is ready to deduct, the amount of the card is tested again and found that there is no money (of course, the second test amount must wait for the wife to transfer the amount of the transaction is submitted). Programmers will be very depressed, when the card is rich...
Analysis: This is the read commit, if there is a transaction to UPDATE the data (UPDATE) operation, the read operation transaction must wait for the update operation transaction to be submitted before reading the data, which can solve the dirty read problem. But in this case, two identical queries within the scope of a transaction return different data, which is a non-repeatable read.

#### Phantom Read
Example: The programmer went to consume one day, spent 2,000 yuan, and then his wife went to check his consumption record today (full table scanning FTS, wife affairs opened), saw that it was indeed spent 2,000 yuan, at this time, the programmer spent 10,000 to buy a computer, that is, added a new INSERT consumption record, and submitted. When the wife printed the programmer's consumption record list (wife affairs submission), found that it spent 12,000 yuan, it seemed to appear hallucination, which is fantasy reading.
Analysis: The repeated read can solve the unrepeatable read problem. Writing here, it should be understood that the non-repeatable read corresponds to a modification, that is, an UPDATE operation. But there may also be phantom reading problems. Because the phantom read problem corresponds to the INSERT operation, not the UPDATE operation.

### Durability 
Ensures that changes to your data made by successfully executed transactions will be saved, even in the event of system failure.

## Methodologies
### transaction ID
For each transaction, it must be monotone increased, some methds can be used:
* server timestamp
* clinet timestamp
* snowflake ID

### WAL
Write Ahead Log (WAL) is a common method used in database systems to ensure the atomicity and persistence of data operations.

In computer science, "Write-ahead logging" (WAL) is a set of techniques used to provide atomicity and durability (two of the ACID properties) in relational database systems. On systems that use WAL, all changes are written to the log file before being committed.

The log file usually contains redo and undo information. The purpose of this can be illustrated by an example. Suppose a program is in the process of performing some operation when the machine loses power. Upon restart, the program may need to know whether the operation performed at the time was successful, partially successful, or failed. If WAL is used, the program can check the log file and compare what was planned to be done in case of a sudden power failure with what was actually done. On the basis of this comparison, the program can decide whether to undo the operation, continue to complete the operation, or leave it as it is.

### Lock
To implement different level of isolation, database needs to use lock to avoid concurrent conflict. There are two types of lock on different lock level, one is record lock and another is gap lock. The first one is to lock a specific record like a row or a document. Besides, some databases have intent locks to indicate an intent to read or write a resource using a finer granularity lock.

Besides, write lock and read lock are two main locks, which can solve write-read and write-write conflict.

### MVCC
To implement different level of isolation, database needs to use MVCC to avoid concurrent conflict. Multiversion concurrency control (MVCC) is a database optimization technique that creates duplicate copies of records so that data can be safely read and updated at the same time. With MVCC, DBMS reads and writes don’t block each other.

MVCC also requires garbage collection, otherwise too much old version data will take up unnecessary storage space.

MVCC is mainly used for write-read conflict, to solve write-write conflict, the MVCC methodology need to do re-try or throw exception.

### LWW
Latest write win is a pretty brust force way without using locking or MVCC to handle concurrency of write conflict, which has been used on early-aged NoSQL database.

### 2PC
cooperative termination protocol

## Applications
### MySQL
It is worth mentioning that the default transaction isolation level of most databases is Read committed, such as Sql Server and Oracle. The default isolation level for Mysql is Repeatable read.

Binlog is the logical operation log of MySQL, which is widely used in replication and recovery. Before MySQL 5.1, Statement is the default Binlog format, which records the SQL requests received by the system in sequence. In 5.1 and later, MySQL provides two Binlog formats, Row and Mixed.

Starting with MySQL 5.1, if statement level Binlog is turned on, RC and Read-Uncommited isolation levels are not supported. To use the RC isolation level, you must use the Mixed or Row format.

The point of non-repeatable reading is to modify:
The same condition of select, you read the data, read again to find that the value is not the same
The focus of phantom reading is to add or delete:
Under the same condition, the number of records read by the first and second select is different
From the result point of view, both are inconsistent results for multiple reads. But if you look at it from an implementation point of view, they are quite different:
for the former, under RC, only the records that meet the conditions need to be locked to avoid being modified by other transactions, that is, select for update, select in share mode; Using MVCC to achieve repeatable read under RR isolation;
For the latter, the gap between the records that meet the condition and all of them is locked, that is, the gap lock is required.

MySQL's default isolation level RR uses Gap-Lock for phantom reads and Record-Lock for dirty and repeatable reads. Therefore, the RR level is implemented by Next-Key Lock(Gap-Lock + Record-Lock). The MVCC mechanism is introduced by MySQL to achieve consistent non-locked read and improve the read and write efficiency.
MySQL uses MVCC to implement RR and RC. The read in MySQL is divided by snapshot read and current read.

### Cassandra
Cassandra only support row-level AID (with C since it doesn't support foreign key).

### MongoDB


### Spanner






