---
title: MySQL Index
categories:
 - Database
tags: [Database Index, MySQL]
---

## Introduction
Index is a data structure that helps the database system obtain data efficiently. It improves the data retrieval process in the database at the cost of adding additional write operations and storage space for maintaining the index data structure. To search an entry, database would search the index for the entry first, and index is pointing to real physical location of the entry.


## Binary Search Tree
When we talk about search, in general, there are two types of search senarios: equal search and range search, so the first data structure might be binary seach tree. The defination of BST is a tree has following properties:
* The left subtree of a node contains only nodes with values lesser than the node’s value.
* The right subtree of a node contains only nodes with values greater than the node’s value.
* The left and right subtree each must also be a binary search tree.

The complexity analysis of BST shows that, on average, the insert, delete and search takes `O(logn)` for n nodes. In the worst case, they degrade to that of a singly linked list: `O(n)`. So if we can get a binary search tree with lower height, we would get a better search performance. AVL tree and red-black tree are two kinds of balanced BST, they both sacrifice the performance of insert/delete to get a better search performance. RBT would get a balanced performance among all operations, so it is widely used in-memory search (even there would be some improvements).  

## B-Tree, B+ Tree
But if we get more data which cannot fit all in memory, then we need to read data from disk, things are chagned. Reading from disk is much slower than from memory. B-tree is a balanced multi-way search tree, the biggest difference between B-tree and red-black tree is that B-tree nodes can have multiple children, from a few to several thousand.
* It is perfectly balanced: every leaf node is at the same depth.
* Every node, except perhaps the root, is at least half-full, i.e. contains M/2 or more values (of course, it cannot contain more than M-1 values). The root may have any number of values (1 to M-1).

B + Tree is a variation of the B-tree data structure, it has following specialities:
* leaf nodes store all key values, so some of the key values of the leaf nodes also appear in the internal nodes. (B Tree doesn't duplicated key values at leaf nodes)
* only leaf nodes store data values/pointers. (in B Tree, data values/pointers store with key together, maybe at non-leaf nodes)
* the leaf nodes are linked to providing ordered access to the records, which can improve range query. 

![vm_directory @8x]({{ "/assets/images/post/database-index/mysql-1.drawio.svg" | absolute_url }})

For most of the relational databases like MySQL, Oracle, they use B+ Tree for their index. In general, the index itself is also very large and cannot be all stored in memory, so the index is often stored on disk in the form of index files. In this way, the index search process will produce disk I/O consumption, relative to memory access, I/O access consumption is several orders of magnitude higher, so the most important indicator to evaluate a data structure as an index is the gradual complexity of the number of disk I/O operations in the search process. In other words, the index is structured to minimize the number of disk I/O accesses during lookup.

Let us use MySQL as an example, there are two mainly used database engines: MyISAM and InnoDB.



### MySQL InnoDB
In InnoDB, it clusters the Primary Key with the data (clustered index). This says that a PK lookup can rapidly get to the entire row. A secondary key lookup, however, has to go through 2 BTree lookups -- one in the secondary key's BTree, then a second in the PK's BTree.
![vm_directory @8x]({{ "/assets/images/post/database-index/mysql-2.drawio.svg" | absolute_url }})


![vm_directory @8x]({{ "/assets/images/post/database-index/mysql-4.drawio.svg" | absolute_url }})

We can see the page size on InnoDB is 16 k, and the size of a B-Tree node is chosen according to the page size. Because a whole page of data would read even if only reading a few bits.



### MySQL MyISAM
In MyISAM, the data areas of B-Tree leaf node are not the real data, it is a pointer to a `.MYD` file.
![vm_directory @8x]({{ "/assets/images/post/database-index/mysql-3.drawio.svg" | absolute_url }})


## Best Practice
In general, when we use MySQL, we usually care about the ACID traction, so in this section we mainly talk about the index in InnoDB. Besides, we usually use composite index and the leftmost prefix is the most important rule.The usable prefix of the index, in most databases and under most circumstances, is up to and including the first inequality or range search condition. For example if we have a four-column index on columns (a, b, c, d) and we search for records where a=X, and b=Y, and c>Q, and d=R, then the usable prefix is only the first three columns.

### Index for Search

* Case 1 Full Column Match: All where clauses are coverred by a composite index with `=` or `IN` query will definately use the index, with respect to leftmost rule.

* Case 2 String Match Query: `LIKE 'Senior%'` could use index.

* Case 3 Range Query: Only one column can use index at most.

* Case 4 Function Query: Cannot use index when where clause contains function 

### Index for Order








