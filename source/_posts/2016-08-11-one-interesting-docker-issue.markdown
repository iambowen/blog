---
layout: post
title: "one interesting docker issue"
date: 2016-08-11 15:52:10 +0800
comments: true
categories: ["docker"]
---
项目上Akamai的回归测试运行在数据中心一台用Puppet管理的固定的虚拟服务器上，这台服务器是Bamboo Agent，负责运行所有遗留系统的自动化部署任务。
前几天一个客户的Ops找我帮忙一起让这台服务器支持Docker，然后将测试放在docker中运行。我们修改puppet脚本，然后更新了Docker，结果发现2.6的内核最多运行docker 1.7，而运行测试的docker compose需要的docker客户端要高于1.7。 鉴于改动较大，于是我们换一种思路，用在AWS账户下已有的Bamboo docker agent去运行测试。所以revert了Puppet修改，并且在服务器上运行。
以为一切都结束了，没想到过了几天，另一个组的Ops来找我说staging的部署失败了，问我什么原因，提示大意是没有找到NetScaler服务器的路由。我觉得很奇怪，就看了眼服务器上的路由表。结果发现了下面的现象：
```
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
```
囧，staging的IP range也是`172.17`，原来是这个原因。
于是，先停止这个网络设备，然后删除，之后再重启网络服务解决问题。

```bash
ip link down docker0
ip link del docker0
service network restart
```
我觉得从这个错误中可以学到两个事情

1. 配置管理工具的不可靠性，Puppet并没有完整的清理掉所有docker相关的东西
2. 这种`Pet`服务器的不可靠性，如果服务器是每天都按照配置重新创建也不会出现这样的问题
