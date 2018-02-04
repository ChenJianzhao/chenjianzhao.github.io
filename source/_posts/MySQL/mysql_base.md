---
categories: MySQL
date: 2018-01-25 15:00
title: MySQL 基础
---

MySQL 基础语法

<!-- more -->

## MySQL 创建数据表

**语法**

以下为创建MySQL数据表的SQL通用语法：

`CREATE TABLE table_name (column_name column_type);`

以下例子中我们将在 RUNOOB 数据库中创建数据表runoob_tbl：

```sql
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

实例解析：

- 如果你不想字段为 NULL 可以设置字段的属性为 NOT NULL， 在操作数据库时如果输入该字段的数据为NULL ，就会报错。
- AUTO_INCREMENT定义列为自增的属性，一般用于主键，数值会自动加1。
- PRIMARY KEY关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。
- ENGINE 设置存储引擎，CHARSET 设置编码。

</br>

## MySQL 删除数据表

MySQL中删除数据表是非常容易操作的，但是你再进行删除表操作时要非常小心，因为执行删除命令后所有数据都会消失。

**语法**

以下为删除MySQL数据表的通用语法：

````sql
DROP TABLE table_name ;
````

</br>

## MySQL 插入数据

MySQL 表中使用 INSERT INTO SQL语句来插入数据。

**语法**

以下为向MySQL数据表插入数据通用的 INSERT INTO SQL语法：

```sql
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( value1, value2,...valueN );
```

如果数据是字符型，必须使用单引号或者双引号，如："value"。

同时插入多条记录

```sql
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( valueA1, valueA2,...valueAN ), 
                       ( valueB1, valueB2,...valueBN ),
                       ......,
                       ( valueK1, valueK2,...valueKN );
```

</br>

## MySQL 查询数据

MySQL 数据库使用SQL SELECT语句来查询数据。

**语法**

以下为在MySQL数据库中查询数据通用的 SELECT 语法：

```sql
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[LIMIT N][ OFFSET M]
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号(,)分割，并使用WHERE语句来设定查询条件。
- SELECT 命令可以读取一条或者多条记录。
- 你可以使用星号（*）来代替其他字段，SELECT语句会返回表的所有字段数据
- 你可以使用 `WHERE` 语句来包含任何条件。
- 你可以使用 `LIMIT` 属性来设定返回的记录数。
- 你可以通过 `OFFSET`指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0

</br>

## MySQL WHERE 子句

我们知道从 MySQL 表中使用 SQL SELECT 语句来读取数据。

如需有条件地从表中选取数据，可将 WHERE 子句添加到 SELECT 语句中。

**语法**

以下是 SQL SELECT 语句使用 WHERE 子句从数据表中读取数据的通用语法：

```sql
SELECT field1, field2,...fieldN 
FROM table_name1, table_name2...
[WHERE condition1 [AND [OR]] condition2.....
```

- 查询语句中你可以使用一个或者多个表，表之间使用逗号, 分割，并使用WHERE语句来设定查询条件。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以使用 AND 或者 OR 指定一个或多个条件。
- WHERE 子句也可以运用于 SQL 的 DELETE 或者 UPDATE 命令。
- WHERE 子句类似于程序语言中的 if 条件，根据 MySQL 表中的字段值来读取指定的数据。

</br>

## MySQL UPDATE 查询

如果我们需要修改或更新 MySQL 中的数据，我们可以使用 SQL UPDATE 命令来操作。.

**语法**

以下是 UPDATE 命令修改 MySQL 数据表数据的通用 SQL 语法：

```sql
UPDATE table_name SET 
field1=new-value1, 
field2=new-value2
[WHERE Clause]
```

- 你可以同时更新一个或多个字段。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以在一个单独表中同时更新数据。
- 当你需要更新数据表中指定行的数据时 WHERE 子句是非常有用的。

</br>

## MySQL DELETE 语句

