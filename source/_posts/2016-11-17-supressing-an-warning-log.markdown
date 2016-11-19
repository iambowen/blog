---
layout: post
title: "supressing an warning log "
date: 2016-11-17 22:49:53 +1100
comments: true
categories: ["log","spring"]
---
客户的主站系统是一个基于Spring的单块系统，每天的流量大概在几千万，其所有的应用日志以及访问日志，都会上传到[Splunk](https://www.splunk.com)服务器上。随着服务数量的增加，日志的数量也越来越多，对应的Splunk的费用也越来越感人，鉴于主站的系统对日志的贡献最多，所以就从它入手降低无用日志的上传。

其中的一条无用的warning日志信息显示如下:
```
WARN org.apache.commons.httpclient.HttpMethodBase - Going to buffer response body of large or unknown size. Using getResponseBodyAsStream instead is recommended.
```
一周内这条warning的数量超过百万条，还是比较可观的。简单查询下其原因是`httpclient`里面的`getResponseBody()`调用触发的。

```java
                int limit = getParams().getIntParameter(HttpMethodParams.BUFFER_WARN_TRIGGER_LIMIT, 1024*1024);
                if ((contentLength == -1) || (contentLength > limit)) {
                    LOG.warn("Going to buffer response body of large or unknown size. "
                            +"Using getResponseBodyAsStream instead is recommended.");
                }
```
看完这个后我们的理解是，有些请求的response body过大，超过缺省的1M(代码会从`Content-Length` header中获取这个大小)，就会触发这个warning，当时没有意识到还有可能是确实不知道response body的长度。
相关的方法调用在代码中有10几处，当时我们也无法定位那段代码引发了这个问题，无脑修改的话，成本比较高，可能要增加一些测试用例，以及做回归测试。所以当时就想着用成本最低的方式修改，从配置文件中给`BUFFER_WARN_TRIGGER_LIMIT`赋一个更大的值，如20M，毕竟这是个遗留项目，熟悉代码的人以及比较少了。没有选择调整日志的级别是因为`HttpMethodBase`类是个超类，粗暴调整可能会掩盖其它有用的warning日志。
部署完成后比较日志数量发现并没有太大变化，不得不让我们重新回来审视这个问题的根本原因在哪里。幸好当时系统加了一个transactionID的功能，每次的请求过来时，在应用中用UUID生成一个transactionID写入应用日志，response返回时再写入access log。这样我们就在请求和对应的代码调用之间建立了联系。
功能上线后重新在splunk中搜索，立刻就定位了是在请求Google Map API时触发了这个warning，而且在本地可以稳定重现。

```
 ~> curl -I   "https://maps.google.com.au/maps/api/geocode/json?address=Sunnydale%2C+SA+5354&language=en_AU&sensor=false"
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Date: Fri, 18 Nov 2016 23:20:16 GMT
Expires: Sat, 19 Nov 2016 23:20:16 GMT
Cache-Control: public, max-age=86400
Access-Control-Allow-Origin: *
Server: mafe
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Alt-Svc: quic=":443"; ma=2592000; v="36,35,34"
Transfer-Encoding: chunked
Accept-Ranges: none
Vary: Accept-Encoding
```
然后发现原来返回是没有`Content-Length`的:(。
`Content-Length` header是客户端用于了解服务器返回body的大小，从而在获得等大的内容后，结束连接，节省开销。但在实际的应用中，`Content-Length`有可能无法准确反映返回body的大小，其值过大会导致pending，过小内容又会被截断。
`Transfer-Encoding: chunked` 是用来分块编码传输内容，每个分块中包含了长度值和数据，最后一个分块长度值是0，这样就可以准确知道边界了。
定位到问题在哪里之后就很容易解决了，最后只要改动一行代码:

```java
-            String response = query.getResponseBodyAsString();
+            String response = IOUtils.toString(query.getResponseBodyAsStream());
```
回过头来反思整个过程，因为是遗留系统，所以处理的方式有些粗糙，如果当时我们遵循下面的过程也许会更好些:
1) 定位问题，找到根本原因(有transactionID的配合会更方便)，而不是盲目用生产环境来测试配置的正确性;
2) 在本地重现问题，应用解决方案，并与熟悉遗留系统的同事沟通
3) 回归测试后上线
