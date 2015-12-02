---
layout: post
title: "Allow CORS under Apache"
description: ""
category:
tags: [apache]
---

处于安全的考虑，不允许Javascript跨站调用其他网站的服务。对于开放的API服务来说，允许跨站调用又是必须的。为了“解除”这个限制，需要在服务返回的header中添加`Access-Control-Allow-Origin`字段。
在Apache的配置中，`<Directory>`, `<Location>`, `<Files>` ,`<VirtualHost>`或者`.htaccess`任意一个section下添加如下内容：

```apache
  Header set Access-Control-Allow-Origin "*"
```
意为该服务允许接受任何跨站请求。当然你可以限制具体的请求网站，如：

```apache
  Header set Access-Control-Allow-Origin "www.example.com"
```
要设置允许多个请求源, 可以很没节操的，写上很多遍`Header set Access-Control-Allow-Origin "www.xxx.com"`, 或者

```apache
<IfModule mod_headers.c>
   SetEnvIf Origin "http(s)?://(www.)?(domain1.com|domain2.com)$" AccessControlAllowOrigin=$0$1
   Header add Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
</IfModule>
```
除了限制请求源，还可以限制http(s)请求的方法，添加“Access-Control-Allow-Methods”的白名单。

```apache
  Header set Access-Control-Allow-Methods "GET, POST"
```

###References
[1](http://enable-cors.org/server_apache.html)
[2](http://twlidong.github.io/blog/2013/12/22/kua-yuan-zi-yuan-gong-xiang-cross-origin-resource-sharing-cors/)
