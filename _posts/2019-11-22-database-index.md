---
title: Database Index
categories:
 - Database
tags: [Database Index, MySQL, Cassandra]
---

## Introduction
An index is a data structure designed to enhance the efficiency of data retrieval in a database system. While it optimizes the data retrieval process, it does so at the expense of introducing extra write operations and requiring additional storage space to maintain the index data structure. There are several methods to implement indexes in various database systems, and different data structures can also be employed. This article will concentrate on exploring index methodologies and their applications.


## Methodologies
This section primarily focuses on the data structures of various indexing methods, specifically explaining how indexes are stored in a file system or memory. An index usually establishes a connection to the original data. It may store the original data alongside the index in various ways or use a pointer to indicate the data's location. Consequently, by understanding the organization of the index, we can also discern how the original data is structured.

### Hash
When discussing search in general, two types of search scenarios commonly arise: point search and range search. A Hash index excels at point search; if all indexes reside in memory, it facilitates random addition, deletion, modification, and search, with a time complexity for reading and writing of `O(1)`. However, it is not well-suited for range search due to its lack of order.

### Binary Search Tree
The next data structure to consider is the binary search tree (BST), which imposes an implicit order among all its nodes. The definition of a BST includes the following properties:
* The left subtree of a node contains only nodes with values lesser than the node’s value.
* The right subtree of a node contains only nodes with values greater than the node’s value.
* The left and right subtree each must also be a binary search tree.

The complexity analysis of a BST reveals that, on average, the operations of insert, delete, and search take O(logn) for n nodes. However, in the worst case, these operations can degrade to that of a singly linked list, resulting in a time complexity of O(n). To enhance search performance, it is desirable to obtain a binary search tree with a lower height.

AVL trees and red-black trees are two types of balanced BSTs designed to address this issue. They both introduce a level of balance, though at the cost of slightly slower insert and delete operations, aiming to achieve better search performance. Among them, red-black trees strive for a more balanced performance across all operations, making them widely utilized in in-memory search applications, with ongoing efforts for further enhancements.

### B-Tree, B+ Tree
But if we get more data which cannot fit all in memory, then we need to read data from disk, things are chagned. Reading from disk is much slower than from memory. B-tree is a balanced multi-way search tree, the biggest difference between B-tree and red-black tree is that B-tree nodes can have multiple children, from a few to several thousand.
* It is perfectly balanced: every leaf node is at the same depth.
* Every node, except perhaps the root, is at least half-full, i.e. contains M/2 or more values (of course, it cannot contain more than M-1 values). The root may have any number of values (1 to M-1).

