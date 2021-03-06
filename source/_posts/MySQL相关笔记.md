---
title: MySQL相关笔记
date: 2018-03-13 16:10:54
tags:
- MySQL
comments: true
categories: 原创
---

# MySQL引擎
&nbsp;&nbsp;&nbsp;&nbsp;MySQL的默认引擎是InnoDB，该引擎支持行锁，支持事务安全表。其次，MySQL其他常用的引擎是MyISAM，但是它只支持表锁，并且不支持事务安全表。

&nbsp;&nbsp;&nbsp;&nbsp;InnoDB引擎的默认隔离级别为Reaptable Read，他消除了不可重复读（删、改）的现象，并与MVCC兼容消除了幻读（增）的现象。MVCC只与Read Committed和Reaptable Read两种隔离级别兼容。<!-- more -->

---
<br>

# MySQL的事务
&nbsp;&nbsp;&nbsp;&nbsp;MySQL的事务有四个特性：原子性、一致性、隔离性、持久性。
- 原子性：构成事务的所有操作必须是一个逻辑单元，要么全部执行，要么全部不执行。
- 一致性：数据库总是从一个一致性的状态转换到另外一个一致性的状态。
- 隔离性：通常来说，一个事务所做的修改在最终提交之前，对其他事务是不可见的。
- 持久性：一旦事务提交，其所做的修改会永久保存到数据库中。

&nbsp;&nbsp;&nbsp;&nbsp;事务实现隔离性的原理就是依赖于InnoDB引擎行锁。

---
<br>

# MySQL索引
&nbsp;&nbsp;&nbsp;&nbsp;MySQL的数据是放在外部存储的，也就是说IO才是性能瓶颈的关键，所以我们的索引需要减少树的深度，所以我们需要更多分叉的树，还需要更适合磁盘操作特性的数据结构。因此，MySQL选择了B+树作为其索引的数据结构。

&nbsp;&nbsp;&nbsp;&nbsp;使用B+树作为索引数据结构的优势：
- 只有叶子节点才记录数据，非叶子节点只包含索引；所有非终端节点（内部节点）并不存储数据信息，而是保存其叶子节点的最小值作为索引。这样，一次性读入内存中的需要查找的关键字就越多。相对来说IO读写次数也就降低了。
- 能够提供稳定高效的范围扫描功能；这个特点主要因为所有叶子节点相互连接，并且叶子节点本身依关键字的大小自小而大顺序链接。

---
<br>

# MySQL三段封锁协议
&nbsp;&nbsp;&nbsp;&nbsp;共享锁（S锁）：又称读锁，若事务T对数据对象A加上S锁，则其他事务只能再对A加S锁，不能加X锁，直到事务T释放A上的锁。保证了其他事务可以读取数据A，但是在T释放A的锁之前不能对其进行修改。

&nbsp;&nbsp;&nbsp;&nbsp;排他锁（X锁）：又称写锁，若事务T对数据对象A加上X锁，则其他事务都不能再对A加上任何类型的锁。保证了其他事务在T释放A的锁之前既不能读取也不能修改A。

&nbsp;&nbsp;&nbsp;&nbsp;**三段封锁协议：**
- 一级封锁协议：事务T在修改数据R之前必须对其先加上X锁，直到事务结束（正常或异常）才释放。1级封锁协议可防止丢失修改，并保证事务T是可恢复的。在1级封锁协议中，如果仅仅是读数据，是不需要加锁的，所以可能产生不可重复读和脏读。
- 二级封锁协议：在1级封锁协议的基础上加上 事务T在读取数据R之前必须对R加上S锁，**读完后可释放锁**。2级封锁协议防止了修改丢失，还可进一步防止脏读。
- 三级封锁协议：在1级封锁协议的基础上加上 事务T在读取数据R之前必须对R加上S锁，**直到事务结束才释放**。3级封锁协议防止了修改丢失和脏读外，还防止了不可重复读。

---
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)