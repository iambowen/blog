---
layout: post
title: "forward splunk logs to syslog"
date: 2017-01-20 12:41:26 +0800
comments: true
categories: ["splunk", "syslog"]
---
从sponsor哪里收到了一个很奇怪的需求，客户需要用splunk的fowarder将日志同时输出到syslog服务器中，而且不能用别的forwarder，所以我提出的用Fluentd就被否定了。领导命令大于天，虽然syslog没用过，但是也只能硬着头皮上了。

在slack channel里面问了下，大神们纷纷表示没做过，但是splunk的文档指出`heavy forwarder`可能可以同时将日志发给第三方的日志服务器比如syslog server。理论上来说分两步:
1. 将`universal splunk fowarder`变为`heavy forwarder`，并重启服务，[文档在此](https://docs.splunk.com/Documentation/Splunk/6.5.1/Forwarding/Deployaheavyforwarder)
2. 在SplunkForwarder的`outputs.conf`添加对应的配置文件，[文档在此](http://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Forwarddatatothird-partysystemsd#TCP_data)

接下来就是得去验证这些配置是否正确，简单来讲就是启动一个splunk服务器和一个syslog服务器，然后把把任意的splunk收集的日志转发给syslog服务器。

OK，首先找了两个合适的docker image，写在一个`docker-compose`文件中，减少配置修改的成本，同时方便通信。

```yaml
version: '2'
services:
  splunk:
    image: dmaxwell/splunk
    container_name: splunk
    ports:
      - "8000:8000"
      - "9997:9997"
      - "8088:8088"
      - "1514:1514"
    links:
        - syslog:syslog 
  syslog:
    image: voxxit/rsyslog
```
用`docker-compose up -d`启动，访问`localhost:8000`可以先更改下`admin`的密码。其次attach到splunk的container上面，做下面的操作：

1. 测试能否以tcp的方式连接到syslog服务器, looks good.

```bash
docker-compose exec splunk bash

root@003b513bb0f7:/# telnet syslog 514
Trying 172.26.0.2...
Connected to syslog.
Escape character is '^]'.
```
2. 将`universal splunkfowarder`变成`heavy splunkfowarder`

```bash
/opt/splunk/bin/splunk enable app SplunkForwarder -auth admin:password

#重启splunk服务
 /opt/splunk/bin/splunk restart
```
3. 在splunkforwarder配置文件`opt/splunk/etc/apps/SplunkForwarder/default/outputs.conf`中添加下面的内容:

```yaml
[syslog]
defaultGroup=syslogGroup

[syslog:syslogGroup]
server = syslog:514
```
因为是spike，所以没有过滤日志内容。很怂的又重启了一次 -_-！

4. 登陆到syslog服务器上，看不太懂`/etc/rsyslog.conf`内容，不过tail了下`/var/log/messages`，哇塞竟然看到来自splunk的日志了。

所以从splunkforwarder发送日志到syslog服务器是没有问题的，文档诚不欺我。但是怎么把它保存到不同的，我不禁陷入了沉思, hmm......。

显然沉思没有文档有用-_-! 在[这里](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-basic_configuration_of_rsyslog.html#s2-Templates)发现似乎可以通过过滤消息内容来将日志保存在其它文件中。
比如像这样把下面的配置加到`/etc/rsyslog.conf`中，目的是

```yaml
:msg, contains, "splunk"   action(type="omfile" file="/var/log/splunk.log")
```
反复测试后，又成功了……，yeah!

但是这种粗暴的方式很容易把无关的日志也保存到一起，如果可以通过某种方式可以鉴别日志的来源或者格式来过滤会不会更好些？从日志format上来区分的方式我没有尝试成功，但是通过给远程主机来定制一个模板方式成功了。比如像下面这样在`/etc/rsyslog.d/`加一个`splunk.conf`文件，内容如下:

```yaml
## preserve sending host fqdn
$PreserveFQDN on
#
# Receive outside syslog
$template RemoteHost, "/var/log/%HOSTNAME%/1.log"
$RuleSet remote
*.* ?RemoteHost

#Listeners
$InputTCPServerBindRuleset remote
$InputTCPServerRun 5144
```
这里我让syslog监听另外一个端口，对应的也需要在splunk的`outputs.conf`中添加新的syslog server配置。各种重启之后成功的看到了不断更新的`/var/log/splunk.splunk_default/1.log`日志文件。
基本上一个初步的解决方案应该已经形成了，简单的阅读了rsyslog文档，了解下配置，差不多就是这样了，毕竟好像有`s/1hours/3hours/g`没有给老婆打电话了，`docker-compose down`，然后瑟瑟发抖的拿起了电话……。