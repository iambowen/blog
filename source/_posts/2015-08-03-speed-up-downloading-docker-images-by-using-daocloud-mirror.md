---
layout: post
title: "speed up downloading docker images by using daocloud mirror"
description: ""
category: docker
tags: [docker]
---

因为要试用下[hyper](http://hyper.sh/)这个新工具，不得不去下载docker镜像，不过官方的registry
简直慢如狗，加了VPN也很慢。以前有个docker.cn的镜像源也不见了，不过在搜索的时候发现了国内的一家容器
公司[daocloud](http://daocloud.io/)提供了免费的镜像服务。抱着试一试的态度，注册了个账号，在个人
的dashboard页面点击“加速器”，就可以看到定制的镜像链接，如“http://xxxxxxxx.m.daocloud.io”。
下面的文档中给出了不同系统下配置镜像的方法，我的host是Ubuntu系统，需要做如下的修改:

```bash
echo "DOCKER_OPTS=\"\$DOCKER_OPTS --registry-mirror=http://d32e3878.m.daocloud.io\"" | sudo tee -a /etc/default/docker
sudo service docker restart
```
再试着pull一下新的image.

```bash
vagrant@vagrant-ubuntu-trusty-64:~$ time docker pull ubuntu
Pulling repository ubuntu
d2a0ecffe6fa: Download complete
83e4dde6b9cf: Download complete
b670fb0c7ecd: Download complete
29460ac93442: Download complete
Status: Downloaded newer image for ubuntu:latest

real	2m0.469s
user	0m0.034s
sys	0m0.034s
```

这是我在公司里面下载70m左右的容器镜像所耗时间，其实在家只要30s左右，公司网络太差了。

Anyway, 虽然镜像流量使用配额只有1000GB，但是一般情况下应该是够用了，为daocloud公司点个赞，良心企业。