B + Tree is a variation of the B-tree data structure, it has following specialities:
* leaf nodes store all key values, so some of the key values of the leaf nodes also appear in the internal nodes. (B Tree doesn't duplicated key values at leaf nodes)
* only leaf nodes store data values/pointers. (in B Tree, data values/pointers store with key together, maybe at non-leaf nodes)
* the leaf nodes are linked to providing ordered access to the records, which can improve range query. 

![vm_directory @1x]({{ "/assets/images/post/database-index/btree.drawio.svg" | absolute_url }})

For most of the relational databases like MySQL, Oracle, they use B+ Tree for their index. In general, the index itself is also very large and cannot be all stored in memory, so the index is often stored on disk in the form of index files. In this way, the index search process will produce disk I/O consumption, relative to memory access, I/O access consumption is several orders of magnitude higher, so the most important indicator to evaluate a data structure as an index is the gradual complexity of the number of disk I/O operations in the search process. In other words, the index is structured to minimize the number of disk I/O accesses during lookup.

There are two ways to originize indexs: clustered index and non-clustered index. 


### LSM Tree
We have already talked about B Tree Index in MySQL, in this article, I am going to talk about the index techonoly in Cassandra, LevelDB and RocksDB: LSM Tree (The Log-Structured Merge-Tree). It needs to split and re-balance the tree when inserting a new node in B Tree, which could cause lots of random IO, so the insert performance could be degraded. LSM tree is a good way to solve the problem with sacrificing read performance. Besides, B Tree stack also update existing data in place, which also causes some random IO.

LST Tree is divided into two parts, one part is stored in memory, the other is stored on the disk, the data retrieval method in memory can use the red and black tree or jump table, these data structure of low time complexity for retrieval, for the structure in disk, it is similar to B Tree. When the memory data reaches a certain threshold, the data is synchronized to a new disk file. At this time, the mode of writing to the disk is sequential, which is why the lsm write performance is high. Since the data is ordered in memory, it is also guaranteed that each small disk file is ordered when it is written to disk. We call these small disk files sstable. Besides, it will merge SSTable periodically: multi-level consolidation.

![lsm tree]({{ "/assets/images/post/database-index/lsm.drawio.svg" | absolute_url }})

From the point of view of the implementation of lsm, it has been able to understand a data writing and retrieval process. So let me summarize this one more time. When lsm is written, it will be written to memory first, and the retrieval of data in memory is generally a more efficient data structure, such as jump table, red-black tree, etc., and the data in memory is ordered. When the memory reaches a certain threshold, it is marked as read-only, new writes are made to the new memory region, and read-only memory writes ordered data to disk level0, forming an sstable file. When the number of level0 SStables exceeds four, they merge with the level1 SSTables. The index ranges of the levels after level0 do not overlap.

When lsm reads data, it also reads data from the memory first. If it cannot read data, it reads data from the disk from the lower layer to the upper layer. If it can't read data, it returns to the last layer. Since the sstable data of each layer after level0 layer is ordered and non-coexisting, after the sstable where the data is located is quickly retrieved, it can quickly determine whether the data is in the layer through binary search, which is true, and Bloom filtering is also used in the sstable to quickly determine the element is not in the sstable. If that layer is not found, continue to the next layer.

As you can see, when reading data, the worst case scenario is to traverse all levels, which is why lsm is suitable for writing more and reading less, and it is best to read the most recent data when reading.

#### Memtable
Memtables stand for Memory Tables. It is an in-memory data structure that holds data before it is flushed to disk as an SSTable.

#### SSTable
SSTables stands for Sorted Strings Table. It is a persistent file format that stores data on disk in a sorted way. 

#### Compaction
There are many different compaction strategies. Size Tiered and Leveled compaction are two of the more popular strategies:
* Size Tiered Compaction:
* Level Compaction


## Applications
### MySQL InnoDB
In InnoDB, it clusters the Primary Key with the data (clustered index). This says that a PK lookup can rapidly get to the entire row. A secondary key lookup, however, has to go through 2 BTree lookups -- one in the secondary key's BTree, then a second in the PK's BTree.
![vm_directory @1x]({{ "/assets/images/post/database-index/mysql-2.drawio.svg" | absolute_url }})


![vm_directory @1x]({{ "/assets/images/post/database-index/mysql-4.drawio.svg" | absolute_url }})

We can see the page size on InnoDB is 16 k, and the size of a B-Tree node is chosen according to the page size. Because a whole page of data would read even if only reading a few bits.

#### Best Practice
In general, when we use MySQL, we usually care about the ACID traction, so in this section we mainly talk about the index in InnoDB. Besides, we usually use composite index and the leftmost prefix is the most important rule.The usable prefix of the index, in most databases and under most circumstances, is up to and including the first inequality or range search condition. For example if we have a four-column index on columns (a, b, c, d) and we search for records where a=X, and b=Y, and c>Q, and d=R, then the usable prefix is only the first three columns.

##### Index for Search

* Case 1 Full Column Match: All where clauses are coverred by a composite index with `=` or `IN` query will definately use the index, with respect to leftmost rule.

* Case 2 String Match Query: `LIKE 'Senior%'` could use index.

* Case 3 Range Query: Only one column can use index at most.

* Case 4 Function Query: Cannot use index when where clause contains function 

##### Index for Order

### MySQL MyISAM
In MyISAM, the data areas of B-Tree leaf node are not the real data, it is a pointer to a `.MYD` file.
![vm_directory @1x]({{ "/assets/images/post/database-index/mysql-3.drawio.svg" | absolute_url }})


### Cassandra
The primary key in Cassandra is divided by partition key and clustering key. Besides, Cassandra also supports secondary index, and usually there are two methods to implement secondary index: Global Index and Local Index. Cassandra chooses the later, and it is tricky to use and can impact performance greatly. The index table is stored on each node in a cluster, so a query involving a secondary index can rapidly become a performance nightmare if multiple nodes are accessed.


## Conclusion










