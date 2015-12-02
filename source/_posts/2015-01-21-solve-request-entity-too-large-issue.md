---
layout: post
title: "Solve Request Entity Too Large Issue"
description: "nginx"
category: "nginx"
tags: [nginx, aws]
---

[Sonatype Nexus](http://www.sonatype.org/)是一个Maven的仓库管理器。它的好处在于:

1. 作为其他Maven仓库的镜像或者代理，加快本地构建速度，同时节省带宽
2. 作为私有仓库，可以保存自定义的artifacts，支持多种文件类型，jar，rpm等

最近我们将nexus从美西的AWS数据中心迁移到了悉尼的数据中心，迁移后的Nexus架构如下:

1. 新建一个`t2.micro`的instance，挂载一个100GB大小的EBS block，并nexus工作目录放在其中
2. 新建一个ELB，指向该instance，并限制访问的IP地址来源
3. 在R53中建立域名的记录，并将其指向该ELB

为了保证Nexus环境在遇到问题时可以快速恢复，我们做了两件事情:

1. 保证用代码可以重现整个Nexus的基础架构，这点通过CloudFormation结合AWS CLI工具实现
2. 为了保证数据的完整性，我们在Nexus上运行cron job，每天给EBS block做一次snapshot

最近在更新了Nexus 上传artifact用户的credential之后，有其他组反馈往Nexus上publish artifact失败。
我和同事开始以为是有人又修改了密码，重新设置后依然不行。
>这里要说一下Nexus的用户权限管理，它的权限可以细化到能不能用网页的方式登陆，可不可以往某个
>repository中上传文件等。所以，如果发现有credential不能网页登陆Nexus，不代表这个用户不能够上传
>artifact。

于是我和同事登陆到了Nexus的instance上面，一边执行上传artifact的任务，一边查看log。发现Nginx的
access log返回的是`413 - Request Entity Too Large`。原来，Nginx配置中`client_max_body_size`
这项可以限制请求的body大小，我们要上传的jar文件大概有19Mb，远超其默认的大小，所以请求出错，难怪本地也会返回`java.net.SocketException`.
于是，先在Nginx的配置文件的`http`部分添加:

``` nginx
client_max_body_size 50M;
```
保存之后，检查Nginx配置，reload:

``` bash
root@aws nginx # nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

root@aws nginx # nginx -s reload
```
再次尝试上传，果然成功了。

之所以以前没有出现过这样的问题，是因为构建工具直接访问的是Nexus的8081端口，而其本身是没有这样的上传
文件大小限制的。我们在做migration时用了Nginx只是为了做代理，映射80->8081。但是在有ELB的前提下，
已经没有必要再使用Nginx了。所以下一步要做的事情就是更新cloudformation stack，让ELB的80和8081
端口都指向Nexus instance的8081端口即可。
