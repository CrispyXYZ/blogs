---
layout: post
title: "MySQL 学习(2)：DISTINCT，WHERE，OR，NOT"
date: 2025-12-06 23:24:00 +0800
categories: [数据库]
math: false
mermaid: false
pin: false
tags: [MySQL, SQL入门, DISTINCT, WHERE, 练习题, 数据库查询, OR, NOT]
---

## 知识回顾

在开始新的学习之前，我们先来检验一下对第一篇内容的掌握情况。

Q1. 在 MySQL 中，表可以类比编程语言中的哪种数据结构？SQL 的表相比这种数据结构有什么特别之处？

<details>
<summary>A1. [点此展开]</summary>
<div markdown="block">
表可以类比编程语言中的结构体数组。不同的是，SQL 的表支持 `NULL` 值。
</div>
</details>

Q2. 创建表时，如果出现 `ERROR 1046 (3D000): No database selected` 错误，如何解决？

<details>
<summary>A2. [点此展开]</summary>
<div markdown="block">
在执行 SQL 语句之前，需要先选择数据库。使用命令 `USE database_name;` 选择数据库。或者使用 `CREATE DATABASE database_name;` 创建一个新数据库并选择它。
</div>
</details>

Q3. 写出创建Persons表的SQL语句，该表包含以下字段：

- id (整数类型)
- last_name (最大长度255的字符串)
- first_name (最大长度255的字符串)
- address (最大长度255的字符串)
- city (最大长度255的字符串)

<details>
<summary>A3. [点此展开]</summary>
<div markdown="block">

```sql
CREATE TABLE Persons (
    id int,
    last_name varchar(255),
    first_name varchar(255),
    address varchar(255),
    city varchar(255)
);
```

请注意用的是小括号，而不是大括号。

</div>
</details>

Q4. 向Persons表插入数据的两种基本语法是什么？哪种情况下可以省略列名列表？

<details>
<summary>A4. [点此展开]</summary>
<div markdown="block">

```sql
-- 插入单行
INSERT INTO TableName (Col1, Col2, ...)
VALUES (Value1, Value2, ...);

-- 插入多行
INSERT INTO TableName (Col1, Col2, ...)
VALUES
    (Value11, Value12, ...),
    (Value21, Value22, ...),
    ...;
```

当插入所有列的值，且值的顺序与表中列的顺序完全一致时，可以省略 `(Col1, Col2, ...)` 部分。

</div>
</details>

Q5. 如何查看数据库中所有表的列表？如何查看某个表的详细结构？

<details>
<summary>A5. [点此展开]</summary>
<div markdown="block">

查看数据库中所有表的列表：

```sql
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESCRIBE TableName;
-- 或者
DESC TableName;
```

</div>
</details>

Q6. 如何从Persons表中查询所有人的last_name和first_name？如果查询所有列呢？

<details>
<summary>A6. [点此展开]</summary>
<div markdown="block">

```sql
-- 查询所有人的last_name和first_name
SELECT last_name, first_name FROM Persons;

-- 查询所有列
SELECT * FROM Persons;
```

</div>
</details>

Q7. 为什么SELECT语句的列名列表不需要加圆括号，而INSERT语句需要？

<details>
<summary>A7. [点此展开]</summary>
<div markdown="block">

可以这样理解语法设计：

- `SELECT Col1, Col2, ...` 中的列名列表类似于编程语言中的变量名，或者是不定长参数
- `INSERT INTO TableName (Col1, Col2, ...)` 中的列名列表类似于编程语言中的形参列表，或者是一个元组(tuple)

事实上，应该这样理解：

- `SELECT Col1, Col2, ...` 中的 `ColN` 其实是表达式，并不一定必须是列名
- `INSERT INTO TableName (Col1, Col2, ...)` 中的列名列表必须是表中的列名，不能是表达式

</div>
</details>

## 新知识

### `DISTINCT` - 返回唯一值

`DISTINCT` 关键字用于返回唯一的值

语法：

```sql
SELECT DISTINCT column1, column2,... FROM table_name;
```

事实上，`DISTINCT` 关键字可以与 `SELECT`、`UPDATE`、`DELETE` 语句一起使用，用于对结果集进行去重。

我们运行看看：

