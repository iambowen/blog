---
layout: post
title: "tracing and production bug about netty"
description: ""
category:
tags: [netty, bug]
---

CTO在slack上给我们通报了一个bug，具体表现为网站请求的一个服务，在接受完请求后，再发起新请求可能会
返回504.

客户的tech lead和我一起追踪这个问题，出问题的是一个微服务，其架构如下:

```
Akamai -> ELB -> Instances -> Netty App(基于unfilterd-netty-server 0.8.4)
```
当时我的猜测是ELB和instance之间的连接或者处理出了什么问题，但是Tech lead先查看了ELB Cloudwatch
上的错误数量，并不多，Splunk中搜索ELB access log发生的频率也不高，当然从newrelic可以看到应用本
身的rpm也不是很高。

他先做的事情是在产品环境稳定重现bug，手段是通过一个超大header的Get请求，之后再接若干个请求，就可以
复现。CTO认为可能是Akamai和AWS中间发生了什么错误，于是我们查看了Akamai的access log，寻找504，
数量和ELB上出现的错误基本一致，排除是Akamai出问题的可能。

在这个过程当中，他新建了一个github issue，并将分析过程，以及检验的证据comment到issue中，很不错的
实践。

于是问题又回到了ELB和instance之间，我在猜测是不是因为ELB的cross zone负载均衡出了问题，ELB和instance
之间的网络访问出了错，不过回头想想感觉不太可能。查看了instance上access log，发现没有任何5xx的错误
代码记录。

tech lead再次查看了下Splunk，发现在某个时间段后，504的错误突然大幅增加，并且数量、频率比较稳定。
查看CI以及提交记录，发现刚好是一次大的重构的开始时间，将代码改为Freemonad风格，同时引入了`unfilterd-netty-server 0.8.4`
但是理论上来说重构没有功能性修改，而且也很难判断哪些代码会引起问题。

于是我将staging的版本回退到重构前，测试竟然是好的。。。damn it。

我突然想起来，ELB有个选项，idle connection timeout，大概意思就是说ELB和instance上app间如果
有连接空闲，超过一定时间后才关闭，这是为了减少http通信的开销。很有可能第一次的请求处理后，因为某种原因，
复用的连接没有正常的被Netty服务。所以我们测试了下，先发一次请求，成功，紧接着后面的请求理论上会失败，
因为它复用了前一个ELB和instance的连接，但如果这时再发一次请求，它应该也会成功，因为前一个空闲连接
被占用了。测试了下果然如此，并且当超时时间设置为1s时，基本不会出错了。

所以基本上可以认为是Netty处理大headers的问题，另外一个同事给出了这个netty issue的链接[netty#3379](https://github.com/netty/netty/pull/3379)，同时它给出了在本地测试的方法，就是用
curl去同时请求两个链接，这个时候它会复用连接，而不是关闭连接再重新打开。果然，`unfilterd-netty-server`
依赖的netty`4.0.24.Final`是存在问题的，而比较新的`4.0.29.Final`是没有问题的。

暴力覆写netty的版本可能会引入新的问题，不过我们试着冒下险，更改后所有测试都通过，并生成新的AMI，然后
在Staging部署测试，一切正常，最后部署产品环境，通知CTO，问题修复。

我们的做法比较粗暴，而且不是好的解决办法，最合适的方法是给`unfilterd-netty-server`项目发pull
request升级它依赖的netty版本号。不过，花了好长时间也没有找到合适的公共netty站点去证实，加上开源
社区的反馈不一定会很快，所以还是选择了先暴力升级的方法。

鉴于还有其他项目使用这个类库，而它们都会存在相同的问题，所以必须在github中找到这些项目。幸好前一段
时间都在做patch management，通过一些插件可以将项目类库的依赖生成json文件并上传到s3 bucket。通过
查找这些依赖的json文件我很快就定位到了需要修改的系统并发了github issue，这些团队会自己决定如何处理
这个问题。

通过解决这次的bug，学到了很多追踪、分析和解决问题的方式，受益匪浅。
