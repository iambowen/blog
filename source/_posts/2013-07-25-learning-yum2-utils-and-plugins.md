---
layout: post
title: "Learning Yum(2): Utils and Plugins"
description: ""
category: linux
tags: [yum, centos]
---

Yum 提供了一些列的utils以及plugins来增强自身的功能，比如yum-config-manager，fastestmirror等。


```bash
[vagrant@localhost ~]$ yum info yum-utils
.......
Description:
yum-utils is a collection of utilities and examples for the yum package manager. It includes utilities by different authors that make yum easier and more powerful to use. These tools include:
: debuginfo-install, find-repos-of-install, repodiff,
: needs-restarting, package-cleanup, repoclosure,
: repo-graph, repomanage, repoquery, repo-rss, reposync,
: repotrack, show-installed, show-changed-rco, verifytree,
: yum-builddep, yum-complete-transaction, yumdownloader,
: yum-config-manager, yum-debug-dump,
: yum-debug-restore and yum-groups-manager.
```
它们各自的作用可以很容易从自己的命名上看的出来, 我们挑几看上去有意思的命令研究下， 比如yum-config-manager，yumdownloader，find-repos-of-install等。

yum-config-manager: yum配置管理工具
显示base这个repo显示的所有的配置项以及其对应的值:

```bash
yum-config-manager base
[vagrant@localhost ~]$ yum-config-manager base
======== repo: base
[base]
base_persistdir = /var/lib/yum/repos/x86_64/6
baseurl = http://mirrors.163.com/centos/6/os/x86_64/
cache = 0
cachedir = /var/tmp/yum-vagrant-isJUZ8/x86_64/6/base
enabled = True
name = CentOS-6 - Base
retries = 10
timeout = 30.0
......
```
修改base这个repo的enabled选项为false同时保存:

```bash
[vagrant@localhost ~]$ sudo yum-config-manager base --setopt=“base.enabled=0” --save
[vagrant@localhost ~]$ yum-config-manager base
======== repo: base
[base]
enabled = False
......
```
yumdownloader: 从yum的repo下载rpm package

```bash
yumdownloader  --destdir /tmp/  ConsoleKit-devel-0.4.1
ConsoleKit-devel-0.4.1-3.el6.i686.rpm      |  15 kB     00:00
ConsoleKit-devel-0.4.1-3.el6.x86_64.rpm    |  15 kB     00:00
```
find-repos-of-install: 找到已安装package的repo源

```bash
[vagrant@localhost ~]$ find-repos-of-install vim-common
2:vim-common-7.2.411-1.8.el6.x86_64 from repo base
```

试用了一些个utils命令，觉得没有那些是特别有用的，只是能够稍微提高下效率，比如yum-config-manager,可以直接通过这个命令修改配置。

当你使用yum命令的时候，会提示你加载的[plugins](http://yum.baseurl.org/wiki/YumUtils)。相比之下，yum的plugins就显得有用多了。yum有很多plugins，aliases， axelget等都是我觉得比较有用的。配置yum plugins的入口有三个地方，我们以yum-aliases这个plugin作为示例:

```bash
[vagrant@localhost ~]$ sudo yum install yum-aliases #安装插件
/etc/yum.conf
	plugins=1/0     					#使用/不试用所有的插件
/etc/yum/pluginconf.d/aliases.conf
	[main]
	enabled=1/0  					#使用/不试用aliases插件
	# conffile - config. file to use
	# <default> = /etc/yum/aliases.conf
/etc/yum/aliases.conf
	FORCE --skip-broken --disableexcludes=all
	up   upgrade
	in   install
	rm   remove
	.........
```
/etc/yum/aliases.conf中定义了yum很多命令的alias，比如 in => install, 这样我就可以用yum in package_name,少敲几个字，节省体力。alias到option的用处会更大些，比如上面的FORCE。乐意的话，我们也可以按照个人喜好自己定制这个aliases的配置文件。

最近用过的一个比较有用的yum plugin是axelget，作用是从repo中下载package的时候支持多线程，加快下载速度。yum依赖于libcurl，默认应该是只能支持单线程的，如果网络条件一般的话，速度会比较慢。axelget 插件的用法:

```bash
 echo -e "[main]\nenabled=1\nonlyhttp=1\nenablesize=300000\ncleanOnException=1\maxconn=10" > /etc/yum/pluginconf.d/axelget.conf

 curl -o   /usr/lib/yum-plugins/axelget.py  http://yum-axelget.googlecode.com/svn/trunk/axelget.py

 rpm -i http://pkgs.repoforge.org/axel/axel-2.4-1.el5.rf.x86_64.rpm
```
axelget.conf 文件中的maxconn意为同时下载的线程数量。在我们项目中，从build pipeline打包出来的rpm package是保存在企业数据中心的repo中，但是测试环境是亚马逊的美国EC2节点，处于不同的网络中。如果是自动化部署，对于一个70Mb左右的package来说，安装更新消耗的时间大概是11分钟左右，使用axelget插件后，消耗的时间不到2分钟，下载速度提升很明显。虽然还是有点慢，但是已经处于可接受范围了。如此，每次在测试环境部署新的package的时间就大大降低了。

如果是想在某次的yum的transaction中禁止某个plugin的话，只需要加上 --disableplugin=plugin_name的option就好了，比如:

```bash
yum update --disableplugin=aliases, refresh-pack*
```

在开始下一篇关于yum的blog前，给大家介绍yum自动补齐的工具,懒人必备。

```bash
rpm -i http://pkgs.repoforge.org/bash-completion/bash-completion-20060301-1.el6.rf.noarch.rpm
source /etc/bash_completion
```
有了自动补齐后，就不用担心忘记命令或者费劲敲键盘了。

关于配置和插件等，基本就是这样了，下次介绍yum基本的命令用法。
