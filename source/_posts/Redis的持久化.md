---
title: Redis的持久化
date: 2018-04-10 20:19:29
tags:
- Redis
- 持久化
comments: true
categories: 原创
---

# Redis持久化的两种方式
&nbsp;&nbsp;&nbsp;&nbsp;Redis是一种**基于内存**的数据库，因为不需要从磁盘读取数据，所以Redis带来了高效的访问速度，除此之外，redis还支持**多种数据结构**的存储：String，List，Hash散列表，Set和SortSet。

&nbsp;&nbsp;&nbsp;&nbsp;因为Redis是基于内存的数据库，所以当Redis发生掉电或者宕机等意外时，内存里面的数据是会丢失的。因此，Redis用两种方法对数据进行了持<!-- more -->久化，来防止数据的丢失。分别是**RDB**和**AOF**。Redis默认开启RDB，而AOF默认是关闭的。

---
<br>

# RDB持久化
&nbsp;&nbsp;&nbsp;&nbsp;RDB持久化可以在指定时间间隔内生成数据集的时间点快照，RDB每次持久化都会将内存中所有的数据都进行备份，存储到名为dump.rdb的二进制文件中，如果该文件已经存在，则将其覆盖。

&nbsp;&nbsp;&nbsp;&nbsp;RDB有三种方式对数据进行持久化：sava，bgsave以及配置文件进行设置。

&nbsp;&nbsp;&nbsp;&nbsp;配置文件里可以对Redis进行设置，让它在“N秒内数据至少有M个改动”这一条件被满足时，自动保存一次数据集。保存方式为调用bgsave方法。

&nbsp;&nbsp;&nbsp;&nbsp;当我们运用save方法进行调用的时候，会对其他命令造成阻塞；而bgsave会调用fork()方法创建一个子进程，子进程对数据进行持久化，父进程执行其他命令，所以bgsava不会造成阻塞。

### RDB的优点
&nbsp;&nbsp;&nbsp;&nbsp;RDB是一个非常紧凑的文件，它保存了Redis在某个时间点上的数据集。这种文件很适合**备份、灾难恢复**。RDB可以**最大化Redis的性能**：父进程在保存RDB文件的时候只需fork出一个子进程，子进程进行保存工作，父进程无须进行磁盘IO。RDB在恢复大数据集时的速度比AOF要快。

### RDB的缺点
&nbsp;&nbsp;&nbsp;&nbsp;RDB根据时间间隔来持久化数据，如果在时间间隔内发生宕机或者其他意外，在这段时间内的所有数据将会丢失。

---
<br>

# AOF持久化
&nbsp;&nbsp;&nbsp;&nbsp;与RDB通过直接保存Redis的键值对数据不同，AOF持久化是通过**保存**Redis**执行的写命令**来记录Redis的内存数据的。AOF有三个持久化频率，分别为：
- always：每执行以个Redis命令都要同步写入磁盘
- everysec：每秒执行一次同步，显示的将多个命令同步到磁盘
- no：让操作系统来决定应该何时同步

&nbsp;&nbsp;&nbsp;&nbsp;AOF**默认是everysec**级别。AOF文件的生成过程具体包括**命令追加、文件写入、文件同步**三个步骤。

### AOF的优点
&nbsp;&nbsp;&nbsp;&nbsp;AOF持久化会让Redis变得**更加耐久**，在默认情况下，即使发生故障停机，最多也只会丢失1秒的数据。Redis可以在AOF文件体积变得过大时，**对AOF文件进行重写**以缩小文件体积。
&nbsp;&nbsp;&nbsp;&nbsp;

### AOF的缺点
&nbsp;&nbsp;&nbsp;&nbsp;对于相同的数据集来说，AOF的体积通常要大于RDB的体积。根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。

---
<br>

# RDB和AOF之间的相互作用
&nbsp;&nbsp;&nbsp;&nbsp;在版本号大于等于2.4的Redis 中，BGSAVE执行的过程中，不可以执行 BGREWRITEAOF。 反过来说，在 BGREWRITEAOF 执行的过程中，也不可以执行BGSAVE 。

&nbsp;&nbsp;&nbsp;&nbsp;这可以防止两个 Redis 后台进程同时对磁盘进行大量的I/O操作。如果 BGSAVE 正在执行，并且用户显示地调用BGREWRITEAOF命令， 那么服务器将向用户回复一个OK状态，并告知用户，BGREWRITEAOF已经被预定执行：一旦 BGSAVE 执行完毕，BGREWRITEAOF就会正式开始。

&nbsp;&nbsp;&nbsp;&nbsp;当Redis 启动时，如果RDB持久化和AOF持久化都被打开了，那么程序会优先使用AOF文件来恢复数据集，因为AOF文件所保存的数据通常是最完整的。

---
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)