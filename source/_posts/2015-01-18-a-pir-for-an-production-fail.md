---
layout: post
title: "A PIR for a production failure"
description: "PIR"
category:
tags: [PIR]
---

客户这边对产品环境的事故严重性分为5级定义:

1. Sev 1 一级事故: 服务完全不可用，并且影响到了客户;
2. Sev 2 二级事故: 产品的主要功能对于一小部分用户不可用，或者产品的次要功能对于大多数用户不可用；
3. Sev 3 三级事故: 不重要的功能不能使用，对外部可见，但是影响不大，不属于严重事故；
4. Sev 4 四级事故: 没有显著的影响，错误只是内部可见；
5. Sev 5 五级事故: 不期而至的错误，只发生在团队内部；

举个例子来说，比如主站挂掉，这就是个一级错误，但是如果某个服务的产品数据库不能连接，造成的影响要小些，
可能就是二级事故了。蓝绿部署时inactive环境部署失败的话，这应该算是4级事故。将团队用的CI master
以及DB干掉，这个应该算是5级事故了。

一般来说，产品在上线之前，都会经过build pipeline, test/staging环境的各种测试，正常情况下它都是
应该工作的，所以出问题的情况都是非常巧合，甚至令人觉得匪夷所思。这也是这些事故有趣之所在了。

前几天，我就引起了这么一起Sev 2事故。我们做了一次[PIR](https://sysadmincasts.com/episodes/20-how-to-write-an-incident-report-postmortem)回顾
了这次事故，并且提出了相应的改进办法。

### 应用的功能
----
收集用户信息的静态页面


### 技术栈
----

1. 纯前端项目，用Grunt/Bower等前端工作流，Karma做JS测试框架，requirejs做模块管理，前端框架用Backbone；
2. 后端的API基于Scala，集成测试通过Pact([CDC](http://martinfowler.com/articles/consumerDrivenContracts.html))的测试方式；

### 系统架构
----

1. 应用部署在AWS的S3上；
2. 最前面用Akamai做CDN；
3. 通过JSON的方式对后端数据读写；

### Build Pipline以及部署方式
----

1. 本地开发，测试运行后提交代码， PR/Review/Merge
2. CI上面运行单元测试以及Pact集成测试，并且打包生成zip文件
3. 用`unzip-to-s3`这个工具将文件上传到staging S3 bucket 和 production S3 bucket
如果我知道这玩意直接进入production，当时说不定会去多看几眼……。

### 事故回顾
----

1. 我提交footer修改的PR，客户回复`LGTM`，然后merge
2. 2天后客服人员通过Zendesk提交ticket说有几个客户抱怨产品好像不大好用了
3. Tech Leader立刻召集产品相关的几个开发和运维人员检查，发现引用cache busting的js的script tag不知道怎么出问题了
4. 在没有确定最根本的原因时，它们尝试revert了我的代码，然后手动修改了产品环境的文件……-_-！
5. 找到了最根本的原因，是其中一个用来做cache busting的module突然升级，然后在package.json它是以`~0.0.10`的方式被引用的，如此在`npm install`的时候，它会自动升级小版本，没想到那个版本有问题，引入了一个bug。然后打包的过程在Bamboo build上，没人会去仔细看build log。
6. 拿回我的提交，将cache busting模块的版本锁定，重新部署后恢复正常

### 事故后的Action
----

1. 增加相关的测试用例，部署到staging和production后添加相关smoke测试以检验一切加载正常；
2. 用到的cache busting库在github没有很多人参与，所以重新考虑换一个更加稳定，用得人多点的模块；
3. 尝试将zendesk服务和pagerduty集成，如此开发团队会更快得到来自客户的反馈会；
4. 思考下这种直接进入产品环境的持续交付的方式是不是合适的，或者如何去改进；
5. 考虑对项目中使用到的模块用专门的build做升级时检查；
6. 测试时要针对整个打包好的assets；
7. 通过语义化(sematic)工具结合pagerduty报警;

----

对于纯前端的这种app，监控确实不如后端(可以用现成的工具如[NewRelic](https://www.newrelic.com/))好做，目前的方式是通过Nagios做Active check，检查bucket中对应
的文件是否存在，实际的意义并不大，我觉得监控产品环境的集成页面会更有价值些，比如测试相应时间，如果响应
时间超过某个值，比如10s，或者返回的html中没有包含必要的元素，就认为它可能会有问题。

上面的最后一条也不错，通过API收集前端请求的次数，将数据发送到类似Graphite之类的工具，然后用[Skyline](https://github.com/etsy/skyline)之类
的工具进行数据分析，如果有些数据(如访问量)突然高于或者低于某个值，就可以发送警报到pagerduty。
另外，对于产品中引用的第三方类库/模块，除了固定使用的版本之外，还有必要搭建测试最新类库集成后的功能
完整性的build，这个叫做[Canary Build](http://www.thoughtworks.com/radar/techniques)(金丝雀构建)，在Thoughtworks最新
的Tech Radar中有提到。

最后，希望我不会因为制造太多的产品事故而被炒-_-!
