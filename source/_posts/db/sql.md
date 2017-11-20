---
categories: 数据库
date: 2017-11-21 7:00
title: SQL 常用语句
---

## SQL UNION 和 UNION ALL 

**UNION** 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意:

- UNION 内部的 SELECT 语句必须拥有**相同数量的列。列也必须拥有相似的数据类型**。
- 同时，每条 SELECT 语句中的**列的顺序必须相同**。
- 另外，UNION 结果集中的列名总是等于 UNION 中**第一个 SELECT 语句中的列名**。
- 区别：默认地，UNION 操作符选取不同的值。如果**允许重复的值**，请使用 UNION ALL。

### SQL UNION 语法

```sql
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
```

### SQL UNION ALL 语法

```sql
SELECT column_name(s) FROM table_name1
UNION ALL
SELECT column_name(s) FROM table_name2
```



**下面的例子中使用的原始表：**

Employees_China:

| E_ID | E_Name         |
| ---- | -------------- |
| 01   | Zhang, Hua     |
| 02   | Wang, Wei      |
| 03   | Carter, Thomas |
| 04   | Yang, Ming     |

Employees_USA:

| E_ID | E_Name         |
| ---- | -------------- |
| 01   | Adams, John    |
| 02   | Bush, George   |
| 03   | Carter, Thomas |
| 04   | Gates, Bill    |

> 使用 Union 会合并结果集并去除一条重复的记录 "Carter, Thomas" ，最终得到 7 条记录
>
> 视同 Union 会合并结果集的所有记录，包括重复记录，最终得到 8 条记录。

</br>

​	

## SQL SELECT INTO

**SELECT INTO 语句可用于创建表的备份复件。**

- SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。
- SELECT INTO 语句常用于创建表的备份复件或者用于对记录进行存档。

### SQL SELECT INTO 语法

您可以把所有的列插入新表：

```sql
SELECT *
INTO new_table_name [IN externaldatabase] 
FROM old_tablename
```

或者只把希望的列插入新表：

```sql
SELECT column_name(s)
INTO new_table_name [IN externaldatabase] 
FROM old_tablename
```

### **SELECT INTO —— 带有表连接和 WHERE 子句**

```sql
SELECT Persons.LastName,Orders.OrderNo
INTO Persons_Order_Backup
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
```

</br>



## SQL CREATE DATABASE 

CREATE DATABASE 语句用于创建数据库。

### SQL CREATE DATABASE 语法

```sql
CREATE DATABASE database_name
```



## SQL CREATE TABLE 语句

CREATE TABLE 语句用于创建数据库中的表。

### SQL CREATE TABLE 语法

```sql
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)
```

**SQL CREATE TABLE 实例**

本例演示如何创建名为 "Person" 的表。

该表包含 5 个列，列名分别是："Id_P"、"LastName"、"FirstName"、"Address" 以及 "City"：

```sql
CREATE TABLE Persons
(
Id_P int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

</br>



## SQL 约束 (Constraints)

**约束用于限制加入表的数据的类型。**

可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

我们将主要探讨以下几种约束：

- NOT NULL
- UNIQUE
- PRIMARY KEY
- FOREIGN KEY
- CHECK
- DEFAULT

> 注释：下面详细讲解每一种约束。

## SQL NOT NULL 约束

**NOT NULL 约束强制列不接受 NULL 值。**

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，**就无法插入新记录或者更新记录**。

下面的 SQL 语句强制 "Id_P" 列和 "LastName" 列不接受 NULL 值：

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

</br>



## SQL UNIQUE 约束

**UNIQUE 约束唯一标识数据库表中的每条记录。**

- UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。
- PRIMARY KEY 拥有自动定义的 UNIQUE 约束。
- 请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

</br>

### SQL UNIQUE Constraint on CREATE TABLE

下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 UNIQUE 约束：

#### MySQL:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
)
```

#### SQL Server / Oracle / MS Access:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束，请使用下面的 SQL 语法：

#### MySQL / SQL Server / Oracle / MS Access:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
)
```

</br>

### SQL UNIQUE Constraint on ALTER TABLE

当表已被创建时，如需在 "Id_P" 列创建 UNIQUE 约束，请使用下列 SQL：

#### MySQL / SQL Server / Oracle / MS Access:

```sql
ALTER TABLE Persons
ADD UNIQUE (Id_P)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

#### MySQL / SQL Server / Oracle / MS Access:

```sql
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
```
</br>



### 撤销 UNIQUE 约束

如需撤销 UNIQUE 约束，请使用下面的 SQL：

#### MySQL:

```sql
ALTER TABLE Persons
DROP INDEX uc_PersonID
```

#### SQL Server / Oracle / MS Access:

```sql
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```

</br>



## SQL PRIMARY KEY 约束

**PRIMARY KEY 约束唯一标识数据库表中的每条记录。**

- 主键必须包含唯一的值。
- 主键列不能包含 NULL 值。
- 每个表都应该有一个主键，并且每个表只能有一个主键。

### SQL PRIMARY KEY Constraint on CREATE TABLE

下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 PRIMARY KEY 约束：

#### MySQL:

```Sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (Id_P)
)
```

### SQL Server / Oracle / MS Access:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

#### MySQL / SQL Server / Oracle / MS Access:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
```

</br>

### SQL PRIMARY KEY Constraint on ALTER TABLE

如果在表已存在的情况下为 "Id_P" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：

#### MySQL / SQL Server / Oracle / MS Access:

```Sql
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

#### MySQL / SQL Server / Oracle / MS Access:

```Sql
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```

注释：如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）。

</br>



## SQL DEFAULT 约束

**SQL DEFAULT 约束**

- DEFAULT 约束用于向列中插入默认值。
- 如果没有规定其他的值，那么会将默认值添加到所有的新记录。

### SQL DEFAULT Constraint on CREATE TABLE

下面的 SQL 在 "Persons" 表创建时为 "City" 列创建 DEFAULT 约束：

#### My SQL / SQL Server / Oracle / MS Access:

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)
```

通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值：

```sql
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
OrderDate date DEFAULT GETDATE()
)
```

</br>

### SQL DEFAULT Constraint on ALTER TABLE

如果在表已存在的情况下为 "City" 列创建 DEFAULT 约束，请使用下面的 SQL：

#### MySQL:

```
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

##### SQL Server / Oracle / MS Access:

```
ALTER TABLE Persons
ALTER COLUMN City SET DEFAULT 'SANDNES'
```

</br>

### 撤销 DEFAULT 约束

如需撤销 DEFAULT 约束，请使用下面的 SQL：

#### MySQL:

```
ALTER TABLE Persons
ALTER City DROP DEFAULT
```

#### SQL Server / Oracle / MS Access:

```
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```