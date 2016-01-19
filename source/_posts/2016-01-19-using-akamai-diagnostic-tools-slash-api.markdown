---
layout: post
title: "using akamai diagnostic tools/API"
date: 2016-01-19 13:32:46 +0800
comments: true
categories: ["akamai", "diagnostic"]
---

有时候在Akamai上提交应用修改后，因为配置的问题，可能出现错误，像下面这样：

```
#30.657008d1.1452737568.1e40544
```

通过日志查找的方式去发现具体的问题可能会很耗时，因为需要等待akamai把日志上传。Akamai自己提供了解码错误代码的工具和API，具体的用法如下：


### Lunar Control Centre 的 Diagnostic Tools
----
这个比较容易，从`Luna Control Center`选择`Resolve` => `Diagnostic Tools`。在`Service Debugging Tools`部分选择`Error Translator (Reference#)`，然后在`Error String:`的input中输入错误码的字符串，点击`Analyze`，等待一会就可以看到详细的错误信息以及原因。

### 使用Akamai Diagnostic API
----

1. Akamai提供了Sample Client去调用API，除了clone client的repo，还可以直接使用docker，直接运行`docker run -it akamaiopen/api-kickstart /bin/bash`既可。
2. 生成新的client请求的token。首先在`Luna Control Center`选择`CONFIGURE` => `Manage APIs `进入Open API 管理页面。在`Luna APIs`下面添加新的collection，然后在该collection添加新的client，就可以拿到新的tokens，点击右上角的导出按钮，就可以将其导出到一个文本文件中，如名为`api-kickstart.txt`的文件。
3. 在client端设置token。在client的目录下运行`python gen_edgerc.py -s default -f api-kickstart.txt`, 它会在用户根目录生成`~/.edgerc`的credential文件。通过`python verify_creds.py` 可以验证credential的有效性。`.edgerc`文件中的token其实也就是api请求时authorization的headers。
4. 测试请求。`.edgerc`文件设置验证完成后，可以使用`python diagnostic_tools.py`来测试，它实际请求的API endpoint是`/diagnostic-tools/v1/locations`和`/diagnostic-tools/v1/dig`,返回如下：

```
root@16119b2d4eb8:/opt/examples/python# python diagnostic_tools.py

Requesting locations that support the diagnostic-tools API.

There are 72 locations that can run dig in the Akamai Network
We will make our call from Adelaide, Australia

; <<>> DiG 9.8.1-P1 <<>> developer.akamai.com -t A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12919
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 8, ADDITIONAL: 8

;; QUESTION SECTION:
;developer.akamai.com.		IN	A

;; ANSWER SECTION:
developer.akamai.com.	300	IN	CNAME	san-developer.akamai.com.edgekey.net.
san-developer.akamai.com.edgekey.net. 21600 IN CNAME e4777.dscx.akamaiedge.net.
e4777.dscx.akamaiedge.net. 20	IN	A	23.4.164.144

;; AUTHORITY SECTION:
dscx.akamaiedge.net.	4000	IN	NS	n6dscx.akamaiedge.net.
...............

```
Akamai的diagnostic API的列表在[这里](https://developer.akamai.com/api/luna/diagnostic-tools/uses.html)。ErrorCode解释的endpoint是`/diagnostic-tools/v1/errortranslator{?errorCode}`，通过重用例子中的python代码即可发起这样的请求，比如把`diagnostic_tools.py`修改如下（我就是这么懒）：

```
+ location_result = httpCaller.getResult('/diagnostic-tools/v1/errortranslator?errorCode=30.657008d1.1452737568.1e40544')
- location_result = httpCaller.getResult('/diagnostic-tools/v1/locations')
+ print location_result["errorTranslator"]["reasonForFailure"]
```
之后就可以看到错误的原因是`ERR_FWD_SSL_HANDSHAKE&#x7c;err_conn_strict_cert`，也就是说我没有在CDN设置正确的certificate，导致它和origin的ssl handshake失败了。

如果没有什么特殊的需求，akamai web console中的diagnostic tool就可以满足需求，逼格较高或者有自动化需求的可以从命令行调用API输出错误原因。
