---
layout: post
title: "don't put emoji in commit message"
date: 2016-11-03 23:47:07 +1100
comments: true
categories: ["bamboo","emoji"]
---
随着项目上越来越多的使用Slack以及Emoji的流行，很多人情不自禁的会在各种地方使用emoji表情。比如
在channel里面发`:bicyclist::skin-tone-2: :house: :thunder_cloud_and_rain: :disappointed:`。更甚者会在git commit message中添加emoji。比如像这样

```
finish story xxxx. :pear: xiao.
```
意思是完成这个开发的需求是和`xiao`结对做的。pull request发出后，review的人除了会发:+1:这样的表情表示支持外，还会用:shipit:,:ship:之类的表示赞同，可以merge。
这样的emoji为开发增添了乐趣，但是有时候也会带来麻烦，比如我今天就遇到了这样的情况。
在清理完一些旧代码后，我在提交信息里面➕了下面的消息:
```
clean up the :older_man::skin-tone-2: code.
```
提交merge后，过了一段时间看了下build，还没有到运行阶段就挂了。查看了下原因，发现是bamboo在保存提交信息时遇到了一个复杂字符出错了。
```
(org.springframework.jdbc.UncategorizedSQLException : Hibernate flushing: Could not execute JDBC batch update; uncategorized SQLException for SQL [insert into USER_COMMIT (REPOSITORY_CHANGESET_ID, AUTHOR_ID, COMMIT_DATE, COMMIT_REVISION, COMMIT_COMMENT_CLOB, FOREIGN_COMMIT, COMMIT_ID) values (?, ?, ?, ?, ?, ?, ?)]; SQL state [HY000]; error code [1366]; Incorrect string value: '\xF0\x9F\x91\xB4 ...' for column 'COMMIT_COMMENT_CLOB' at row 1; nested exception is java.sql.BatchUpdateException: Incorrect string value: '\xF0\x9F\x91\xB4 ...' for column 'COMMIT_COMMENT_CLOB' at row 1)
```
当时就感觉是这个emoji出问题了，搜了下提示的编码的十六进制，果然是这个原因……。没办法只好reset下，push force，再重新修改提交信息再push。
同事告诉了我这个事故的根本原因是mysql的utf-8对Emoji的支持不够，解决的办法就是把数据库的charset设置为`utf8mb4`，详见这篇[文章](https://mathiasbynens.be/notes/mysql-utf8mb4).
所以，以后玩emoji的时候一定得先确认系统支持，否则可能会带来一些:shit:。
