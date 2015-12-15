---
layout: post
title: "make everything production like - (2/2)"
date: 2015-12-06 17:12:21 +0800
comments: true
categories: ["practice", "AWS"]
---

[开发环境出问题的时候，影响到只是自己](http://iambowen.github.io/2015/07/05/make-everything-production-like/)，如果持续集成环境或者其相关的基础设施出了问题，那影响到的就
是所有人以及整个开发的进展，我们曾经遇到一次这样的事故，整个 [Bamboo](https://www.atlassian.com/software/bamboo)(CI)环境的Master和Database都被干掉了，出乎意料的是AWS RDS的自动镜像同时也被删除,于是所有的人花了一个礼拜才重新建好了全部的流水线。

除此之外，一些基础设施，比如企业私有的Repository(如Nexus, Koji, rubygems服务器等)出现问题，也会影响到整个开发和持续交付的时间。

如何解决这个问题？很简单，提高这些环境的可用性，把他们当做产品环境一样看待，提高出错的响应速度，
减少平均恢复时间等。

先举一个CI环境当做产品环境来对待的例子。
一些简单的背景:

1. 客户使用的持续集成工具是[Bamboo](https://www.atlassian.com/software/bamboo)
2. CI Master，Agent以及数据库服务都采用了AWS的服务，如EC2、RDS、R53等
3. 用CloudFormation去管理整个CI服务的基础设施，同时用Rake task去简化管理的难度。

其具体的结构图如下:
![arch](http://7xp2qy.com1.z0.glb.clouddn.com/bamboo_arch.png)

该结构详细解释如下:

1. Bamboo Agent和 Bamboo Master的依赖及其配置打包成RPM，部署的EC2 instance基于Centos定制过的AMI
2. Bamboo Master/Agent/DB 都用CloudFormation管理
3. 在Bamboo Agent Stack的LaunchConfiguration中的Metadata中，安装在Agent中运行各种build的依赖，
比如不同的Ruby版本等，同时定义`cfn-hup`服务，监听Agent的Stack变化，如果有Metadata的变化，
比如，更新了Agent上支持的Java版本，则在Agent上更新该配置
4. Bamboo Agent由一个AutoScalingGroup管理，除了自动Scale，还可以每天定时启动或者停止Agent
Instance，节省成本
5. Bamboo Master的Stack中做的事情类似
6. Bamboo Master的SecurityGroup只接受来自Bamboo Agent的SecurityGroup的访问，Bamboo
Master DB的SecurityGroup只接受来自Bamboo Master SecurityGroup的请求
7. Bamboo Master DB使用RDS服务
8. Bamboo Master服务器上运行的Cron Job每天会定时备份文件系统的Snapshot
9. Bamboo 服务器上的一个Plan每天会运行定时的任务，创建Master DB的Snapshot,RDS可以设置自动
生成snapshot，不过一旦Master DB被干掉，snapshot也会被一起干掉。所以，安全期间，还是manual
snapshot比较好。

回顾这套结构，如果某个Agent挂掉，AutoScalingGroup会重新spin up一个新的Agent Instance。
如果Bamboo Master或者Master DB挂掉，也可以通过CloudFormation Stack以及备份的Snapshot
在1-2个小时以内恢复，时间的开销相对较少。

仔细的同学可能会注意到，为了满足运行build的各种条件，需要安装各种依赖，比如不同的Ruby版本，
不同的Java版本等，重新创建一个Agent Instance到配置完成注册成为Bamboo服务，时间会比较长。而且
如果Metadata的更新导致环境失败，会迅速影响到所有的Agent。

相信很多人会想到更好的解决方案，比如将每个build任务都在Docker容器中运行，如此作为整个CI环境
的维护者，只需要保证每个Agent上面有docker deamon运行，整个Agent挂掉的几率大大降低，同时维护
的责任分散到每个团队内，减轻了维护的压力。

下面介绍如何提高企业内部的私有Repository，如Nexus的可用性和稳定性以及快速恢复能力。
我们的Nexus服务器的结构图，如下:
![nexus arch](http://7xp2qy.com1.z0.glb.clouddn.com/nexus_arch.png)

详细解释如下:

1. Nexus服务运行在ELB后的一个EC2 Instance上
2. 其部署基于安装有Nexus服务的Base AMI以及CloudFormation stack
3. Nexus的artifact目录挂载在一个EBS volume下，Instance在初始化时配置了InstanceProfile，
在crontab添加脚本，可以用InstanceProfile中的role去创建EBS volume的daily snapshot，以防止artifact数据丢失
4. 监控方面，如果ELB下面的健康的Instance数量少于1或者Instance上的EBS Volume没有正确的挂载，都会触发Cloudwatch Alarm，并通过SNS通知Pagerduty，然后Pagerduty再将警报发给维护Nexus的Ops

对于上面的Nexus结构，由于有足够的备份，不论是Volume挂载失败需要恢复或者是Instance当机，处理的
时间成本都会比较低，在半个小时以内。

开发/测试依赖的环境可能还有很多，更多的把它们当做产品环境对待，会大大增加持续交付的流畅度，减轻环境维护方面的痛楚。
