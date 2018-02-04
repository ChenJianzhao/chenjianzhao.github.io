---
categories: MySQL
date: 2018-01-27 15:00
title: MySQL SQL语句编写注意问题，索引的应用
---

## SQL语句编写注意问题

下面就某些SQL语句的where子句编写中需要注意的问题作详细介绍。在这些where子句中，即使某些列存在索引，但是由于编写了劣质的SQL，系统在运行该SQL语句时也不能使用该索引，而同样使用全表扫描，这就造成了响应速度的极大降低。

<!-- more -->

### 操作符优化

#### **IN 操作符**

用IN写出来的SQL的优点是比较容易写及清晰易懂，这比较适合现代软件开发的风格。但是用IN的SQL性能总是比较低的，从Oracle执行的步骤来分析用IN的SQL与不用IN的SQL有以下区别：

ORACLE试图将其转换成多个表的连接，如果转换不成功则先执行IN里面的子查询，再查询外层的表记录，如果转换成功则直接采用多个表的连接方式查询。由此可见用IN的SQL至少多了一个转换的过程。一般的SQL都可以转换成功，但对于含有分组统计等方面的SQL就不能转换了。

**推荐方案：**在业务密集的SQL当中尽量不采用IN操作符，用EXISTS 方案代替。（EXISTS  用法见文后）



#### NOT IN操作符

此操作是强列不推荐使用的，因为它不能应用表的索引。

**推荐方案：**用NOT EXISTS 方案代替

```sql
explain select * from t_user where  FUsername = 'user' ;
explain select * from t_user where  not (FUsername = 'user' );
```

会发现第一个语句走了索引，第二个没有。



#### IS NULL 或IS NOT NULL操作（判断字段是否为空）

判断字段是否为空一般是不会应用索引的，因为索引是不索引空值的。不能用null作索引，任何包含null值的列都将不会被包含在索引中。即使索引有多列这样的情况下，只要这些列中有一列含有null，该列就会从索引中排除。也就是说如果某列存在空值，即使对该列建索引也不会提高性能。

任何在where子句中使用is null或is not null的语句优化器是不允许使用索引的。

**推荐方案**：

- 用其它相同功能的操作运算代替，如：`a is not null` 改为 `a>0 或 a>''` 等。
- 不允许字段为空，而用一个缺省值代替空值，如申请中状态字段不允许为空，缺省为申请。



#### > 及 < 操作符（大于或小于操作符）

大于或小于操作符一般情况下是不用调整的，因为它有索引就会采用索引查找，但有的情况下可以对它进行优化，如一个表有100万记录，一个数值型字段A，30万记录的A=0，30万记录的A=1，39万记录的A=2，1万记录的A=3。那么执行A>2与A>=3的效果就有很大的区别了，因为A>2时ORACLE会先找出为2的记录索引再进行比较，而A>=3时ORACLE则直接找到=3的记录索引。



#### LIKE 操作符

LIKE操作符可以应用通配符查询，里面的通配符组合可能达到几乎是任意的查询，但是如果用得不好则会产生性能上的问题，如 `LIKE '%5400%'` 这种查询不会引用索引，而LIKE ‘X5400%’则会引用范围索引。

一个实际例子：用 YW_YHJBQK 表中营业编号后面的户标识号可来查询营业编号 `YY_BH LIKE '%5400%'` 这个条件会产生全表扫描，如果改成 `YY_BH LIKE 'X5400%' OR YY_BH LIKE 'B5400%'` 则会利用YY_BH的索引进行两个范围的查询，性能肯定大大提高。



#### UNION操作符

UNION 在进行表链接后会 **筛选掉重复的记录**，所以在表链接后会对所产生的结果集进行排序运算，删除重复的记录再返回结果。实际大部分应用中是不会产生重复的记录。

**推荐方案：**采用 UNION ALL 操作符替代 UNION，因为UNION ALL操作只是简单的将两个结果合并后就返回。



#### Order by语句

