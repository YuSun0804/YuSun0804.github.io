---
title: Database HA
categories:
 - Database
tags: [Database HA, MySQL, Cassandra, Spanner]
---

## Introduction
The real difficulty and pain point of system high availability design is not the design of stateless service, but the high availability design of database, because:

* data persistence in the database, is a stateful service;
* the capacity of the database is large, and the failover duration is longer than that of stateless services.
* some systems, such as databases in financial scenarios, require that data cannot be lost at all, which makes it difficult to achieve high availability.
In fact, from the perspective of architecture, the database high availability itself is also the business high availability.

The requirements of HA:
* If the database is down or unexpectedly interrupted, the availability of the database can be restored as soon as possible to minimize the downtime and ensure that services are not interrupted due to database faults.
* The data on the non-primary node used for backup and read-only copy must be consistent with the data on the primary node in real time or in the end.
* When services are switched over, the contents of the database before and after the switchover must be the same. Services will not be affected due to data loss or data inconsistency.

## Methodologies
It is not easy to implement HA both theoretically and industrially. For different databases, they could use different way to implement HA, but they usually follow similar principles. Let us take a look at simple case first with only DB and multiple application server. All requests would be on the the same DB.
![vm_directory @1x]({{ "/assets/images/post/database-ha/ha1.drawio.svg" | absolute_url }})


### Replication
The first idea to improve availability is adding some replication ndoes, which replicate the data in the database (master). Then both replication and primary node can accept requests.

![vm_directory @1x]({{ "/assets/images/post/database-ha/replication.drawio.svg" | absolute_url }})


#### Data Synchronization
* asynchronous replication: cannot assue data integrity and consistency, since the replication lag.
* synchronous replication: need to wait for the entire replication process to complete, cannot stand for any instance failure.
* semi-synchronous replication: falls between asynchronous and fully synchronous replication. The primary database needs to wait for at least one slave database to receive and write ACK messages.
* quorum replication: primary and replication nodes are together to make a replication group, and must be submitted by a majority of nodes in the group (N / 2 + 1).

#### Failover
To implement fast failover, database first needs to detect failure. In centralized system, there is always a controller node to monitor health status of the primary and replication nodes, if no contronller node, then all nodes together to vote which one is down. While in decentralized system (p2p) since no controller node, besides it does not distinguish primary and replication nodes, all nodes are the same, they use some consensus algorithm to mark which one is down.

Then for new leader selection, centralized system is easy, the controller node would pick a node with more latest data as new master node. For decentralized system, do not need to select new leader.


### Horizontal Scale
Before we mention horizontal Scale, let us first take a look at vertical scale, often known as “scaling up,” is the process of increasing the power of an existing system, such as the CPU or RAM, to meet the rising demands. Because there is no need to alter the logic, vertical scaling is simpler. Instead, you are only executing the same code on machines with more capacity.

Horizontal scale is also called scale-out, it divides the whole database into multiple shardings/partitions.
![vm_directory @1x]({{ "/assets/images/post/database-ha/scale.drawio.svg" | absolute_url }})

#### Sharding
When a database has multiple shardings, it will need a way to know which shard has which data. Consistent hashing is a special hashing algorithm. After using the consistent hashing algorithm, a change in the number of slots (size) in the hash table requires on average only a remapping of K/n keywords, where K is the number of keywords and n is the number of slots. However, in a traditional hash table, adding or removing a slot requires remapping almost all keywords.

#### Routing
After we split a whole data into multiple partitions, we need to a place to store which partition is on which node. In centralized system, the controller node can also be metadata node, which store all partition information. Then it can put routing logic in client-side, server-side (not usually) or even add a new layer in between. For decentralized node, each node shares the metadata information by gossip protocol, it always use server-side routing.


## Applications
### MySQL
总结来说，MySQL 的主从复制：异步单线程。
Master上 1 个IO线程，负责向Slave传输binary log（binlog）
Slave上 2 个线程：IO 线程和执行SQL的线程，其中：
    IO线程：将获取的日志信息，追加到relay log上；
    执行SQL的线程：检测到relay log中内容有更新，则在Slave上执行sql；
特别说明：MySQL 5.6.3 开始支持「多线程的主从复制」，一个数据库一个线程，多个数据库可多个线程。

- 同步复制
如果要满足主从架构的强一致性，采取「同步复制」的 2PC 策略即可：
第一阶段：Master 收到 Client 的写入数据请求，在本地写入数据；
第二阶段：Master 收到 Slave 写入成功的消息，再向 Client返回数据写入成功；
主流数据库均支持这种完全的同步模式，MySQL的Semi-sync功能（从MySQL 5.6开始官方支持），就是基于这种原理。
「同步复制」对数据库的写性能影响很大，适用场景：
银行等严格要求强一致性的应用，对于写入延迟一般没什么要求（延迟几个小时都可以接受，数据不出错就行）。
- 异步复制
异步复制：Master 数据写入成功后，Slave 上异步进行数据写入，只要保证数据最终一致性即可。

- 基于语句的复制：实际上是把主库上执行的SQL语句在从库上重放一遍，因此效率高，占用带宽小，但不如基于行的复制精确，对于不确定性的语句（例如包含时间函数的语句）会有问题。另外这种复制是串行的，为了保证串行执行，需要加更多的锁。
- 基于行的复制：此时二进制日志记录的是数据本身，这无疑会增加网络带宽消耗和I/O线程负载，优点是从库无需sql语句重放，但无法判断执行了哪些SQL语句
- 混合模式，即上面两种方式的组合
mysql默认基于语句复制
### Cassandra

### MongoDB

### Spanner






