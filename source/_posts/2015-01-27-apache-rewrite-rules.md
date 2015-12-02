---
layout: post
title: "apache rewrite rules"
description: "apache"
category: "apache"
tags: [apache]
---


### 什么是Apache
----
[Apache](http://httpd.apache.org/)是一款开源的HTTP服务器，长期占据服务器市场第一名的位置。

### 什么是URL rewriting
----

有时候，为了让URL对用户友好，方便记忆，或是SEO优化，我们会使用更加简单的URL，比如http://example.com/help.html
但是实际开发人员命名的静态html文件名为help_2014_12_12.html,在这种情况下，就需要重写URL，让服务器
返回正确的页面(只是个例子)，亦或是原来的页面更换了新的域名，需要告诉搜索引擎或者浏览器新的地址，都需要
重写URL。URL重写后，会返回给浏览器重写后的链接以及3xx的返回码，然后浏览器会请求重写后链接的内容。

### 301 和 302的区别
----

- *301*: 永久性转移(Permanently Moved)
- *302*: 暂时性转移(Temporarily Moved)

从效果上来讲它们都会造成跳转，区别在于SEO。如果是301跳转，那么搜索引擎将A页面的Pagerank导入
到跳转后的B页面，搜索引擎以后会忽略A页面，只在搜索结果中显示B页面。如果是302跳转，搜索引擎会保留A页面
的PR，但是显示的内容可能B页面的。

### Apache rewrite module
----

[mod_rewrite](http://httpd.apache.org/docs/current/mod/mod_rewrite.html)是Apache中提供得基于规则的rewriting引擎，加载模块后重启apache就可以使用了。

``` bash
vagrant@vagrant-ubuntu-trusty-64:~$ apachectl -t -D DUMP_MODULES | grep rewrite
# 如果没有加载
vagrant@vagrant-ubuntu-trusty-64:~$ a2enmod rewrite
Enabling module rewrite.
Could not create /etc/apache2/mods-enabled/rewrite.load: Permission denied
vagrant@vagrant-ubuntu-trusty-64:~$ sudo a2enmod rewrite
Enabling module rewrite.
To activate the new configuration, you need to run:
service apache2 restart
vagrant@vagrant-ubuntu-trusty-64:~$ less /etc/apache2/mods-enabled/rewrite.load
LoadModule rewrite_module /usr/lib/apache2/modules/mod_rewrite.so
```
在`http.conf`文件中可以引用具体的rewrite规则文件，如下

```bash
RewriteEngine on
RewriteLog /var/log/www/server.rewrite.log
RewriteLogLevel 1
Include /web/conf/httpd/conf.d/rewrites.conf
```
根据需求将rewrite的规则写入`rewrites.conf`中即可。

### rewrite配置文件的写法
基本上来说，重写URL就是在`􏰭􏰆􏰖􏰉􏰃􏰏􏰆􏰓􏰊􏰕􏰌RewriteCond`下，执行自定义的`RewriteRule`，通过Server Variables
以及正则等匹配的方式，返回结果以及正确的返回码。

## 域名的转换

``` apache
RewriteCond  %{HTTP_HOST}   ^www\.example\.com$  [NC]
RewriteRule  ^(.*)$         http://www.example2.com/$1  [R=301,L]
```
第一行是触发URL重写的条件，Flag `NC` 表示匹配时无视大小写，第二行表示在前一个条件满足时重写URL，
同时返回`301`, `L`表示本次重写URL的行为结束。

### 临时更改页面地址

``` apache
RewriteRule ^page.html$   new_page.html   [R=302,L]
```

### 更改 Query String

``` apache
RewriteCond %{QUERY_STRING} a=something
RewriteCond %{QUERY_STRING} b=(else1|else2)
RewriteRule ^/some_query_here$ http://www.example3.com/redirect? [R=301,L]
```
在Query String匹配 "a=something"的前提下，如果后面的Query String匹配`b=else1`或者`b=else2`
则重写URL，`?`表示截断URL。

最近在做一张卡的时候，要重写的URL类似

```
http://www.example.com/#category=Water Ball
```
如果直接将URL写在RewriteRule里面，Apache缺省会将URL中的特殊字符转义，比如会变成

```
http://www.example.com/%23category=Water%20Ball
```
要防止Apache将其转义，可以在Flag中加`NE`，同时注意Flag之间不能有空格。

``` apache
RewriteCond %{QUERY_STRING} b=something
RewriteRule ^/some_query_here$ http://www.example.com/#category=Walter\ Ball? [R=301,NE,L]
```
### 体会
Apache的URL重写规则感觉还是比较简单，而且很强大。

### references
1. [rewriting for beginners](https://www.addedbytes.com/articles/for-beginners/url-rewriting-for-beginners/)
2. [rewrite cheat sheet](http://www.addedbytes.com/apache/mod_rewrite-cheat-sheet/)
