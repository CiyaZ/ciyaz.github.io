# 数据定义语句

SQL分为DML（数据操作语句）和DDL（数据定义语句），DML的常见用法前文已经叙述过。这篇笔记主要是针对DDL语句。

注意：由于各种数据库DDL语法和支持情况可能不同，这篇笔记使用MySQL语法。

## CREATE DATABASE

创建数据库。

```sql
CREATE DATABASE 数据库名
```

通常使用数据库的第一件事是创建数据库，然后才能创建数据表，增删改查等。

开发时通常有这样的需求，如果数据库存在就先删除再创建，不存在就直接创建：

```sql
/*适用于MySQL*/
drop database if exists 数据库名;
create database 数据库名;
```

注意：这个例子可能不同数据库写法不同，需要具体问题具体分析。

## CREATE TABLE

创建数据表

```sql
CREATE TABLE 表名称
(
    列名称1 数据类型,
    列名称2 数据类型,
    列名称3 数据类型,
    ...
)
```

注意：

1. 不同的数据库规定的数据类型都不同，具体根据所使用数据库和编程语言进行选择。
2. 注意创建数据表的顺序，表之间有复杂的外键约束，如果先创建的表引用了不存在的表中列，就会报错。

## 约束

### NOT NULL

该列不接受NULL值，如果插入是该列为空，不允许插入。例子：

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

### UNIQUE

该列数据具有唯一性，插入时如果已经存在相同数据，不允许插入。

定义一列的UNIQUE约束：

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

使用CONSTRAINT定义UNIQUE约束（可以起别名，定义多列）：

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

### PRIMARY KEY

主键约束默认是NOT NULL UNIQUE的。定义主键约束：

```sql
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

使用CONSTRAINT定义主键约束：

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

注意：一个表主键可以是多个字段，但是只有一个PRIMARY KEY约束。实际上不需要多个字段为主键，通常使用自增长、无实际意义的数字或GUID作为逻辑主键。

### CHECK

CHECK约束用于限制列的取值范围。

```sql
CREATE TABLE Persons
(
    Id_P int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CHECK (Id_P>0)
)
```

使用CONSTRAINT定义CHECK约束：

```sql
CREATE TABLE Persons
(
    Id_P int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
)
```

### DEFAULT

DEFAULT约束用于在插入数据时某列为空的情况下，向该列插入默认值。例子：

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

使用函数插入默认值：

```sql
CREATE TABLE Orders
(
    Id_O int NOT NULL,
    OrderNo int NOT NULL,
    Id_P int,
    OrderDate date DEFAULT GETDATE()
)
```

注意：通常把默认值插入的代码方法外层的编程语言中，能够更加符合业务需求。

### FOREIGN KEY

外键约束。一个表的外键指向另一个表的主键，外键用于连接两个表，执行子查询或连接查询。定义外键例子：

```sql
CREATE TABLE Orders
(
    Id_O int NOT NULL,
    OrderNo int NOT NULL,
    Id_P int,
    PRIMARY KEY (Id_O),
    FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
)
```

使用CONSTRAINT定义外键约束：

```sql
CREATE TABLE Orders
(
    Id_O int NOT NULL,
    OrderNo int NOT NULL,
    Id_P int,
    PRIMARY KEY (Id_O),
    CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
)
```

## CREATE INDEX

用于在表中创建索引，在不读取整个表的情况下，索引可以使数据库更快的查找数据。用户无法看到索引，它只能用来加速查询。但是，创建索引后，INSERT，DELETE，UPDATE操作（增删改）所用的时间也更多，因为处理数据表的同时也要处理索引。

```sql
CREATE INDEX 索引名
ON 表名 (列名)
```

注意：

* 列可以是多个
* 可以使用UNIQUE指定索引（CREATE UNIQUE INDEX 索引名 ON 表名 （列名）），意味着索引的行不能有相同值。

## DROP/TRUNCATE

DROP可以删除数据库，数据表，列，索引，TRUNCATE用于清空数据表内数据，但不删除数据表本身。

```sql
DROP DATABASE 数据库名称
DROP TABLE 表名称
TRUNCATE TABLE 表名称
ALTER TABLE 表名称 DROP INDEX 索引名
```

### ALTER TABLE

ALTER TABLE用于在已有的表中，添加、修改或删除列。

```sql
/*添加列*/
ALTER TABLE table_name
ADD column_name datatype
/*删除列*/
ALTER TABLE table_name
DROP COLUMN column_name
/*更改列数据类型*/
ALTER TABLE table_name
ALTER COLUMN column_name datatype
```

注意：在软件的开发阶段，不会用到ALTER语句，该语句通常用于数据库管理员根据业务需求，对已经上线的项目进行小幅修改。

## AUTO INCREMENT

设置该字段自增长。

```sql
/*仅适用于MySQL*/
CREATE TABLE Persons
(
    P_Id int NOT NULL AUTO_INCREMENT,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255),
    PRIMARY KEY (P_Id)
)
```

## VIEW

视图是基于SQL结果集的可视化表，视图包含行和列，就像一个真实存在的表，我们可以在视图中增删改查，但是视图并不真实对应该数据库中任何表，而是来自于一个查询结果集。创建视图，也完全不会影响到数据库表本身的结构。

### CREATE VIEW

创建视图。

```sql
CREATE VIEW 视图名 AS
SELECT 列名(s)
FROM 表名
WHERE 条件
```

注意：AS后接一个完整的SELECT查询语句。

### DROP VIEW

删除视图。

```sql
DROP VIEW 视图名
```
