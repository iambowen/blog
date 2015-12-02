---
layout: post
title: "a good lesson on security when provisioning"
description: "security, puppet, provisioning"
category:
tags: ["security", "puppet", "provision"]
---


客户很多老的(legacy)系统都部署在租用的数据中心中，其中部署的方式大多是通过puppet去做zero-downtime
或者blue-green部署。绝大多数服务器都使用基于Centos的base image,其中包含基础架构需要的软件，如
nagios agent, splunk agent等。应用程序通过流水线打包为rpm package，上传到[Koji](http://fedoraproject.org/wiki/Koji/ServerHowTo), Koji将repo
分为三类，dev,staging,production，大致的持续交付流程如下:

1. 代码提交后触发ci的build job，通过所有测试后生成package，上传到Koji，并tag在dev repo下面；
2. 自动化部署测试环境，服务器的`yum repo`的配置中缺省enable dev，所以`yum install`即可更新package；
3. 接受测试结束后，运行build去promote这个package到staging repo；
4. 在部署的build中执行deploy staging 的`dry-run`job和实际应用的job，由于staging服务器缺省
enable的repo是staging，所以它也会自动安装最新的package，并不需要你在puppet脚本中指定package的
版本号，`ensure => latest`即可。
5. 产品环境的部署过程同staging类似。

看上去还不错，然而这里面有几个问题:

1. 每次上传新的rpm包到koji时，它需要重新indexing去更新metadata，这个时间很长，可能有10几分钟，
会极大的影响持续交付流水线的时间
2. 现在没有专门的团队去照看它，一旦koji出现问题，会极大的影响应用的部署上线时间

按道理说我们应该去解决Koji的indexing的问题，这样比较一劳永逸。但是客户的重心逐渐都转移到了AWS上，
所以不愿意再投入时间去修复这个问题，加之CI工具，Bamboo或者Jenkins都可以host rpm artifact，所以
后来的解决方案就是直接从Bamboo或者Jenkins取package，然后安装更新。我们在应用的部署上已经采取了这样
的方式，提交跑完流水线加部署到产品环境大概需要20分钟多点的时间，是以前的一半，当然这不是完美的解决方案，
如其中有安全问题，如产品环境(数据中心)需要通过代理去访问AWS部署的CI的artifact。不过作为过渡方案，
目前来说大家还都能接受。

好，该说到今天的问题了，Koji服务器不能显示导入的package，致使一个应用不能部署最新的package，其中
包含一个比较紧急的bug修复，于是客户问我试试通过Jenkins去获取rpm包进行部署，绕过Koji。

首先，我做了如下的测试:

1. 下载Jenkins host的rpm包是否需要验证(用户名/密码)，答案是否定的，Bamboo需要验证，Jenkins的
安全性比较差，but anyway.
2. 是否能在staging和production服务器上下载在Jenkins上的package，答案也是可以。

那么我就有了一个大致的思路，通过传入buildnumber, 用puppet脚本调用wget等命令将package下载到服
务器的/tmp下，然后在package定义中，通过指定rpm文件，进行local install.

``` puppet
package { "my-app":
   provider => "rpm",
   ensure => "$version",
   source => File["/tmp/$package_name"],
   require => File["/tmp/$package_name"]
 }
```
因为对puppet的type理解不足，我总以为`File["/tmp/$package_name"]` 返回的是文件的路径，但是
没想到返回的却是一个字符串，类似这样`"file /tmp/my-app-version.rpm"`，所以rpm update总是
失败。因为有点着急，我就开始各种hack，通过`Exec`类型，执行`yum localinstall`的方式更新，然后，
我就被客户的安全专家怒喷……，针对我的puppet修改提交，他提出的几点关于安全方面的原因如下:

1. puppet脚本不应该用root账号去执行，而是应该用一个专用的限定权限的账号，如puppet,并且只授予安装
删除以及在某些特定目录下修改文件的权限，这样它就不能执行一些其他的命令；
2. 将文件放在/tmp下是非常危险的，因为当黑客通过web应用程序漏洞获得`www`用户权限时，他可以将一些
恶意的文件(如包含恶意脚本的rpm包)放在/tmp下，这时候如果我安装rpm包，很有可能中招。

大神在喷完我后给出了几点建议:

1. 鉴于情况比较特殊，所以可以沿用当前的方式，但是可以将文件下载到如/opt之类只有root用户组才能访问的
目录中；
2. 可以考虑使用S3作为yum的repository(这个以前我们就有讨论过，以后应该就是这种方式了)

我觉得他说的很有道理，所以先静下心来，找到了为什么source文件找不到的原因，然后修改了保存pacakge的
目录，最后的`package`定义脚本如下:

```puppet
package { "my-app":
  provider => "rpm",
  ensure => "$version",
  source => "/opt/$package_name",
  require => File["/opt/$package_name"]
 }
```

虽然被喷有点尴尬，但是学到了东西感觉还是很开心的，以后要更加稳重些，多找下root cause和best
practice。
