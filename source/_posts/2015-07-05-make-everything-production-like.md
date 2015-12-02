---
layout: post
title: "make everything production like - (1/2)"
description: ""
category:
tags: [development]
---

一般来说，软件生命周期涉及到的环境的典型划分如下：

1. Development
2. Test
3. Staging
4. Production

通常Staging和Production的环境非常的接近，差别可能只在于服务器的性能、数量以及数据的完整性等方面，
而且Staging/Production也会得到更多的照顾，什么`DR`,`Infrastructure as code`。各种monitoring，Active崩了有Inactive顶着，Master挂了，Slave顶上来，出了问题分分钟就有人来修好。
反观于Dev和Test环境，真是各种苦逼，14年Java经验的工程师设置了三天还没有把Ruby开发环境搞好，一
个前端工程师去解决为什么用Phantomjs在bamboo centos agent上运行测试不稳定，痛苦的要自尽。

除了这些，开发环境的不一致也会带来一些问题。最有名的就是“It works on my machine”。不过
这也不稀奇，毕竟不是所有人的开发环境和产品环境都是一致的。可能你在用Mac开发Ruby程序，但是在它
产品环境它却需要在Centos的机器上去运行。

对于开发环境来说，这些痛点总结来说，大概有如下几种场景:

1. 开发环境和系统环境混在一起, 任何对于开发环境的修改可能会影响系统环境。
2. 开发环境和产品环境的不一致。
3. 本地开发时，除了运行必要的测试，还要启动多个依赖的服务，做一些手工的验证测试。
针对这些问题，有如下的解决办法。

------
### 使用环境隔离的工具
可以使用一些类似沙盒的工具去管理开发环境，比如，

1. [rbenv](http://rbenv.org/)
2. [rvm](http://rvm.io/)
3. [chruby](https://github.com/postmodern/chruby)
4. [virtualenv](https://virtualenv.pypa.io/en/latest/)
5. [jenv](http://jenv.io/)

前三个工具都是针对Ruby语言版本管理，开发环境下常用的是rbenv和rvm，测试和产品环境有可能用到chruby，
因为它更加轻量。这些工具本质上都是修改BIN PATH，cd等，以rvm为例，推荐的使用方式如下：
安装开发环境需要的ruby版本如2.0

```bash
rvm install 2.0.0
```
安装bundler，用来管理项目依赖

```bash
rvm use 2.0.0
gem install bundler
```
对于ruby的项目，将依赖的gem安装在工程的`vendor/bundle`下，而不是装在`~/.rvm`的gemsets目录下。
进一步将环境隔离到工程内，同时最好将`vendor/bundle`加在`.gitignore`的列表中，这样不会将gem提交
到repository中。

```bash
bundle install --path=vendor/bundle
```
virtualenv针对的是Python的环境管理，Jenv针对的是JVM系列的语言，Java/Scala/Clojure。
这些工具最大的作用在于项目和系统环境的隔离，不能用来保证环境一致性，一旦有系统环境变动，如，
mysql版本升级，OpenSSL升级等，就得重新编译链接类库，很不方便。

-----
###用Vagrant解决环境一致性问题

我们很容易能够想到用虚拟机来解决环境一致性的问题，不用说，就是[vagrant](https://www.vagrantup.com/)
这个工具。vagrant是一个跨平台的虚拟机管理工具，通过Vagrantfile，可以配置：

1. 管理virtualbox或者VMware的虚拟机
2. 以headless/gui的方式运行虚拟机
3. 配置私有/NAT网络
4. bash/puppet/chef/ansible多种provision的方式
5. 共享/同步host/guest machine目录内容

Vagrantfile本身也可以看做是ruby文件，可以用ruby语言编写更加复杂的配置文件。[vagrantcloud](https://vagrantcloud.com)
提供了现成的base box，可以很方便的下载和发布box，比如，在vagrantcloud找到最新的Ubuntu14.04的box`ubuntu/trusty64`，
在工程中运行命令:

```bash
vagrant init ubuntu/trusty64
vagrant up
```
即可生成Vagrantfile并且启动虚拟机。对于一些开发环境要求比较复杂的项目，可以先用bash/chef/puppet
自动化配置的过程，然后运行`vagrant provision`配置即可。假设当前为gradle/java项目，guest共享的
目录为`/vagrant`，运行:

```bash
vagrant -c 'cd /vagrant && ./gradlew test'
```
即可在虚拟机中运行测试。`vagrant package`命令可以将当前的配置好的虚拟机打包为新的镜像，共享给团队的其他同事。这里有一点要注意的地方，虚拟机`/etc/udev/rule.d/70-persistent-net.rules`保存了当前虚拟机持久化网络设备udev规则。创建新的虚拟机时，新分配的设备和规则不符，导致网络出问题。所以在打包镜像之前，需要删除这个文件。

-----
###vagrant+docker解决依赖多个服务的问题

在微服务大行其道的今天，一个前端的工程可能会依赖多个微服务。前几天参加会议，还有人说微服务就是大家用
自己喜欢的语言/框架实现一个简单的服务。我替各位前端工程师们送他一句cnm，走好不送。
即便有类似[Pact](https://github.com/realestate-com-au/pact)
这样的基于[CDC](http://martinfowler.com/articles/consumerDrivenContracts.html)的更加轻量的集成测试方式，还是需要真实的启动浏览器，
请求真实的服务去在本地做测试，尽快的获得反馈。在本地配置这样的e2e或者模拟产品环境，大致有如下的几种思路：

1. clone所有依赖的工程，配置每个工程的环境，在不同的端口启动服务，覆写`hosts`配置，并且可能还需要
安装`nginx`做反向代理。这样做的成本比较高，而且对全局的环境影响比较大，也太过复杂。
2. 用vagrant为其它微服务配置好虚拟机，同样可能需要覆写`hosts`或者安装`nginx`做反向代理。这样
启动多台虚拟机，开销太高，还能不能愉快的开发。
3. 用vagrant+docker在一台虚拟机的多个容器上运行微服务，同时在虚拟机上用`nginx`作反向代理。
4. 修改`hosts`,将其他微服务指向测试环境或者staging环境，这样做的代价较小，但是公用环境可能会导致
数据问题。而且如果同时修改前端项目和其中一个微服务，这样做可能就比较困难了。

不卖关子了，第三种方案也就是我一年前在项目中使用过的方案，剥离了业务相关内容后放在了[github](https://github.com/iambowen/leviathan)。设计思路可以参见下面的架构图，以及项目中的文档。
<img src="https://dl.dropboxusercontent.com/u/92660221/images/Screen%20Shot%202014-07-30%20at%2012.53.05%20AM.png" style="width:800px;height=600px;">
这个设计相对是比较灵活，因为可以根据`nginx`配置的不同，选择使用哪个环境上的微服务，同时host环境几乎不需要做任何修改。当时没有选择使用`vagrant package`的方式去打包配置好的镜像，因为这样生成的镜像太大，客户的程序员下载太慢。所以选择从头provision的方式，但是当时的docker版本较低，docker-registry的性能也比较差，pull image的速度确实慢，所以真正的用户只有我同事一个人o(╯□╰)o，现在要好很多了。
我想说的是这样的方式是可行的，当然还需要继续改进。

下一节，我会继续介绍如何让测试环境更加接近产品环境。