ORDER BY语句决定了Oracle如何将返回的查询结果排序。Order by语句对要排序的列没有什么特别的限制，也可以将函数加入列中(象联接或者附加等)。任何在Order by语句的**非索引项或者有计算表达式**都将降低查询速度。

仔细检查order by语句以找出非索引项或者表达式，它们会降低性能。解决这个问题的办法就是重写order by语句以使用索引，也可以为所使用的列建立另外一个索引，同时应绝对避免在order by子句中使用表达式。



#### 尽量避免 OR 操作

```sql
select * from dept where dname='jaskey' or loc='bj' or deptno=45;
--如果条件中有or,即使其中有条件带索引也不会使用。换言之,就是要求使用的所有字段,都必须建立索引
```

所以除非每个列都建立了索引，否则不建议使用OR，在多列OR中，可以考虑用UNION 替换

```sql
select * from dept where dname='jaskey' union
select * from dept where loc='bj' union
select * from dept where deptno=45;
```



## 其他

**（1） 选择最有效率的表名顺序(只在基于规则的优化器中有效)：**

ORACLE 的解析器按照从右到左的顺序处理FROM子句中的表名，FROM子句中写在最后的表(基础表 driving table)将被最先处理，在FROM子句中包含多个表的情况下,你必须选择记录条数最少的表作为基础表。如果有3个以上的表连接查询, 那就需要选择交叉表(intersection table)作为基础表, 交叉表是指那个被其他表所引用的表.

**（2） WHERE子句中的连接顺序：**

ORACLE采用自下而上的顺序解析WHERE子句,根据这个原理,表之间的连接必须写在其他WHERE条件之前, 那些可以过滤掉最大数量记录的条件必须写在WHERE子句的末尾.



## 补充

### EXISTS 的用法

例子：比如在Northwind数据库中有一个查询为

```sql
SELECT c.CustomerId,CompanyName FROM Customers c
WHERE EXISTS(
SELECT OrderID FROM Orders o WHERE o.CustomerID=c.CustomerID) 
```

这里面的EXISTS是如何运作呢？子查询返回的是OrderId字段，可是外面的查询要找的是CustomerID和CompanyName字段，这两个字段肯定不在OrderID里面啊，这是如何匹配的呢？ 

**EXISTS 用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False。**

`EXISTS` 指定一个子查询，检测行的存在。

**语法：** EXISTS subquery
**参数：** subquery 是一个受限的 SELECT 语句 (不允许有 COMPUTE 子句和 INTO 关键字)。
**结果类型：** Boolean 如果子查询包含行，则返回 TRUE ，否则返回 FLASE 。



### EXISTS 和 IN

比较使用 `EXISTS` 和 `IN` 的查询。注意两个查询返回相同的结果。

```sql
select * from TableIn where exists(select BID from TableEx where BNAME=TableIn.ANAME)
select * from TableIn where ANAME in(select BNAME from TableEx)
```

`NOT EXISTS` 的作用与 `EXISTS` 正好相反。如果子查询没有返回行，则满足了 NOT EXISTS 中的 WHERE 子句。

结论：
EXISTS(包括 NOT EXISTS )子句的返回值是一个**BOOL值**。 EXISTS内部有一个子查询语句(SELECT ... FROM...)， 我将其称为EXIST的内查询语句。其内查询语句返回一个结果集。 EXISTS子句根据其内查询语句的结果集空或者非空，返回一个布尔值。

**一种通俗的可以理解为：将外查询表的每一行，代入内查询作为检验，如果内查询返回的结果取非空值，则EXISTS子句返回TRUE，这一行行可作为外查询的结果行，否则不能作为结果。**

EXISTS与IN的使用效率的问题，通常情况下采用exists要比in效率高，因为**IN不走索引**，但要看实际情况具体使用：IN适合于外表大而内表小的情况；**EXISTS适合于外表小而内表大的情况**。

参考：[SQL中EXISTS的用法](http://www.cnblogs.com/netserver/archive/2008/12/25/1362615.html)
