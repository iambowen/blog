---
layout: post
title: "Set locale in Debian Wheezy"
description: ""
category: linux
tags: [linux, debian]
---

最近和一个客户聊天，说到一个很有意思的事情。他认为，在产品环境使用*inx的情况下，公司给开发人员普遍
配发Mac是不太合适的, 当然这个是见仁见智了。不过,如果对产品环境的系统熟悉的话，在遇到一些问题的时候，可以节省些时间。

想把Mac Book/Air操作系统更换成Linux代价还是略高，所以最好的办法是用虚拟机,必须就是Vagrant了。今天在使用Debian7 Wheezy的时候，就遇到了关于Locale的简单问题。

[locale](http://gerardnico.com/wiki/application/locale)是关于系统语言和区域选择，目的是为了表示正确的时间和数字格式。悲剧的是，下载的debian 7的virtual box默认竟然是用法语，提示信息各种看不懂。只好自己查找资料修改了。

最初使用的方法是修改LANG/LC_ALL的环境变量:

```bash
  echo -e "export LANG=en_us.UTF-8 \nexport LC_ALL=en_us.UTF-8" >> ~/.bash_profile;   
  source ~/.bash_profile
```

奇怪的是，并非是所有的输出消息都变成了英文，仍然有法语出现。于是，尝试从根上修改:

```bash
  sudo vi /etc/default/locale
  LANG=en_US.UTF-8
```

重启系统，按照常理应该已经OK，怎知还是出现警告，说设置locale失败。技穷只能求助文档，先查看了下当前
locale中已有的语言环境:

```bash
  vagrant@vagrant-debian-wheezy:~$ locale -a
  C
  C.UTF-8
  POSIX
  fr_FR.utf8
```

我去，只有这么几个选项，只能重头添加en_US.UTF-8了。所幸并不是很困难:

```bash
  sudo /usr/sbin/dpkg-reconfigure  locales
```

在弹出的选项框中，选中en_US.UTF-8确定即可。这个时候/etc/default/locale文件也被修改成支持en_US.UTF-8，运行下 locale -a :

```bash
vagrant@vagrant-debian-wheezy:~$ locale -a
C
C.UTF-8
POSIX
en_US.utf8
fr_FR.utf8
```

Nice.
[reference](http://people.debian.org/~schultmc/locales.html )
