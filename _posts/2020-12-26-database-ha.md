---
title: Database HA
categories:
 - Database
tags: [Database HA, MySQL, Cassandra, Spanner]
---

## Introduction
The real difficulty and pain point of system high availability design is not the design of stateless service, but the high availability design of database, because:

* data persistence in the database, is a stateful service;
* the capacity of the database is large, and the Failover duration is longer than that of stateless services.
* some systems, such as databases in financial scenarios, require that data cannot be lost at all, which makes it difficult to achieve high availability.
In fact, from the perspective of architecture, the database high availability itself is also the business high availability.

## Methodologies

### Replication
The first idea to improve availability is adding some backup/slave ndoes, which replicate the data in the database (master). There are couples of methods:
* asynchronous replication: cannot assue data integrity and consistency, since the replication lag.
* synchronous replication: need to wait for the entire replication process to complete, cannot stand for any instance failure.
* semi-synchronous replication: falls between asynchronous and fully synchronous replication.
* quorum replication: like raft

#### Failover
1. detect failure
2. leader selection


### Horizontal Scale
Before we mention horizontal Scale, let us first take a look at vertical scale, often known as “scaling up,” is the process of increasing the power of an existing system, such as the CPU or RAM, to meet the rising demands. Because there is no need to alter the logic, vertical scaling is simpler. Instead, you are only executing the same code on machines with more capacity.

Horizontal scale is also called scale-out, it divides the whole database into multiple shardings/partitions.
#### Routing
When a database has multiple shardings, it will need a way to know which shard has which data. Consistent hashing is a special hashing algorithm. After using the consistent hashing algorithm, a change in the number of slots (size) in the hash table requires on average only a remapping of K/n keywords, where K is the number of keywords and n is the number of slots. However, in a traditional hash table, adding or removing a slot requires remapping almost all keywords.

* Client Routing
* Proxy Routing
* Server Routing



## Applications
### MySQL

### Cassandra

### MongoDB

### Spanner






