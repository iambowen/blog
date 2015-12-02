---
layout: post
title: "使用KeePassX管理你的密码"
description: ""
category: 
tags: []
---


SaaS化的趋势已经不可阻挡，传统软件的功能逐渐web化。网上服务越来越多，对应的账号也越来越多，所有账号一个密码显然不现实，至少支付宝的登陆密码和支付密码就是不一样的。再加上密码要求越来越严格，大小写字母，数字，特殊字符组合……，尼玛刚设置10分钟后就忘记了。这个时候，就得靠软件了……，比如[KeePassX](http://www.keepassx.org/)。
KeePassX是一款跨平台的免费开源的密码管理软件，安装和使用都比较简单。

[下载地址](http://www.keepassx.org/downloads/)

界面如下:
<img src="https://dl.dropboxusercontent.com/u/92660221/images/keepass1.png", style="width: 680px;">
设置:

1. 创建密码数据库.
 点击 File ->  New Database, 新建一个密码的数据库。创建的时候需要你设置文件保护密码，保存后会生成一个.kdb文件，

2. 密码管理. KeePassX管理密码的方式有两种，组(Group) 和入口(Entry)。比如设置邮件组,里面放置所有的邮件账号。新建一个入口(Entry)，输入标题，账号已经密码，确定即可。
<img src="https://dl.dropboxusercontent.com/u/92660221/images/keepass2.png", style="width: 680px;">

3. 也可以用KeePassX帮你生成随机密码，设置密码过期时间等等，不过我没有用。
使用:
异常简单，假设你现在要登录163的邮箱，选中标题，Command+B 拷贝用户名，Command+C拷贝密码，输入登录即可。

设置一个强度较高的文件保护密码，只要不是电脑丢了，应该是比较安全的。除此之外，还有类似[1Password](https://agilebits.com/onepassword)之类的软件，只不过需要收费，上次公司给报销，但是当时没有意识到，错过了。。。
