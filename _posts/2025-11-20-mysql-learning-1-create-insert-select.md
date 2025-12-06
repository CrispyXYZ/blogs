---
layout: post
title: "MySQL 学习(1)：表，CREATE，INSERT，SELECT"
date: 2025-11-20 23:10:00 +0800
categories: [数据库]
math: true
mermaid: false
pin: false
tags: [MySQL, SQL入门, CREATE, INSERT, SELECT]
---

本文是对 [SQL入门教程 - CSDN](https://blog.csdn.net/shanguicsdn000/article/details/142332546) 的学习记录。

## 表的概念

一个表可以理解为一个 `struct[]` 数据结构。

例如下面的名为 `Persons` 的表

|id |last_name  |first_name |address        |city       |
|-  |-          |-          |-              |-          |
|1  |Adams      |John       |Oxford Street  |London     |
|2  |Bush       |George     |Fifth Avenue   |New York   |
|3  |Carter     |Thomas     |Changan Street |Beijing    |

就相当于

```c
struct {
    int id;
    char last_name[255];
    char first_name[255];
    char address[255];
    char city[255];
} persons[3];
```

与之不同的是，SQL 的表支持 `NULL` 值。

## `CREATE TABLE` - 创建表

```sql
CREATE TABLE TableName (
    ColName1 DataType1,
    ColName2 DataType2,
    ColName3 DataType3,
    ...
);
```

常用的数据类型此处不再列出。

创建上面的 `Persons` 表，语句如下

```sql
CREATE TABLE Persons (
    id int,
    last_name varchar(255),
    first_name varchar(255),
    address varchar(255),
    city varchar(255)
);
```

运行，发现出错

```sql
mysql> CREATE TABLE Persons (
    ->     id int,
    ->     last_name varchar(255),
    ->     first_name varchar(255),
    ->     address varchar(255),
    ->     city varchar(255)
    -> )
    -> ;
ERROR 1046 (3D000): No database selected
```

显示我们还没有选中数据库，所以我们先选中一下（也可以用 `CREATE DATABASE database_name;` 创建数据库），再创建表：

```sql
mysql> SHOW databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| learn_mysql        |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.05 sec)

mysql> USE learn_mysql;
Database changed
mysql> CREATE TABLE Persons (
    ->     id int,
    ->     last_name varchar(255),
    ->     first_name varchar(255),
    ->     address varchar(255),
    ->     city varchar(255)
    -> );
Query OK, 0 rows affected (0.09 sec)
```

进行查看：

```sql
mysql> SHOW TABLES;
+-----------------------+
| Tables_in_learn_mysql |
+-----------------------+
| Persons               |
+-----------------------+
1 row in set (0.02 sec)

mysql> DESCRIBE Persons;
+------------+--------------+------+-----+---------+-------+
| Field      | Type         | Null | Key | Default | Extra |
+------------+--------------+------+-----+---------+-------+
| id         | int          | YES  |     | NULL    |       |
| last_name  | varchar(255) | YES  |     | NULL    |       |
| first_name | varchar(255) | YES  |     | NULL    |       |
| address    | varchar(255) | YES  |     | NULL    |       |
| city       | varchar(255) | YES  |     | NULL    |       |
+------------+--------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

## `INSERT` - 插入数据

```sql
INSERT INTO TableName (Col1, Col2, ...)
VALUES (Value1, Value2, ...);

INSERT INTO TableName (Col1, Col2, ...)
VALUES
    (Value11, Value12, ...),
    (Value21, Value22, ...),
    ...;
```

当 `(Col1, Col2, ...)` 是表中所有列时，可以省略

我们运行看看效果：

```sql
mysql> INSERT INTO Persons
    -> VALUES
    -> (1,'San','Zhang','West Wenhua Road 2', 'Weihai'),
    -> (2,'Si','Li','West Wenhua Road 180', 'Weihai');
Query OK, 2 rows affected (0.05 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> INSERT INTO Persons (id,last_name,first_name)
    -> VALUES (3,'Si','Zhang')
    -> ;
Query OK, 1 row affected (0.02 sec)
```

插入之后，怎么查询呢？这就要用到 `SELECT` 语句了。

## `SELECT` - 查询数据

```sql
SELECT ColName1,ColName2,... FROM TableName;
```

其中 `ColName1,ColName2,...` 可以为 `*` ，表示选取所有列。

```sql
mysql> SELECT * FROM Persons;
+------+-----------+------------+----------------------+--------+
| id   | last_name | first_name | address              | city   |
+------+-----------+------------+----------------------+--------+
|    1 | San       | Zhang      | West Wenhua Road 2   | Weihai |
|    2 | Si        | Li         | West Wenhua Road 180 | Weihai |
|    3 | Si        | Zhang      | NULL                 | NULL   |
+------+-----------+------------+----------------------+--------+
3 rows in set (0.00 sec)

mysql> SELECT last_name,first_name FROM Persons;
+-----------+------------+
| last_name | first_name |
+-----------+------------+
| San       | Zhang      |
| Si        | Li         |
| Si        | Zhang      |
+-----------+------------+
3 rows in set (0.02 sec)
```

这里我们注意到 `SELECT` 语句的列表不需要加圆括号，而 `INSERT` 语句的列表需要加圆括号，为什么这么设计？

事实上，我们可以把圆括号看作是元组 `tuple`，没加圆括号的看作是不定长参数 `args...`，那么语法作如下化简：

```sql
SELECT ColArgs... FROM TableName;

INSERT INTO TableName ColTuple
VALUES TupleArgs...
```

这样逻辑就清晰一些了。