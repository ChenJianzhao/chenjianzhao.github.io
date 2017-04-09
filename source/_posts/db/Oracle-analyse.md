---
categories: 数据库
date: 2016-12-28 10:50
status: public
title: Oracle-analyse
---

## Oracle优化器
***
Oracle查询会经过优化器的判断选择查询方式。
oracle优化器：RBO（基于规则优化器）和CBO（基于成本优化器）两种， 从oracle10g开始优化器已经抛弃了RBO


## CBO（Cost Base Optimizer）
***
CBO的预测结果跟统计信息有关，如果统计信息陈旧，Oracle的执行计划将会出现偏差。


## Oracle收集统计信息
***
```sql
-- 注意：下述语句中 'TEST0510' 需要替换为环境中的数据库名
--select owner from dba_tables where table_name=UPPER('t_bg_bgcontrolitemmap');
BEGIN
dbms_stats.gather_table_stats('TEST082401','t_bg_bgcontrolitemmap');
dbms_stats.gather_index_stats('TEST082401','UX_BG_BGCONTROLIM');
END;
-- 或者
exec dbms_stats.gather_table_stats(user,'t',cascade => true);  
```


## 查看执行计划
***
```sql
EXPLAIN PLAN FOR [sql stattement]
```