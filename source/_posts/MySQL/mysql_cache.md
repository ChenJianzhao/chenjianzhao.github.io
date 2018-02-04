---
categories: MySQL
date: 2018-02-02 22:00
title: MySQL 缓存
---

对于很多的数据库系统都能够缓存执行计划，对于**完全相同的sql**, 可以使用已经已经存在的执行计划，从而跳过解析和生成执行计划的过程。

MYSQL以及Oracle提供了更为高级的查询结果缓存功能，对于**完全相同的SQL (字符串完全相同且大小写敏感)** 可以执行返回查询结果。本文主要介绍MYSQL 查询缓存的一些特性，Oracle query cache可以参考 [Oracle query cache](http://www.oracle.com/technetwork/articles/sql/11g-caching-pooling-088320.html)



## 查询缓存的工作机制

Mysql 判断是否命中缓存的办法很简单，首先会将要缓存的结果放在引用表中，然后使用查询语句，数据库名称，客户端协议的版本等因素算出一个hash值，这个hash值与引用表中的结果相关联。如果在执行查询时，根据一些相关的条件算出的hash值能与引用表中的数据相关联，则表示查询命中

通过 `have_query_cache` 服务器系统变量指示查询缓存是否可用：

```
mysql> SHOW VARIABLES LIKE 'have_query_cache';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | YES   |
+------------------+-------+
```

为了监视查询缓存性能，使用SHOW STATUS查看缓存状态变量：

```
mysql> SHOW STATUS LIKE 'Qcache%';
+-------------------------+--------+
|变量名                   |值 |
+-------------------------+--------+
| Qcache_free_blocks      | 36     |
| Qcache_free_memory      | 138488 |
| Qcache_hits             | 79570  |
| Qcache_inserts          | 27087  |
| Qcache_lowmem_prunes    | 3114   |
| Qcache_not_cached       | 22989  |
| Qcache_queries_in_cache | 415    |
| Qcache_total_blocks     | 912    |
+-------------------------+--------+
```



## 查询缓存机制失效的场景

先不论查询缓存机制有利有弊，先看看哪些场景下会导致缓存机制失效

- 如果查询语句中包含一些不确定因素时（例如包含 函数Current()），该查询不会被缓存,不确定因素主要包含以下情况
  - 引用了一些返回值不确定的函数
  - 引用自定义函数(UDFs)。
  - 引用自定义变量。
  - 引用mysql系统数据库中的表。