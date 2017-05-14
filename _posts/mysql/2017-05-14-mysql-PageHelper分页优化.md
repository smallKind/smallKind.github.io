---
layout: post
title: PageHelper分页优化
date: 2017-05-14 09:59:01 +8000
category: MySQL
tags: 性能优化
---

* content
{:toc}

### 问题原由

项目采用PageHelper插件分页，但因数据量大的时候，分页的性能不理想，所以进行分页性能的优化。

### 问题重现

create_time为索引列，表数据大约在300多万行：
  
  项目中分页查询类似于：select * from 表名 order by create\_time limit 1000000,5;
  
  提出的优化方案：select id from 表名 order by create\_time limit 1000000,5;
  
  解决思路在于：上者在于先从磁盘中拿出所有符合条件的数据，然后截取想得到的返回结果（也就是从磁盘取出1000005条数据，截取5条返回）；后者在于先从索引中截取，得到想要的返回结果主键（id）,然后根据主键从磁盘取出数据；（也就是从索引截取5条数据的id，根据id在到磁盘中查找出返回数据）。从磁盘IO的次数也就可以看出，后者更优。
  
  但在过程中发现一下问题：
  
  1. 原返回结果的id与优化方案返回结果的id不一致？
  2. 从MySQL的执行计划分析，原SQL采用的是全表扫描，明明存在create_time的单索引列，为什么没有使用索引，而是全表扫描？

### 问题分析

select * from 表名 order by create\_time limit 100,5;

select id from 表名 order by create\_time limit 100,5;

把limit偏移位置改成100，比较两者的结果，发现两者结果的id一致，修改成其他比较小的值也是一样的。但修改到比较大的值的时候，返回结果的id又不一致的。why?

explain分析SQL语句，发现偏移量小，访问类型为index，也就是create_time索引生效。但当偏移量大，访问类型为ALL，也就是全表扫描。也就是说在偏移量不同时，MySQL优化器选择的执行计划是不一样的，为什么在偏移量很大的时候，放弃索引而选择全表扫描？

select * from 表名 force index(index\_create\_time) order by create\_time limit 1000000,5;强制MySQL优化器选择使用该索引，发现结果与预期结果一致。原先在分析select * from 表名 order by create\_time limit 1000000,5的执行计划时，它是全表扫描，using filesort。也就是说原先返回结果不一致，是因为filesort缘故，在MySQL进行数据排序的时候，相同的key值，排序时位置次序发生变化，导致的结果发生了变化。为什么会using filesort，原因在于全表扫描，所以又回到了第二个问题上。

分析什么时候MySQL优化器为什么不选择索引，而是用全表扫描？哪些情况下，优化器会弃用索引？具体不一一列举，最符合情况的就是：表字段的值导致不走索引，导致优化器认为需要扫描索引大部分数据且聚簇因子很大，最终导致弃用索引扫描而改用全表扫描方式。也就是说该索引列的区分度不够高。

select count(distinct create_time) from 表名;返回数据量大小为1700多，也就说400多万条的数据，create_time不重复的只有1700多条，至此上述两个问题也就找到了原因。

### 解决思路

在回归到最开始的问题中，怎么优化分页性能，从项目分析，可以从两点入手：

1. select id（主键） 代替 select * 或 select a,b,c,d……,原因上述也分析，在偏移量越大的情况下，性能越佳，缺点因使用PageHelper需要查询数据库两次。在有where条件下一次查询，也避免了全表扫描。
2. PageHelper中分页，select count(*)操作explain分析虽然使用了索引，但耗时也是很明显的，而且每次查询的时候都select count(*),既然不能优化它执行的时间，但可以减少它执行的次数，在第一次select count(*)之后，每次翻页查询的时候，把count(*)的数据回传。在每次PageHelper分页的时候，只有第一次进行select count(*),后面的默认不查询
