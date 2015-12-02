---
layout: post
title: "change system library path"
description: ""
category: linux
tags: [linux]
---

找不到共享文件是我们在native build的时候经常遇到的问题，有一种情况是操作系统上确实没有该共享文件，还有一种是含有该共享文件的目录没有被加入到library 查询的路径中。

假设现在我们需要把 `/opt/lib/lib.test.so` 加入到library path中.
在不想影响到系统配置的情况下，我们可以修改环境变量:

```bash
export LD_LIBRARY_PATH=/opt/lib:$LD_LIBRARY_PATH
```

easy right?
如果你确信你是想一劳永逸的把```/opt/lib/```加入到library path中，那么可以直接修改ldconfig的配置:

```bash
[vagrant@localhost ~]$ sudo ldconfig | grep opt
[vagrant@localhost ~]$
```
目前尚未出现```/opt/```。
怎么办呢？直接去修改```/etc/ld.so.conf.d/``` 下的配置文件。

```bash
[vagrant@localhost ~]$ ls -al /etc/ld.so.conf.d/
total 16
drwxr-xr-x.  2 root root 4096 Mar  9  2013 .
drwxr-xr-x. 65 root root 4096 Nov 18 10:04 ..
-r--r--r--.  1 root root  324 Feb 22  2013 kernel-2.6.32-358.el6.x86_64.conf
-rw-r--r--.  1 root root   17 Dec  7  2012 mysql-x86_64.conf

[vagrant@localhost ~]$ less /etc/ld.so.conf.d/mysql-x86_64.conf
/usr/lib64/mysql
```

看上去规则很简单，第一步为app添加配置文件，第二步在文件中写入library的路径。

```bash
sudo echo “/opt/lib” > /etc/ld.so.conf.d/opt.conf
```

make it happen

```bash
[vagrant@localhost ld.so.conf.d]$ sudo ldconfig

[vagrant@localhost ld.so.conf.d]$ ldconfig -v | grep opt
/opt/lib:
ldconfig: File /opt/lib/lib.test.so is empty, not checked.    #测试文件
```

So, are you happy now?
