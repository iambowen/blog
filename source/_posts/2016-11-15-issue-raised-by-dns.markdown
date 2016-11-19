---
layout: post
title: "issue raised by DNS"
date: 2016-11-15 21:33:22 +1100
comments: true
categories: ["DNS"]
---
最近在客户现场出差，见证了不少有趣的线上事故，下面要讲的就是其中之一。
一段时间依赖，某个微服务在生产环境的response的延迟陡然增加了几百毫秒，而部署的代码并不是造成延迟原因。从Newrelic的监控可以发现，该API的延迟增大的主要原因是它依赖的一个服务响应时间增大了。

我们暂且把这个外部的服务称为service.mycompany.com，这个服务分别部署在澳洲和欧洲的两个数据中心，入口处是Akamai，做负载均衡，尽可能的按照访问来源去分发请求。

该微服务部署在AWS悉尼的数据中心，所以理论上来讲，当它请求service.mycompany.com时，Akamai应该返回的是位于悉尼的edge节点的IP，同时其访问的origin服务器也应该位于悉尼。但是通过在该微服务的服务器debug，发现ping值以及traceroute的值都比较高，办公室访问却都一切正常。当时怀疑是Akamai的GEOIP判断出了问题，把来自亚马逊悉尼的请求当成了来自美国的IP的请求，于是用部署于欧洲的数据中心的服务处理请求。和基础设施部门管理网络的人讨论，再次调查后结论类似。

问题出在这个AWS账户下的VPC的DHCP options的配置。因为是比较早期使用的share的AWS账户，所以下面的网络配置比较复杂，配置有Direct Connect 连往其他数据中心，以及很多VPC Peering。不知道因为什么原因，这个微服务部署的Cloudformation template里面选择了包含google DNS `8.8.8.8`和`8.8.8.4`的DHCP Options。我们都知道对于在Akamai上注册的服务service.mycompany.com来说，如:
```
 ~> host service.mycompany.com
service.mycompany.com is an alias for mycompany.generic.edgekey.net.
mycompany.generic.edgekey.net is an alias for e8888.g.akamaiedge.net.
e8888.g.akamaiedge.net has address 104.116.190.24
```
第一次DNS请求返回的记录是CName，之后进一步返回Akamai动态DNS的CName，也就是edge server的CName，之后再根据DNS服务器返回对应的edge服务器的IP地址，如果查询的是Google的DNS，那么它会返回美国的edge服务器地址……。我们可以测试下:

```
 ~> dig @8.8.8.8 service.mycompany.com

; <<>> DiG 9.8.3-P1 <<>> @8.8.8.8 service.mycompany.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44304
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;service.mycompany.com.	IN	A

;; ANSWER SECTION:
service.mycompany.com. 86399 IN	CNAME	mycompany.generic.edgekey.net.
mycompany.generic.edgekey.net. 263	IN	CNAME	e8888.g.akamaiedge.net.
e8888.g.akamaiedge.net.	19	IN	A	23.53.156.156

;; Query time: 603 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Tue Nov 15 22:15:52 2016
;; MSG SIZE  rcvd: 130
```
查询下IP地址信息，
```
 ~> whois 23.53.156.156 | grep Country
Country:        US
```
所以，这个微服务的请求先到Akamai美国的edge服务器，之后很有可能请求被发送到了欧洲的origin服务器，这个延迟不增加才👻了……。

解决的办法很简单，更新配置，DHCP Options选择Amazon提供的DNS就可以了，响应时间就降下去了。

这个事情给我们的教训就是，不管怎么样都不能崇洋媚外，虽然澳洲一直follow美国，但是DNS还是得用自己的。
