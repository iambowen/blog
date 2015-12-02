---
layout: post
title: "mysql access control"
description: "mysql"
category: mysql
tags: [mysql]
---

连接到Mysql的请求被接受或者拒绝的条件是:

1. 是否通过身份验证
2. 账号是否被锁定

身份验证利用了`mysq.user`表中的`user`,`host`和`password`字段，账号锁定与否是用`account_locked`
字段进行判断。身份的内容基于连接的客户端的主机和连接时使用的用户名。

当user字段为空时，意味着它可以匹配任意用户，不为空时，只有用户名和主机名都匹配时，才能进入下一个验证
的阶段。连接的用户名为空时，又被称为匿名用户，如果测试下本地的mysql user表，可能会发现如下的一些内容:

```
+------------------+--------------+---------------+
| user             | host         | password      |
+------------------+--------------+---------------+
|                  | localhost    |               |
```
这就是为什么我们可以用`mysql`命令直接进入mysql的控制台，因为它匹配到了来自本地的空用户名，空密码的
这条记录。

从这条记录也可以看到，用户的密码也可以为空，这意味着该用户登录不要指定密码。非空的密码并非明文存储，
而是通过`PASSWORD()`函数加密。

主机名不会为空，缺省为`%`，意为匹配任意主机。`144.155.166.%`意为匹配144.155.166的C类地址。

mysql在验证请求时，会将现有的`mysql.user`排序，然后先按照host，再按照user，顺序匹配。如这样的
mysql user表

```
+-----------+----------+-
| Host      | User     | ...
+-----------+----------+-
| %         | root     | ...
| %         | jeffrey  | ...
| localhost | root     | ...
| localhost |          | ...
+-----------+----------+-
```
排序后就变成

```
+-----------+----------+-
| Host      | User     | ...
+-----------+----------+-
| localhost | root     | ...
| localhost |          | ...
| %         | jeffrey  | ...
| %         | root     | ...
+-----------+----------+-
```
可以通过`CURRENT_UER()`函数来查看当前连接使用的用户信息。

之所以会总结关于mysql连接验证的知识，是因为最近在配置一个新的服务的时候遇到了这样的一个问题。
AWS Staging VPC通过Direct Connection连回数据中心，而在Staging环境的服务需要连接到数据中心
的数据库服务器。在数据库服务器添加账号的时候，Host的信息填的是NetScaler的地址，比如"192.168.%.%"
但实际上应用服务器的网络属于"10.15.%.%"，导致连接出错。话说我刚觉得应用服务器配置的DB地址应该用
NetScaler VIP的地址而不是DB服务器的地址…………，shit。

----
#### Reference:

1. [mysql connection access](http://dev.mysql.com/doc/refman/5.7/en/connection-access.html)
2. [mysql request access](http://dev.mysql.com/doc/refman/5.7/en/request-access.html)
