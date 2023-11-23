---
title: A Review of Database Index
categories:
 - Database
tags: [Database Index]
---

## Introduction
Index is a data structure that helps the database system obtain data efficiently. It improves the data retrieval process in the database at the cost of adding additional write operations and storage space for maintaining the index data structure. To search an entry, database would search the index for the entry first, and index is pointing to real physical location of the entry.
Query pattern
* equals query
* range query

## B-Tree
For most of the relational databases like MySQL, they use B+ Tree for their index. In general, the index itself is also very large and cannot be all stored in memory, so the index is often stored on disk in the form of index files. In this way, the index search process will produce disk I/O consumption, relative to memory access, I/O access consumption is several orders of magnitude higher, so the most important indicator to evaluate a data structure as an index is the gradual complexity of the number of disk I/O operations in the search process. In other words, the index is structured to minimize the number of disk I/O accesses during lookup.

### MySQL MyISAM
unclustered indexes

### MySQL InnoDB
clustered indexes

Minimum storage unit
![vm_directory @8x]({{ "/assets/images/_post/d2.svg" | absolute_url }})

We can see the page size on InnoDB is 16 k, and the size of a B-Tree node is chosen according to the page size. Because a whole page of data would read even if only reading a few bits.



data BTree node
index BTree node


**Note**  
For performance reasons, computers usually access data in bytes, such as 4KB. In different scenarios, there are different names: disk data is generally called a block, and memory data is generally called a page. Both scenario databases are covered, so the terms can be mixed.


## LSM



## Invert Index