```sql
mysql> INSERT INTO Persons VALUES (4, "Wu", "Wang", "West Xingang Road 135", "Guangzhou");
Query OK, 1 row affected (0.05 sec)

mysql> SELECT * FROM Persons;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
4 rows in set (0.01 sec)

mysql> SELECT DISTINCT city FROM Persons;
+-----------+
| city      |
+-----------+
| Weihai    |
| NULL      |
| Guangzhou |
+-----------+
3 rows in set (0.00 sec)
```

### `WHERE` - 条件过滤

`WHERE` 子句用于对结果集进行条件过滤，只返回满足指定条件的行语法如下：

```sql
SELECT column1, column2,... FROM table_name WHERE condition;
```

`WHERE` 后面的 `condition` 可以是任何表达式，包括比较运算符、逻辑运算符、函数、子查询等。

常用运算符有：

| 运算符 | 说明 |
| --- | --- |
| = | 等于 |
| <> | 不等于 |
| > | 大于 |
| < | 小于 |
| >= | 大于等于 |
| <= | 小于等于 |
| BETWEEN | 在某个范围内 |
| LIKE | 模糊匹配 |
| IN | 匹配列表中的值 |
| IS NULL | 为空 |
| IS NOT NULL | 不为空 |

我们运行看看：

```sql
mysql> SELECT * FROM Persons WHERE city="Weihai";
+------+-----------+------------+----------------------+--------+
| id   | last_name | first_name | address              | city   |
+------+-----------+------------+----------------------+--------+
|    1 | San       | Zhang      | West Wenhua Road 2   | Weihai |
|    2 | Si        | Li         | West Wenhua Road 180 | Weihai |
+------+-----------+------------+----------------------+--------+
2 rows in set (0.01 sec)
```

`WHERE` 子句还可以与 `DISTINCT` 关键字一起使用，用于对结果集进行去重，此处省略。

### `AND` / `OR` / `NOT` - 逻辑运算符

`AND`、`OR`、`NOT` 三个逻辑运算符用于对条件进行逻辑组合：

- `AND`（与）：所有条件都为真时返回结果。
- `OR`（或）：只要其中一个条件为真时返回结果。
- `NOT`（非）：取反，返回不满足条件的行。

我们运行看看：

```sql
mysql> SELECT * FROM Persons;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
4 rows in set (0.01 sec)

mysql> INSERT INTO Persons (id, last_name, first_name, city)
    -> VALUES
    -> (5, "Si", "Wang", "Weihai"),
    -> (6, "San", "Li", "Guangzhou");
Query OK, 2 rows affected (0.07 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM Persons WHERE ((address IS NOT NULL) AND (first_name="Li")) OR (NOT city="Guangzhou");
+------+-----------+------------+----------------------+--------+
| id   | last_name | first_name | address              | city   |
+------+-----------+------------+----------------------+--------+
|    1 | San       | Zhang      | West Wenhua Road 2   | Weihai |
|    2 | Si        | Li         | West Wenhua Road 180 | Weihai |
|    5 | Si        | Wang       | NULL                 | Weihai |
+------+-----------+------------+----------------------+--------+
3 rows in set (0.01 sec)
```

注：
`id=3`（张四）为什么没被选中：`city` 为 `NULL`，`NULL = "Guangzhou"` 的结果是 `UNKNOWN`，但 `NOT UNKNOWN` 仍然是 `UNKNOWN`，在 `WHERE` 条件中视为 `false`

## 练习题

数据集：
```sql
mysql> SELECT * FROM Persons;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
|    5 | Si        | Wang       | NULL                  | Weihai    |
|    6 | San       | Li         | NULL                  | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
6 rows in set (0.02 sec)
```

Q1. 请写出以下 SQL 语句：

- 选择 `Persons` 表中 `first_name` 为 `Zhang` 的所有行。
- 选择 `Persons` 表中 `first_name` 为 `Zhang` 或 `Li` 的所有行。
- 选择 `Persons` 表中 `first_name` 为 `Zhang` 或 `Li` 的所有行，且 `address` 不为空。

Q2. 请写出以下 SQL 语句：

- 选择唯一的 `city` 值。

Q3. 请写出以下 SQL 语句：

- 选择满足以下条件的唯一的 `city` 值：`first_name` 为 `Zhang` 或 `Li`，且 `city` 不为空。（注：`DISTINCT` 关键字在 `WHERE` 子句之后执行）

答案下期更新。
