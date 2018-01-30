---
title: Explain参数解析
date: 2017-11-30 19:58:09
tags:
- MySQL
- Explain
comments: true
categories: 日志
---
&nbsp;&nbsp;&nbsp;&nbsp;当我们想要查询Mysql数据库里面的索引的时候，我们可以使用Explain语句来查看具体的索引信息，例如：explain  select * from user  ，它将会显示如何使用索引来处理select语句以及连接表，并帮助我们选择更好的索引和写出更优化的查询语句。

&nbsp;&nbsp;&nbsp;&nbsp;下面将会讲解几个主要的参数以及它们的值的具体含义。

---

## select_type
&nbsp;&nbsp;&nbsp;&nbsp;select的具体类型，它有以下几种值：<!-- more -->
- **simple**：简单的select查询，没有union或者子查询
- **primary**：最外面的select，在有子查询的语句中，最外面的select查询就是primary
- **union**：union语句的第二个或者后面那一个select语句
- **dependent union**：union中的第二个或者后面的select语句，取决于外面的查询
- **union result**：union的结果


## type
&nbsp;&nbsp;&nbsp;&nbsp;联接类型。有多个参数，从最佳类型到最差类型进行排列：
- **system**：表仅有一行。这是const联接类型的一个特例，平时不会出现
- **const**：表最多有一个匹配行，因为只匹配一行数据，所以很快。一定要primary_key或者unique，并且只检索出两条数据的情况下才会是const
- **eq_ref**：对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY时
- **ref**：对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref
- **ref_or_null**：该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行
- **index_merge**：该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素
- **unique_subquery**
- **index_subquery**
- **range**： 给定范围内的检索，使用一个索引来检查行
- **index**：该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小（index从索引中读取，all从硬盘中读取）
- **all**：对于每个来自于先前的表的行组合，进行完整的表扫描

## possible_keys
&nbsp;&nbsp;&nbsp;&nbsp;提示使用哪个索引会在该表中找到行

## keys
&nbsp;&nbsp;&nbsp;&nbsp;Mysql在查询中实际使用的索引，若没有使用索引，则为null

## key——len
&nbsp;&nbsp;&nbsp;&nbsp;Mysql使用的索引的长度，如果索引为null，则长度为null

## ref
&nbsp;&nbsp;&nbsp;&nbsp;显示使用哪个列或常数与key一起从表中选择行

## rows
&nbsp;&nbsp;&nbsp;&nbsp;显示Mysql执行查询的行数，数值越大越不好，说明没有用好索引

## Extra
&nbsp;&nbsp;&nbsp;&nbsp;该列包含了Mysql解决查询的详细信息：
- **Distinct**：Mysql发现匹配行后，停止为当前的行组合搜索更多的行
- **Not exists**：MySQL能够对查询进行LEFT JOIN优化,发现1个匹配LEFT JOIN标准的行后,不再为前面的的行组合在该表内检查更多的行
- **range checked for each record**：没有找到合适的索引
- **using filesort**：MySQL需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配WHERE子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行
- **using index**（覆盖索引）：只是用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息。
- **using temporary**：为了解决查询，MySQL需要创建一个临时表来容纳结果
- **using where**：WHERE子句用于限制哪一个行匹配下一个表或发送到客户。
- **Using sort_union(...), Using union(...), Using intersect(...)**：这些函数说明如何为index_merge联接类型合并索引扫描
- **Using index for group-by**：表示MySQL发现了一个索引，可以用来查询GROUP BY或DISTINCT查询的所有列，而不要额外搜索硬盘访问实际的表