你可以使用 SQL 的 DELETE FROM 命令来删除 MySQL 数据表中的记录。

**语法**

以下是 SQL DELETE 语句从 MySQL 数据表中删除数据的通用语法：

```sql
DELETE FROM table_name [WHERE Clause]
```

- 如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。
- 你可以在 WHERE 子句中指定任何条件
- 您可以在单个表中一次性删除记录。
- 当你想删除数据表中指定的记录时 WHERE 子句是非常有用的。

</br>

## MySQL LIKE 子句

runoob_author 字段含有 "COM" 字符的所有记录，这时我们就需要在 WHERE 子句中使用 SQL LIKE 子句。

SQL LIKE 子句中使用百分号 `%` 字符来表示任意字符，类似于UNIX或正则表达式中的星号 `*`。

如果没有使用百分号 `%`, LIKE 子句与等号 = 的效果是一样的。

**语法**

以下是 SQL SELECT 语句使用 LIKE 子句从数据表中读取数据的通用语法：

```sql
SELECT field1, field2,...fieldN 
FROM table_name
WHERE field1 LIKE condition1 
[AND [OR]] filed2 = 'somevalue'
```

- 你可以在 WHERE 子句中指定任何条件。
- 你可以在 WHERE 子句中使用LIKE子句。
- 你可以使用LIKE子句代替等号 =。
- LIKE 通常与 % 一同使用，类似于一个元字符的搜索。
- 你可以使用 AND 或者 OR 指定一个或多个条件。
- 你可以在 DELETE 或 UPDATE 命令中使用 WHERE...LIKE 子句来指定条件。

</br>

## MySQL UNION 操作符

描述 MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

**语法**

MySQL UNION 操作符语法格式：

```sql
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions]

UNION [ALL | DISTINCT]

SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions];
```

**参数**

- expression1, expression2, ... expression_n: 要检索的列。
- tables: 要检索的数据表。
- WHERE conditions: 可选， 检索条件。
- DISTINCT: 可选，删除结果集中重复的数据。默认情况下 UNION 操作符已经删除了重复数据，所以 DISTINCT 修饰符对结果没啥影响。
- ALL: 可选，返回所有结果集，包含重复数据。

</br>

## MySQL 排序

如果我们需要对读取的数据进行排序，我们就可以使用 MySQL 的 ORDER BY 子句来设定你想按哪个字段哪种方式来进行排序，再返回搜索结果。

**语法**

以下是 SQL SELECT 语句使用 ORDER BY 子句将查询数据排序后再返回数据：

```sql
SELECT field1, field2,...fieldN from
table_name1, table_name2...
ORDER BY field1, [field2...] [ASC [DESC]]
```

- 你可以使用任何字段来作为排序的条件，从而返回排序后的查询结果。
- 你可以设定多个字段来排序。
- 你可以使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。
- 你可以添加 WHERE...LIKE 子句来设置条件。

</br>

## MySQL GROUP BY 语句

GROUP BY 语句根据一个或多个列对结果集进行分组。

在分组的列上我们可以使用 COUNT, SUM, AVG,等函数。

**语法**

