---
layout: post
title: "Export/Import database/procedures in  mysql"
description: ""
category: 
tags: [mysql]
---


最近在做一个关于[Docker](http://docker.io)的spike的时候，遇到了这样的一个需求。我需要在Docker build Mysql服务的image的时候完成两个数据库的schema migration，不包含数据的migration. 其中一个数据库只有表，另外一个数据库还包括了store procedures和函数.
假设只有表的数据库叫做table_only, 有函数和store procedure的数据库叫做table_with_sp，数据在的主机名/地址为 host1，目标mysql服务器为host2。

#####Step 1: 导出schema/store procedures/functions

```bash
mysqldump -h host1 -u root -p --no-data table_only > table_only.sql
mysqldump -h host1 -u root -p --no-data --routines table_with_sp > table_with_sp.sql
```

#####Step 2: Migration
```bash
mysql -h host2 -u root -p -e "create database table_only; create database table_with_sp" 
mysql -h host2 -u root -p table_only < table_only.sql
mysql -h host2 -u root -p table_with_sp < table_with_sp.sql
```

####Notes
- 不想用任何命令的，可以用可视化的数据库工具进行导出，Mac下可以使用[SequelPro]()，非常方便，是免费的工具。
- 如果想自动化整个过程，可以考虑对host2的mysql 配置做如下的修改，让mysqldump的时候自动使用如下的authentication. 

```bash
[mysqldump]
user=root
password=password
```



