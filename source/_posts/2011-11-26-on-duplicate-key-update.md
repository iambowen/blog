---
layout: post
title: mysql--on duplicate key update
tag: database
categories: ["mysql"]
---
On duplicate key update is to stop the duplication in unique index or primary
key, and try to update the row with new value.And it only functions with mysql
because it is not standard SQL grammar.

Create database and choose it

```bash
mysql> create database test;
Query OK, 1 row affected (0.00 sec)
mysql> use test;
Database changed
```

Create a table with primary key

```bash
mysql> create table pet(a CHAR(1), b CHAR(1), primary key(a));
Query OK, 0 rows affected (0.06 sec)
```

And initialize a row

```bash
mysql> insert into pet values('1', '2');
Query OK, 1 row affected (0.00 sec)
```

insert new value with the same primary key

```bash
mysql> insert into pet values('1', '3');
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```

drop primary key property

```bash
mysql> alter table pet drop PRIMARY KEY;
Query OK, 1 row affected (0.06 sec)
Records: 1  Duplicates: 0  Warnings: 0
```

insert again, cool, done

```bash
mysql> insert into pet values('1', '3');
Query OK, 1 row affected (0.00 sec)
```

what we have now? values with duplicated key

```bash
mysql> select * from pet;
+---+------+
| a | b    |
+---+------+
| 1 | 2    |
| 1 | 3    |
+---+------+
```

change the primary key back
```bash
mysql> alter table pet add primary key(b);
Query OK, 2 rows affected (0.11 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

insert the value with  on duplicate key update
```bash
mysql> insert into pet values('4', '3') on duplicate key update a = values(a);
Query OK, 2 rows affected (0.00 sec)
```

the result:
```bash
mysql> select * from pet;
+---+---+
| a | b |
+---+---+
| 1 | 2 |
| 4 | 3 |
+---+---+
2 rows in set (0.00 sec)
```
