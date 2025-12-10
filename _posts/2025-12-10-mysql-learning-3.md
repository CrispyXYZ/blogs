---
layout: post
title: "MySQL 学习(3)：LIKE、IN、ORDER BY、LIMIT"
date: 2025-12-10 23:07:00 +0800
categories: [数据库]
math: false
mermaid: false
pin: false
tags: [MySQL, SQL入门, LIKE, IN, ORDER BY, LIMIT]
---

## 上期习题解答

```sql
mysql> SELECT * FROM Persons WHERE first_name="Zhang"; -- Q1.1
+------+-----------+------------+--------------------+--------+
| id   | last_name | first_name | address            | city   |
+------+-----------+------------+--------------------+--------+
|    1 | San       | Zhang      | West Wenhua Road 2 | Weihai |
|    3 | Si        | Zhang      | NULL               | NULL   |
+------+-----------+------------+--------------------+--------+
2 rows in set (0.01 sec)

mysql> SELECT * FROM Persons WHERE first_name="Zhang" OR first_name="Li"; -- Q1.2
+------+-----------+------------+----------------------+-----------+
| id   | last_name | first_name | address              | city      |
+------+-----------+------------+----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2   | Weihai    |
|    2 | Si        | Li         | West Wenhua Road 180 | Weihai    |
|    3 | Si        | Zhang      | NULL                 | NULL      |
|    6 | San       | Li         | NULL                 | Guangzhou |
+------+-----------+------------+----------------------+-----------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM Persons WHERE (first_name="Zhang" OR first_name="Li") AND address IS NOT NULL; -- Q1.3
+------+-----------+------------+----------------------+--------+
| id   | last_name | first_name | address              | city   |
+------+-----------+------------+----------------------+--------+
|    1 | San       | Zhang      | West Wenhua Road 2   | Weihai |
|    2 | Si        | Li         | West Wenhua Road 180 | Weihai |
+------+-----------+------------+----------------------+--------+
2 rows in set (0.00 sec)

mysql> SELECT DISTINCT city FROM Persons; -- Q2
+-----------+
| city      |
+-----------+
| Weihai    |
| NULL      |
| Guangzhou |
+-----------+
3 rows in set (0.01 sec)

mysql> SELECT DISTINCT city FROM Persons WHERE (first_name="Zhang" OR first_name="Li") AND city IS NOT NULL; -- Q3
+-----------+
| city      |
+-----------+
| Weihai    |
| Guangzhou |
+-----------+
2 rows in set (0.00 sec)
```

## 新内容

### `LIKE` 运算符 - 模糊匹配

`LIKE` 运算符用于模糊匹配，语法：

```sql
SELECT ... FROM ... WHERE col LIKE pattern;
```

也有否定形式：

```sql
SELECT ... FROM ... WHERE col NOT LIKE pattern;
```

其中 `pattern` 遵循以下规则：

- `%`：匹配任意字符串，包括空字符串。
- `_`：匹配单个字符。

MySQL 的 `LIKE` 默认不区分大小写，如果需要区分大小写，可以使用 `BINARY` 关键字。

MySQL 也支持正则表达式匹配，即 `REGEXP` 和 `RLIKE`，语法与 `LIKE` 相同，但使用正则表达式进行匹配，此处不做详细介绍。

我们运行看看：

```sql
mysql> SELECT * FROM Persons WHERE first_name LIKE "%ang";
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
|    5 | Si        | Wang       | NULL                  | Weihai    |
+------+-----------+------------+-----------------------+-----------+
4 rows in set (0.02 sec)

mysql> SELECT * FROM Persons WHERE address LIKE "West%Road 1__";
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
2 rows in set (0.00 sec)
```

### `IN` 运算符 - 多值匹配

`IN` 运算符是用于简化多值比较的工具，它可以替代多个 `OR` 条件，使查询更加简洁高效。语法：

```sql
SELECT ... FROM ... WHERE col IN (value1, value2, ...);
SELECT ... FROM ... WHERE col IN (SELECT ...);
```

它也有其否定形式，即 `NOT IN`。

此处不再展示运行结果。

### `ORDER BY` - 排序

`ORDER BY` 子句用于对结果集进行排序，默认按照升序排序，也可以使用 `DESC` 关键字进行降序排序。语法：

```sql
SELECT ... FROM ... ORDER BY col1 [ASC|DESC], col2 [ASC|DESC],...;
```

- `col1`、`col2` 等表示要排序的列名，可以指定多个列进行排序。
- `ASC` 表示升序排序，`DESC` 表示降序排序。

注：这里的 `colN` 是表达式，这与第二期习题 Q7 中所述相符，不同的是，这里的 `colN` 也可以是列编号，例如 `ORDER BY 2` 表示按照第二列进行排序。

我们运行看看：

```sql
mysql> SELECT * FROM Persons ORDER BY 1;
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
6 rows in set (0.00 sec)

mysql> SELECT * FROM Persons ORDER BY 1 DESC;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    6 | San       | Li         | NULL                  | Guangzhou |
|    5 | Si        | Wang       | NULL                  | Weihai    |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
+------+-----------+------------+-----------------------+-----------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM Persons ORDER BY 2;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    6 | San       | Li         | NULL                  | Guangzhou |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    5 | Si        | Wang       | NULL                  | Weihai    |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM Persons ORDER BY 2,3;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    6 | San       | Li         | NULL                  | Guangzhou |
|    1 | San       | Zhang      | West Wenhua Road 2    | Weihai    |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
|    5 | Si        | Wang       | NULL                  | Weihai    |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
+------+-----------+------------+-----------------------+-----------+
6 rows in set (0.00 sec)
```

### `LIMIT` - 限制结果集

`LIMIT` 子句用于限制结果集的数量，语法：

```sql
SELECT ... FROM ... LIMIT [offset,] count;
```

或者

```sql
SELECT ... FROM ... LIMIT count OFFSET offset;
```

- `offset` 表示偏移量，即跳过的行数，默认为 0。
- `count` 表示返回记录的数量。

比较简单，我们运行看看：

```sql
mysql> SELECT * FROM Persons ORDER BY 1 DESC LIMIT 5;
+------+-----------+------------+-----------------------+-----------+
| id   | last_name | first_name | address               | city      |
+------+-----------+------------+-----------------------+-----------+
|    6 | San       | Li         | NULL                  | Guangzhou |
|    5 | Si        | Wang       | NULL                  | Weihai    |
|    4 | Wu        | Wang       | West Xingang Road 135 | Guangzhou |
|    3 | Si        | Zhang      | NULL                  | NULL      |
|    2 | Si        | Li         | West Wenhua Road 180  | Weihai    |
+------+-----------+------------+-----------------------+-----------+
5 rows in set (0.02 sec)
```
