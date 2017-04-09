---
categories: 数据库
date: 2016-12-09 16:43
status: public
title: 'Oracle 删除大量重复数据'
---

## 背景
业务中遇到一个问题，因数据量本身较大（120万左右），普通组合索引查询缓慢，尝试建立唯一索引加快查询速度，但过程中发现索引字段存在重复数据，无法建立唯一所以。
## 解决方法：
1.利用 Oracle 的 rowid 删除重复数据，保留 rowid 最小的那条数据。
```sql
-- 删除重复数据
delete from T_BG_BgControlItemMap a where rowid > 
(select min(b.rowid) from T_BG_BgControlItemMap b 
where a.fbillitemcombinvalue = b.fbillitemcombinvalue
and a.fbgctrlschemeid = b.fbgctrlschemeid
and a.fbgitemcombinkey = b.fbgitemcombinkey);
```
2.建立唯一索引
```sql
CREATE UNIQUE INDEX UX_BG_BGCONTROLIM ON T_BG_BgControlItemMap(FBillItemCombinValue ASC, FBgCtrlSchemeID ASC);
```
> Oracle 在数据表发生大量数据变动时，需要手动收集统计信息，用于执行语句前的决策。

3.手动执行收集表和索引的统计信息
```sql
BEGIN
dbms_stats.gather_table_stats('DB_OWNER','t_bg_bgcontrolitemmap');
dbms_stats.gather_index_stats('DB_OWNER','UX_BG_BGCONTROLIM');
END;
```