```sql
SELECT column_name, function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

</br>

## Mysql 连接的使用

你可以在 SELECT, UPDATE 和 DELETE 语句中使用 Mysql 的 JOIN 来联合多表查询。

JOIN 按照功能大致分为如下三类：

- INNER JOIN（内连接,或等值连接）：获取两个表中字段匹配关系的记录。
- LEFT JOIN（左连接）：获取左表所有记录，即使右表没有对应匹配的记录。
- RIGHT JOIN（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

</br>

## MySQL NULL 值处理

为了处理这种情况，MySQL提供了三大运算符:

1. IS NULL: 当列的值是 NULL,此运算符返回 true。
2. IS NOT NULL: 当列的值不为 NULL, 运算符返回 true。
3. <=>: 比较操作符（不同于=运算符），当比较的的两个值为 NULL 时返回 true。

- 关于 NULL 的条件比较运算是比较特殊的。你不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。
- 在 MySQL 中，NULL 值与任何其它值的比较（即使是 NULL）永远返回 false，即 NULL = NULL 返回false 。
- MySQL 中处理 NULL 使用 IS NULL 和 IS NOT NULL 运算符。

</br>

## MySQL ALTER命令

当我们需要修改数据表名或者修改数据表字段时，就需要使用到MySQL ALTER命令。

### 删除表字段

如下命令使用了 ALTER 命令及 DROP 子句来删除以上创建表的 i 字段：

```sql
ALTER TABLE testalter_tbl  DROP i;
```

如果数据表中只剩余一个字段则无法使用DROP来删除字段。

### 添加表字段

MySQL 中使用 ADD 子句来向数据表中添加列，如下实例在表 testalter_tbl 中添加 i 字段，并定义数据类型:

```sql
ALTER TABLE testalter_tbl ADD i INT;
```

如果你需要指定新增字段的位置，可以使用MySQL提供的关键字 FIRST (设定位第一列)， AFTER 字段名（设定位于某个字段之后）。

尝试以下 ALTER TABLE 语句, 在执行成功后，使用 SHOW COLUMNS 查看表结构的变化：

```sql
ALTER TABLE testalter_tbl DROP i;
ALTER TABLE testalter_tbl ADD i INT FIRST;
ALTER TABLE testalter_tbl DROP i;
ALTER TABLE testalter_tbl ADD i INT AFTER c;
```

### 修改字段类型及名称

如果需要修改字段类型及名称, 你可以在ALTER命令中使用 MODIFY 或 CHANGE 子句 。

例如，把字段 c 的类型从 CHAR(1) 改为 CHAR(10)，可以执行以下命令:

```sql
ALTER TABLE testalter_tbl MODIFY c CHAR(10);
```

使用 CHANGE 子句, 语法有很大的不同。 在 CHANGE 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型。尝试如下实例：

```sql
ALTER TABLE testalter_tbl CHANGE i j BIGINT;
ALTER TABLE testalter_tbl CHANGE j j INT;
```

### ALTER TABLE 对 Null 值和默认值的影响

当你修改字段时，你可以指定是否包含只或者是否设置默认值。

以下实例，指定字段 j 为 NOT NULL 且默认值为100 。

```sql
ALTER TABLE testalter_tbl 
MODIFY j BIGINT NOT NULL DEFAULT 100;
```

如果你不设置默认值，MySQL会自动设置该字段默认为 NULL。

### 修改字段默认值

你可以使用 ALTER 来修改字段的默认值，尝试以下实例：

```sql
ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000;
```

你也可以使用 ALTER 命令及 DROP子句来删除字段的默认值，如下实例：

```sql
ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
```

修改数据表类型，可以使用 ALTER 命令及 TYPE 子句来完成。尝试以下实例，我们将表 testalter_tbl 的类型修改为 MYISAM ：

**注意：** 查看数据表类型可以使用 `SHOW TABLE STATUS` 语句。

```sql
ALTER TABLE testalter_tbl ENGINE = MYISAM;
```

### 修改表名

如果需要修改数据表的名称，可以在 ALTER TABLE 语句中使用 `RENAME` 子句来实现。

```sql
ALTER TABLE testalter_tbl RENAME TO alter_tbl;
```

ALTER 命令还可以用来创建及删除MySQL数据表的索引，该功能我们会在接下来的章节中介绍。

</br>

## MySQL 事务

MySQL 事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！

- 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
- 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
- 事务用来管理 insert,update,delete 语句

一般来说，事务是必须满足4个条件（ACID）：：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

> 在 MySQL 命令行的默认设置下，事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作。因此要显式地开启一个事务务须使用命令 BEGIN 或 START TRANSACTION，或者执行命令 SET AUTOCOMMIT=0，用来禁止使用当前会话的自动提交。

### 事务控制语句：

- **BEGIN** 或 START TRANSACTION；显式地开启一个事务；

- **COMMIT**；也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改称为永久性的；

- **ROLLBACK**；有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

- **SAVEPOINT identifier**；SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；

- **RELEASE SAVEPOINT identifier**；删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

- **ROLLBACK TO identifier**；把事务回滚到标记点；

- **SET TRANSACTION**；用来设置事务的隔离级别。

  > InnoDB存储引擎提供事务的隔离级别有 `READ UNCOMMITTED`、`READ COMMITTED`、`REPEATABLE READ`和 `SERIALIZABLE`。

### MYSQL 事务处理主要有两种方法：

1、用 BEGIN, ROLLBACK, COMMIT来实现

- **BEGIN** 开始一个事务
- **ROLLBACK** 事务回滚
- **COMMIT** 事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式:

- **SET AUTOCOMMIT=0** 禁止自动提交
- **SET AUTOCOMMIT=1** 开启自动提交

</br>

## MySQL 索引

- MySQL索引的建立对于MySQL的高效运行是很重要的，索引可以大大提高MySQL的检索速度。
- 索引分单列索引和组合索引。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。组合索引，即一个索引包含多个列。
  - 创建索引时，你需要确保该索引是应用在SQL 查询语句的条件(一般作为 WHERE 子句的条件)。
- 实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。
- 上面都在说使用索引的好处，但过多的使用索引将会造成滥用。因此索引也会有它的缺点：虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。
- 建立索引会占用磁盘空间的索引文件。

### 普通索引

- 创建索引 这是最基本的索引，它没有任何限制。它有以下几种创建方式：

  ```Sql
  CREATE INDEX indexName ON mytable(username(length)); 
  ```

  如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。

- 修改表结构(添加索引)

  ```Sql
  ALTER table tableName ADD INDEX indexName(columnName)
  ```

- 创建表的时候直接指定

  ```sql
  CREATE TABLE mytable(  
  ID INT NOT NULL,   
  username VARCHAR(16) NOT NULL,  
  INDEX [indexName] (username(length))  
  ); 
  ```

- 删除索引的语法

  ```Sql
  DROP INDEX [indexName] ON mytable; 
  ```

### 唯一索引

它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：

- 创建索引

```sql
CREATE UNIQUE INDEX indexName ON mytable(username(length)) 
```

- 修改表结构

```sql
ALTER table mytable ADD UNIQUE [indexName]
(username(length))
```

- 创建表的时候直接指定

```sql
CREATE TABLE mytable(  
ID INT NOT NULL,   
username VARCHAR(16) NOT NULL,  
UNIQUE [indexName] (username(length))
);  
```

### 使用ALTER 命令添加和删除索引

有四种方式来添加数据表的索引：

- ALTER TABLE tbl_name ADD **PRIMARY KEY** (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
- ALTER TABLE tbl_name ADD **UNIQUE** index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
- ALTER TABLE tbl_name ADD **INDEX** index_name (column_list): 添加普通索引，索引值可出现多次。
- ALTER TABLE tbl_name ADD **FULLTEXT** index_name (column_list):该语句指定了索引为 FULLTEXT ，用于全文索引。



### 使用 ALTER 命令添加和删除主键

主键只能作用于一个列上，添加主键索引时，你需要确保该主键默认不为空（NOT NULL）。实例如下：

```sql
ALTER TABLE testalter_tbl MODIFY i INT NOT NULL;
ALTER TABLE testalter_tbl ADD PRIMARY KEY (i);
```

你也可以使用 ALTER 命令删除主键：

```sql
ALTER TABLE testalter_tbl DROP PRIMARY KEY;
```

删除主键时只需指定PRIMARY KEY，但在删除索引时，你必须知道索引名。

### 显示索引信息

你可以使用 SHOW INDEX 命令来列出表中的相关的索引信息。可以通过添加 \G 来格式化输出信息。

尝试以下实例:

```sql
SHOW INDEX FROM table_name; \G
```

</br>

## MySQL 临时表

MySQL 临时表在我们需要保存一些临时数据时是非常有用的。临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。

如果你使用了其他MySQL客户端程序连接MySQL数据库服务器来创建临时表，那么只有在关闭客户端程序时才会销毁临时表，当然你也可以手动销毁。

### 实例

### 创建MySQL 临时表

```sql
mysql> CREATE TEMPORARY TABLE SalesSummary (
    -> product_name VARCHAR(50) NOT NULL
    -> , total_sales DECIMAL(12,2) NOT NULL DEFAULT 0.00
    -> , avg_unit_price DECIMAL(7,2) NOT NULL DEFAULT 0.00
    -> , total_units_sold INT UNSIGNED NOT NULL DEFAULT 0
);
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO SalesSummary
    -> (product_name, total_sales, avg_unit_price, total_units_sold)
    -> VALUES
    -> ('cucumber', 100.25, 90, 2);

