---
layout: post
title: "Be careful when you playing with history database"
description: ""
category: database 
tags: [mysql, database]
---



When we talking about History database, we are generally saying the database that deposit the actions performed on other databases by application. It is freaking important for recording the critical action that performed by users in both  business and development point of view.  With the help of ChangeLog database, we can easily track that an purchased transaction has or has not happened at certain time, thus by querying this table, we will 100% sure if the user wants to confirm with the transaction.

A typical table, say ChangeLog in History database, would have such kind of schema:

```bash
mysql> show create table ChangeLog\G
*************************** 1. row ***************************
       Table: ChangeLog
Create Table: CREATE TABLE `ChangeLog` (
  `ID` bigint(40) NOT NULL AUTO_INCREMENT,
  `TableName` varchar(40) DEFAULT NULL,
  `ForeignKey` varchar(40) DEFAULT NULL,
  `Who` varchar(255) DEFAULT NULL,
  `Field` varchar(40) DEFAULT NULL,
  `Mod` int(11) NOT NULL DEFAULT '0',
  `OldValue` text,
  `NewValue` text,
  `Comments` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`ID`,`Mod`),
  KEY `idx1` (`TableName`,`ForeignKey`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 
```
The schema shows that it would record action done on some table by someone from certain value to other value on certain time. Two columns, TableName and ForeignKey, are indexed, which would be faster if you take those two in the where clause. The thing you need to aware is that index of ForeignKey is only working when TableName is also existed in the where clause.

History database is normally huge and rarely has replica, as we can imagine. The reason, I suspect, is the changing-write/read operation much more frequently than other tables. Normally, it has several following characters:


  1. Query/Scan/Join operation could cause performance on database for it has large bunch of data;
  2. It's pretty pretty important that I don’t need to talk about this twice;
  3. Things in the history database is genuine, application or log can fake but the history table does not lie;
  4. Since it’s might be the only database, clearly it’s not 

Due to the importance of History database, few people would have access. But there gonna be chance that you would talk to it. And the following are the tips I learned from a senior developer about interacting with History database:


  1. Pair with someone who are experienced and can make sure the query you write would not do harm to the database;
  2. Check the ChangeLog table schema first, attention whether it has some columns indexed;
  3. Use “--“  in the beginning to comment out your query in case if you hit enter before you complete the query;
  4. To “explain" the query you want to run firstly, to see whether the query would happen in a very large dataset, if it is, then you need to consider the consequent for running the query.

```bash
mysql> Explain select * from ChangeLog where TableName = '' and ForeignKey = '';
+----+-------------+-----------+------+---------------+------+---------+-------------+------+-------------+
| id | select_type | table     | type | possible_keys | key  | key_len | ref         | rows | Extra       |
+----+-------------+-----------+------+---------------+------+---------+-------------+------+-------------+
|  1 | SIMPLE      | ChangeLog | ref  | idx1          | idx1 | 86      | const,const |   61 | Using where |
+----+-------------+-----------+------+---------------+------+---------+-------------+------+-------------+
1 row in set (0.47 sec)
```
What we can see from the result of explain is that the query would happen only in 61 rows, which would barely cause performance issue on the database. Of course, good to go.