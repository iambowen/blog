---
layout: post
title: "puppet cron pitfall"
description: "puppet"
category: "puppet"
tags: [puppet]
---

最近完成了一个任务，内容是更新一个cron运行perl脚本的连接数据库的credential。该脚本用来清除数据库中一些
过期的匿名用户，脚本部署在一个内网的服务器上，服务器的配置由puppet管理。脚本一直要保持运行，所以在
脚本中会检查pid的modified time，如果小于一个小时，则进程退出。因此，在脚本第一次运行后，以后的cron
运行的进程会自动退出。
所以，我的计划是这样的:

1. 更新puppet管理的脚本内容中数据库连接的credential
2. 更改puppet中服务器中cron运行的时间为每两分钟运行一次，这样，下次运行时，脚本pick up新的配置
3. `puppet --test`应用配置更改
4. 停止正在运行的脚本，`pkill process`
5. `touch -t some_old_time_stamp`
6. 监控log，等待脚本启动
7. 监控数据库，是否有新的credential连接进程， `mysql -e "show full processlist;" | grep new_user`
8. 一切运转正常，将puppet中cron的时间改回去并应用

关于cron配置的代码原本如下,原本cron运行的时间为23:55:

```puppet
cron { test:
  source => "some scripts",
  user => root,
  hour => 23,
  minute => 55
}
```
应用后生成的cron应该如此: `55 23 * * *  some scripts`
更改后的puppet配置如下:

```puppet
cron { test:
  source => "some scripts",
  user => root,
  minute => "*/2"
}
```
期望应用后的cron应该如此: `*/2 * * * * some scripts`
一切都非常顺利，直到step 6，等待日志输出时，发现时间过了4分钟还是没有日志，WTF!赶紧看了眼更改后的
cron运行时间:

```
*/2 22 * * * some scripts
```
顿时就震惊了，鉴于是产品环境，不敢担待，于是直接就顺手改了，两分钟后运行，如释重负。由此可见，puppet
cron的这个地方是略坑的, 如果更改比较多，最好是指定完整的时间配置，如:

```puppet
cron { test:
  source => "some scripts",
  user => root,
  year => "*",
  month => "*",
  day => "*",
  hour => "*",
  minute => "*/2"
}

```

话说这还不是最坑的地方，因为脚本还是没有顺利运行，真是日了狗了，不得不手动revert脚本，然后让脚本正常
运行，然后再重新查看脚本，测试，查找问题在哪里。
最后的原因实在是让人崩溃:

``` perl
die "ERROR: Missing or invalid database username '$dbuser'" unless (defined($dbuser) && $dbuser =~ m/^[a-zA-Z0-9]+$/);
```
新的`dbuser`名字中包含了连接线`-`，因为不能满足regex检查，所以程序直接退出了。实在是不能理解为什么
会有这样的约束条件，我只知道mysql数据库用户的名字不能超过16个字符，但是没听说过限制数据库用户名字格式
是个什么鬼。实在是无语，不过，我能理解，不会挖坑的程序员不是好程序员。