mysql> SELECT * FROM SalesSummary;
+--------------+-------------+----------------+------------------+
| product_name | total_sales | avg_unit_price | total_units_sold |
+--------------+-------------+----------------+------------------+
| cucumber     |      100.25 |          90.00 |                2 |
+--------------+-------------+----------------+------------------+
1 row in set (0.00 sec)
```

当你使用 **SHOW TABLES**命令显示数据表列表时，你将无法看到 SalesSummary表。

如果你退出当前MySQL会话，再使用 **SELECT**命令来读取原先创建的临时表数据，那你会发现数据库中没有该表的存在，因为在你退出时该临时表已经被销毁了。



### 删除MySQL 临时表

默认情况下，当你断开与数据库的连接后，临时表就会自动被销毁。当然你也可以在当前MySQL会话使用 **DROP TABLE**命令来手动删除临时表。

以下是手动删除临时表的实例：

```sql
mysql> DROP TABLE SalesSummary;
mysql>  SELECT * FROM SalesSummary;
ERROR 1146: Table 'RUNOOB.SalesSummary' doesn't exist
```



## MySQL 复制表

### 简单的复制表

方法一：先创建一个结构和源表相同的目标表，在将源表的数据复制到目标表。

```sql
CREATE TABLE targetTable LIKE sourceTable;
INSERT INTO targetTable SELECT * FROM sourceTable;
```

方法二：创建目标表的同时复制源表的数据。

```sql
CREATE TABLE targetTable AS (SELECT * from sourceTable);
```

其他方法：

可以拷贝一个表中其中的一些字段:

```sql
CREATE TABLE newadmin AS
(
    SELECT username, password FROM admin
)
```

可以将新建的表的字段改名:

```sql
CREATE TABLE newadmin AS
(  
    SELECT id, username AS uname, password AS pass FROM admin
)
```

可以拷贝一部分数据:

```sql
CREATE TABLE newadmin AS
(
    SELECT * FROM admin WHERE LEFT(username,1) = 's'
)
```

可以在创建表的同时定义表中的字段信息:

```sql
CREATE TABLE newadmin
(
    id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY
)
AS
(
    SELECT * FROM admin
)  
```



### 完全复制表

简单复制表你会发现，复制的表没有包含源表的主键、索引默认值等信息。

如果我们需要完全的复制MySQL的数据表，包括表的结构，索引，默认值等。 如果仅仅使用CREATE TABLE ... SELECT 命令，是无法实现的。

使用 SHOW CREATE TABLE 命令获取创建数据表(CREATE TABLE) 语句，该语句包含了原数据表的结构，索引等。 复制以下命令显示的SQL语句，修改数据表名，并执行SQL语句，通过以上命令 将完全的复制数据表结构。 如果你想复制表的内容，你就可以使用 INSERT INTO ... SELECT 语句来实现。 实例 尝试以下实例来复制表 runoob_tbl 。

步骤一：

获取数据表的完整结构。

```
SHOW CREATE TABLE runoob_tbl \G;

