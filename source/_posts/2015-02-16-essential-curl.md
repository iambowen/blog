---
layout: post
title: "essential curl"
description: "curl"
category: "ops"
tags: ["linux", "curl", "ops"]
---


### 什么是curl
---
[curl](http://curl.haxx.se/)是我们在Linux下常用的下载文件或者测试web服务的工具，罗列下他们常
用的一些场景，当做总结。curl支持多种网络协议,包括最新的HTTP/2。

### 获取网页源码
----

```bash
~> curl www.google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com.hk/url?sa=p&amp;hl=zh-CN&amp;pref=hkredirect&amp;pval=yes&amp;q=http://www.google.com.hk/%3Fgws_rd%3Dcr&amp;ust=1423205620492528&amp;usg=AFQjCNGJFP3kOpmF9vbNWK_cgRr7G5yffQ">here</A>.
</BODY></HTML>
```

### 文件下载
----
`curl`缺省会输出文件的内容，用下面的命令可以用`curl`将文件保存到指定的目录，curl比较苦逼的地方在于，
如果不是保存当前目录，需要自己指定要保存的文件名。

```bash
~> cd /opt/ && curl -O http://ergonlogic.com/files/boxes/debian-current.box
~> curl -o /opt/current.box http://ergonlogic.com/files/boxes/debian-current.box
```

### 多线程下载
----

`curl`不支持多线程下载，我用过的一个多线程下载的工具是[aria2](http://aria2.sourceforge.net/)，蛮好用的，而且这货还支持bt,ed2k等p2p协议下载。

```bash
~> aria2c -x5 https://github.com/jose-lpa/veewee-openbsd/releases/download/v0.5.5/openbsd55.box
```
### 通过代理访问资源
----

有两种方式可以让`curl`使用proxy，第一种是通过设置环境变量，`http_proxy, https_proxy`等，用`no_proxy`去设置屏蔽代理的列表。

```bash
~> export http_proxy="http://proxy:3128"
```
让proxy对一些地址无效。

```bash
~> export no_proxy="127.0.0.1,localhost.localdomain"
```

另外一种是在运行命令时直接指定使用的proxy或者不使用proxy。

```bash
~> curl -x "http://proxy:3128" www.google.com
~> curl -x "socks5://proxy:3128" www.google.com
~> curl --noproxy * www.google.com
```
curl的`--noproxy`是需要指定一个exclude的列表。

### 花式下载和断点续传
----
对于很规则的下载url，可以通配符或者正则来处理，比如我想下载自己博客的几个图片。

```bash
~> curl -O http://iambowen.github.io/images/[1-3].png

[1/3]: http://iambowen.github.io/images/1.png --> 1.png
--_curl_--http://iambowen.github.io/images/1.png
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                Dload  Upload   Total   Spent    Left  Speed
100  222k  100  222k    0     0  59816      0  0:00:03  0:00:03 --:--:-- 59808

[2/3]: http://iambowen.github.io/images/2.png --> 2.png
--_curl_--http://iambowen.github.io/images/2.png
100  342k  100  342k    0     0  33117      0  0:00:10  0:00:10 --:--:-- 48607

[3/3]: http://iambowen.github.io/images/3.png --> 3.png
--_curl_--http://iambowen.github.io/images/3.png
100  208k  100  208k    0     0  32245      0  0:00:06  0:00:06 --:--:-- 39914
```
断点续传,除了保证文件完整性，还可以检查文件是否有变动，如果有变化则更新，没有则保持文件原有状态。

```bash
~> curl -O -C http://ergonlogic.com/files/boxes/debian-current.box
```


### 获取headers信息
----
有时候，我们只需要看到返回的headers信息，查看cache是否命中，或者返回码。

```bash
~> curl -I www.baidu.com
HTTP/1.1 200 OK
Date: Tue, 10 Feb 2015 08:25:12 GMT
Content-Type: text/html; charset=utf-8
Connection: Keep-Alive
......
```

传说使用`telnet`命令可能拿到更多的http headers信息，无法验证，方式如下。

```bash
~> telnet 0 4000
Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.
GET / HTTP/1.1
HOST: 127.0.0.1:4000

HTTP/1.1 200 OK
Etag: 232a2c-320c-54d9ce22
Content-Type: text/html
Content-Length: 12812
Last-Modified: Tue, 10 Feb 2015 09:23:46 GMT
Server: WEBrick/1.3.1 (Ruby/1.9.3/2014-11-13)
Date: Tue, 10 Feb 2015 09:29:24 GMT
Connection: Keep-Alive
```
### follow redirection信息
----
有时候访问的资源，可能被临时或者永久转移，所以会有中间跳转的过程，但是直接去`curl`是拿不到完整信息的。

```bash
~> curl -I www.google.com
HTTP/1.1 302 Found
Location: http://www.google.com.hk/url?sa=p&hl=zh-CN&pref=hkredirect&pval=yes&q=http://www.google.com.hk/%3Fgws_rd%3Dcr&ust=1423564147298386&usg=AFQjCNHZ7kXJXOfOQyCSOQ6ZGOPjeVaIYg
Cache-Control: private
Content-Type: text/html; charset=UTF-8
......
```

通过下面的方式可以拿到所有的返回信息。

```bash
~> curl -LI www.google.com
HTTP/1.1 302 Found
Location: http://www.google.com.hk/url?sa=p&hl=zh-CN&pref=hkredirect&pval=yes&q=http://www.google.com.hk/%3Fgws_rd%3Dcr&ust=1423559822825114&usg=AFQjCNEEY2qyq9HghaQVZ89ugMv9kvDlLA
......

HTTP/1.1 302 Found
Location: http://www.google.com.hk/?gws_rd=cr
......

HTTP/1.1 200 OK
Date: Tue, 10 Feb 2015 09:16:35 GMT
Expires: -1
......
```
### 加入验证的请求
----
有时候在请求一些资源时，需要通过验证才能完成访问，可以用`username:password`加URL即可。

```bash
~> curl http://user:password@echo.httpkit.com?queryString
```

### 伪装user agent的请求
----
有的服务器可能会限制访问的User-Agent类型，用curl测试的时候可以用相应得参数进行伪装。

```bash
~> curl  echo.httpkit.com
 "headers": {
   "host": "echo.httpkit.com",
   "user-agent": "curl/7.37.1",
   "accept": "*/*"
}

~>  curl -A "Bad Ass"  echo.httpkit.com
 "headers": {
   "host": "echo.httpkit.com",
   "user-agent": "Bad Ass",
   "accept": "*/*"
 },
```

### 发起HTTP请求
----
用`-X`可以指定发起请求的HTTP方法, 如果用`POST`或者`PUT`等方法，可以用 `-d`指定request body，`-H`可以指定请求的一些Headers。

```bash
~> curl -X PUT -H 'Content-Type: application/json' -d '{"firstName":"Kris", "lastName":"Jordan"}' echo.httpkit.com
{
 "method": "PUT",
 "uri": "/",
 "path": {
   "name": "/",
   "query": "",
   "params": {}
 },
 "headers": {
   "x-forwarded-for": "210.74.157.146",
   "host": "echo.httpkit.com",
   "user-agent": "curl/7.37.1",
   "accept": "*/*",
   "content-type": "application/json",
   "content-length": "41"
 },
 "body": "{\"firstName\":\"Kris\", \"lastName\":\"Jordan\"}",
 "ip": "127.0.0.1",
 "powered-by": "http://httpkit.com",
 "docs": "http://httpkit.com/echo"
 }
```

### 上传文件
----

```bash
~> curl --form "fileupload=@filename.txt" http://hostname/resource
```

### 处理cookies
----
保存cookies到本地

```bash
 ~> curl -c echo.cookies  http://www.baidu.com > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 88672    0 88672    0     0   298k      0 --:--:-- --:--:-- --:--:--  297k
 ~> less echo.cookies
# Netscape HTTP Cookie File
# http://curl.haxx.se/docs/http-cookies.html
# This file was generated by libcurl! Edit at your own risk.

.baidu.com      TRUE    /       FALSE   3571557287      BAIDUID 287BE3DB1621FDE977C39D21BD02CA45:FG=1
.baidu.com      TRUE    /       FALSE   3571557287      BAIDUPSID       287BE3DB1621FDE977C39D21BD02CA45
www.baidu.com   FALSE   /       FALSE   0       BDSVRTM 0
www.baidu.com   FALSE   /       FALSE   0       BD_HOME 0
.baidu.com      TRUE    /       FALSE   0       H_PS_PSSID      10422_1449_11089
```
使用cookie 发起请求

```bash
~> curl -b echo.cookies  http://www.baidu.com > /dev/null
```
### fail on error
----
对于服务器端错误，http请求没有任何输出，`curl`的返回为0，使用`-f`参数可以在遇到服务器错误是返回非0(22)。
最近在完成一个用Bamboo host的package去部署的任务的时候用到了这个参数，一旦下载package出错，部署会中断。

```bash
~> curl -f http://iambowen.github.io/ksjdkfsjdf
curl: (22) The requested URL returned error: 404 Not Found
~> echo $?
22
```

### 从浏览器"偷取"curl的命令
----
觉得`curl`指令太难记忆，可以直接从浏览器中"偷"取， Chrome提供了这样的功能:

![chrome](/images/curl.png)

### References
----
[1](http://www.ruanyifeng.com/blog/2011/09/curl.html)
[2](https://gist.github.com/303182519/132568fd0e58cae57202)
[3](http://httpkit.com/resources/HTTP-from-the-Command-Line/)
