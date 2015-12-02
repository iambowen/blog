---
layout: post
title: "add swap space to solve 'can not allocate memory' issue"
description: ""
category:
tags: [vagrant，docker]
---

这几天要把一个新的api服务扔到docker container里面，可能是因为把api和postgre数据库放在了一个container里面，出现了内存不足的情况。

```bash
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000b0000000, 357892096, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (malloc) failed to allocate 357892096 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /tmp/jvm-663/hs_error.log
```

首先尝试了增加虚拟机内存的方法，在Vagrantfile中修改了分配给virtualbox的内存:

```ruby
  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.memory = 1024
  end
```
错误依旧存在，于是尝试增加swap memory的空间：

```bash
dd if=/dev/zero of=/root/myswapfile bs=1M count=1024  #1g的swap空间
chmod 600 /root/myswapfile
mkswap /root/myswapfile
swapon /root/myswapfile
```
然后问题就解决了……, 更合理的方式应该是把数据库和应用放在两个不同的container里面。