*************************** 1. row ***************************
       Table: runoob_tbl
Create Table: CREATE TABLE `runoob_tbl` (
  `runoob_id` int(11) NOT NULL auto_increment,
  `runoob_title` varchar(100) NOT NULL default '',
  `runoob_author` varchar(40) NOT NULL default '',
  `submission_date` date default NULL,
  PRIMARY KEY  (`runoob_id`),
  UNIQUE KEY `AUTHOR_INDEX` (`runoob_author`)
) ENGINE=InnoDB 
1 row in set (0.00 sec)

ERROR:
No query specified
```

**步骤二：**

修改SQL语句的数据表名，并执行SQL语句。

```
mysql> CREATE TABLE `clone_tbl` (
  -> `runoob_id` int(11) NOT NULL auto_increment,
  -> `runoob_title` varchar(100) NOT NULL default '',
  -> `runoob_author` varchar(40) NOT NULL default '',
  -> `submission_date` date default NULL,
  -> PRIMARY KEY  (`runoob_id`),
  -> UNIQUE KEY `AUTHOR_INDEX` (`runoob_author`)
-> ) ENGINE=InnoDB;
Query OK, 0 rows affected (1.80 sec)
```

**步骤三：**

执行完第二步骤后，你将在数据库中创建新的克隆表 clone_tbl。 如果你想拷贝数据表的数据你可以使用 **INSERT INTO... SELECT** 语句来实现。

```
mysql> INSERT INTO clone_tbl (runoob_id,
    ->                        runoob_title,
    ->                        runoob_author,
    ->                        submission_date)
    -> SELECT runoob_id,runoob_title,
    ->        runoob_author,submission_date
    -> FROM runoob_tbl;
