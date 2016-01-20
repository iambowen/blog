---
layout: post
title: "hand over DNS resolve to VirutalBox"
date: 2016-01-20 17:16:12 +0800
comments: true
categories: ["virutalbox", "dns", 'network']
---

当你用[vagrant](https://www.vagrantup.com/)新建一个虚拟机(driver 为virtualbox)并使用NAT方式让guest虚拟机连接外网时，如果有无线网络的变化，虚拟机中`/etc/resolv.conf`不会对应的修改，导致域名解析失败。

解决的办法是将DNS解析的任务交给虚拟机管理工具如virtualbox，假设我们要修改名为`test`的虚拟机的设置：

```
 ~> VBoxManage list vms
"mesos1" {74214693-3477-4386-a9b7-4abc3b7e608d}
.......
"test" {b269c98f-00e8-49a3-a8d0-53629187ea62}

#保证vm没有在运行，然后执行
 ~> VBoxManage modifyvm test  --natdnsproxy1 on
```

重新启动vm，不管怎么切换网络，应该都不会再出现域名解析的问题。
如果是用Vagrantfile管理虚拟机的配置，可以更改vm的配置：

```
config.vm.provider "virtualbox" do |v|
  v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
end
```