Query OK, 3 rows affected (0.07 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

执行以上步骤后，你将完整的复制表，包括表结构及表数据。

</br>

## MySQL 序列使用

MySQL序列是一组整数：1, 2, 3, ...，由于一张数据表只能有一个字段自增主键， 如果你想实现其他字段也实现自动增加，就可以使用MySQL序列来实现。

### 使用AUTO_INCREMENT

MySQL中最简单使用序列的方法就是使用 MySQL AUTO_INCREMENT 来定义列。

### 实例

以下实例中创建了数据表insect， insect中id无需指定值可实现自动增长。

```sql
mysql> CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, 
    -> date DATE NOT NULL,
    -> origin VARCHAR(30) NOT NULL
);
```

## MySQL 序列使用

MySQL序列是一组整数：1, 2, 3, ...，由于一张数据表只能有一个字段自增主键， 如果你想实现其他字段也实现自动增加，就可以使用MySQL序列来实现。

### 使用AUTO_INCREMENT

MySQL中最简单使用序列的方法就是使用 MySQL AUTO_INCREMENT 来定义列。

### 实例

以下实例中创建了数据表insect， insect中id无需指定值可实现自动增长。

```sql
mysql> CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, 
    -> date DATE NOT NULL,
    -> origin VARCHAR(30) NOT NULL
);
```

### 重置序列

如果你删除了数据表中的多条记录，并希望对剩下数据的AUTO_INCREMENT列进行重新排列，那么你可以通过删除自增的列，然后重新添加来实现。 不过该操作要非常小心，如果在删除的同时又有新记录添加，有可能会出现数据混乱。操作如下所示：

```sql
mysql> ALTER TABLE insect DROP id;
mysql> ALTER TABLE insect
    -> ADD id INT UNSIGNED NOT NULL AUTO_INCREMENT FIRST,
    -> ADD PRIMARY KEY (id);
```

### 设置序列的开始值

一般情况下序列的开始值为1，但如果你需要指定一个开始值100，那我们可以通过以下语句来实现：

```sql
mysql> CREATE TABLE insect
    -> (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY (id),
    -> name VARCHAR(30) NOT NULL, 
    -> date DATE NOT NULL,
    -> origin VARCHAR(30) NOT NULL
)engine=innodb auto_increment=100 charset=utf8;
```

或者你也可以在表创建成功后，通过以下语句来实现：

```sql
mysql> ALTER TABLE t AUTO_INCREMENT = 100;
```



## MySQL 处理重复数据

有些 MySQL 数据表中可能存在重复的记录，有些情况我们允许重复数据的存在，但有时候我们也需要删除这些重复的数据。

### 防止表中出现重复数据

- **PRIMARY KEY（主键）或者 UNIQUE（唯一）索引**

  你可以在MySQL数据表中设置指定的字段为 **PRIMARY KEY（主键）或者 UNIQUE（唯一）索引**来保证数据的唯一性。

  让我们尝试一个实例：下表中无索引及主键，所以该表允许出现多条重复记录。

  ```sql
  CREATE TABLE person_tbl
  (
  	id INT NOT NULL AUTO_INCREMENT,
  	first_name CHAR(20) NOT NULL,
  	last_name CHAR(20) NOT NULL,
  	sex CHAR(10),
  	PRIMARY KEY (id)
  );你设置了双主键，那么那个键的默认值不能为NULL，可设置为NOT NULL。如下所示：
  ```

  如果我们设置了唯一索引，那么在插入重复数据时，SQL语句将无法执行成功,并抛出错。

- INSERT IGNORE INTO

  `INSERT IGNORE INTO`与`INSERT INTO`的区别就是 `INSERT IGNORE` 会忽略数据库中已经存在的数据，如果数据库没有数据，就插入新的数据，如果有数据的话就跳过这条数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的。

  以下实例使用了INSERT IGNORE INTO，执行后不会出错，也不会向数据表中插入重复数据：

  ```sql
  mysql> INSERT IGNORE INTO person_tbl (last_name, first_name)
      -> VALUES( 'Jay', 'Thomas');
  Query OK, 1 row affected (0.00 sec)
  mysql> INSERT IGNORE INTO person_tbl (last_name, first_name)
      -> VALUES( 'Jay', 'Thomas');
  Query OK, 0 rows affected (0.00 sec)
  ```

  INSERT IGNORE INTO当插入数据时，在设置了记录的唯一性后，如果插入重复数据，将不返回错误，只以警告形式返回。 而REPLACE INTO into如果存在primary 或 unique相同的记录，则先删除掉。再插入新记录

- **REPLACE INTO**

  INSERT IGNORE INTO当插入数据时，在设置了记录的唯一性后，如果插入重复数据，将不返回错误，只以警告形式返回。 而 REPLACE INTO 如果存在primary 或 unique相同的记录，则先删除掉。再插入新记录

### 统计重复数据

以下我们将统计表中 first_name 和 last_name的重复记录数：

```sql
mysql> SELECT COUNT(*) as repetitions, last_name, first_name
    -> FROM person_tbl
    -> GROUP BY last_name, first_name
    -> HAVING repetitions > 1;
```

以上查询语句将返回 person_tbl 表中重复的记录数。 一般情况下，查询重复的值，请执行以下操作：

- 确定哪一列包含的值可能会重复。
- 在列选择列表使用COUNT(*)列出的那些列。
- 在GROUP BY子句中列出的列。（Group by 子句中的列必须出现在 Select 语句中）
- HAVING子句设置重复数大于1。

### 过滤重复数据

如果你需要读取不重复的数据可以在 SELECT 语句中使用 DISTINCT 关键字来过滤重复数据。

```sql
mysql> SELECT DISTINCT last_name, first_name
    -> FROM person_tbl;
```

你也可以使用 GROUP BY 来读取数据表中不重复的数据：

```sql
mysql> SELECT last_name, first_name
    -> FROM person_tbl
    -> GROUP BY (last_name, first_name);
```

### 删除重复数据

如果你想删除数据表中的重复数据，你可以使用以下的SQL语句：

```sql
mysql> CREATE TABLE tmp SELECT last_name, first_name, sex
    ->                  FROM person_tbl;
    ->                  GROUP BY (last_name, first_name, sex);
mysql> DROP TABLE person_tbl;
mysql> ALTER TABLE tmp RENAME TO person_tbl;
```

当然你也可以在数据表中添加 INDEX（索引） 和 PRIMAY KEY（主键）这种简单的方法来删除表中的重复记录。方法如下：

```sql
mysql> ALTER IGNORE TABLE person_tbl
    -> ADD PRIMARY KEY (last_name, first_name);
```

</br>

## 存储过程

示例：批量新增 user 表记录

```sql
-- 删除已存在的存储过程
drop procedure if exists create_user;

-- 设置定界符为“//”，因为 procedure 的语句中包含了默认的定界符“;”，
-- 定义 procedure 前需要重新定义定界符，
delimiter //
create procedure create_user(IN start int, IN end int)
begin
declare i int;
set i=start;
while i<=end do
	insert into user(username, password) values(concat('user', i), concat('password', i));
    set i=i+1;
end while;
end; //
```

调用存储过程

```sql
set @start=10, @end=21;
call create_user(@start, @end);
```





## 附录1: information_schema

`information_schema` 数据库是MySQL自带的，它提供了访问数据库`元数据`的方式。什么是元数据呢？元数据是关于数据的数据，如数据库名或表名，列的数据类型，或访问权限等。有些时候用于表述该信息的其他术语包括“数据词典”和“系统目录”。
在MySQL中，把 information_schema 看作是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权 限等。在INFORMATION_SCHEMA中，有数个只读表。它们实际上是视图，而不是基本表，因此，你将无法看到与之相关的任何文件。

**information_schema数据库表说明:**

`SCHEMATA`表：提供了当前mysql实例中所有数据库的信息。是show databases的结果取之此表。

`TABLES`表：提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪个schema，表类型，表引擎，创建时间等信息。是show tables from schemaname的结果取之此表。

`COLUMNS`表：提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。是show columns from schemaname.tablename的结果取之此表。

`STATISTICS`表：提供了关于表索引的信息。是show index from schemaname.tablename的结果取之此表。

USER_PRIVILEGES（用户权限）表：给出了关于全程权限的信息。该信息源自mysql.user授权表。是非标准表。

`SCHEMA_PRIVILEGES`（方案权限）表：给出了关于方案（数据库）权限的信息。该信息来自mysql.db授权表。是非标准表。

`TABLE_PRIVILEGES`（表权限）表：给出了关于表权限的信息。该信息源自mysql.tables_priv授权表。是非标准表。

`COLUMN_PRIVILEGES`（列权限）表：给出了关于列权限的信息。该信息源自mysql.columns_priv授权表。是非标准表。

`CHARACTER_SETS`（字符集）表：提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表。

`COLLATIONS`表：提供了关于各字符集的对照信息。

`COLLATION_CHARACTER_SET_APPLICABILITY`表：指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。

`TABLE_CONSTRAINTS`表：描述了存在约束的表。以及表的约束类型。

`KEY_COLUMN_USAGE`表：描述了具有约束的键列。

`ROUTINES`表：提供了关于存储子程序（存储程序和函数）的信息。此时，ROUTINES表不包含自定义函数（UDF）。名为“mysql.proc name”的列指明了对应于INFORMATION_SCHEMA.ROUTINES表的mysql.proc表列。

`VIEWS`表：给出了关于数据库中的视图的信息。需要有show views权限，否则无法查看视图信息。

`TRIGGERS`表：提供了关于触发程序的信息。必须有super权限才能查看该表



## 附录2: 问题汇总

Count(*) 和 count(1) 和 count(column) 的区别