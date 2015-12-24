---
layout: post
title: "更加安全和简单的方式通过堡垒机ssh"
date: 2015-12-19 14:24:30 +0800
comments: true
categories: ["ssh", "security"]
---
理想情况下的运维，是不需要ops去ssh到服务器上检查问题(包括安全问题)/日志等，这些是可以通过更好
监控,如使用newrelic,或者更好的日志收集系统，如splunk等去避免。不过现实不总是完美的，加上历史
遗留的原因，ops总是会ssh到堡垒机(bastion host)，然后跳转到目标服务器去做操作。

于是，就有很多人(包括我)在堡垒机上生成key/pair, 而且private key很少加密(包括我)，这个存在
很严重的安全风险。

一个比较合理的方式是通过ssh proxy的方式去访问目标服务器，这样不需要把key暴露给bastion，比如:

``` bash
 ~> ssh -L 3333:destination_host:22 user@bastion
```
然后再启动一个新的ssh进程去通过proxy连接:

``` bash
 ~> ssh -p 3333 user@0
```

每次这么操作略麻烦，可以通过在ssh配置文件简化：

```
Host bastion  
        HostName 192.168.1.1  
        HostKeyAlias bastion  
        LocalForward 9999 target:22  
```
那么建立proxy就只是`ssh user@bastion`就可以了，然后同理去`ssh -p 9999 user@0`。
这么做的坏处在于`~/.ssh/config`配置可能会迅速膨胀，同时，每次还是启动两个进程去完成这件事情，不开心。

于是，我们的安全大神介绍一个更加简单的方法，在`~/.ssh/config`中，加入下面的内容:

```
Host */*   
        ProxyCommand ssh $(dirname %h) -W $(basename %h):%
```
如此我就可以通过`ssh user@bastion/target`的方式直接ssh到远程主机，`ProxyCommand`指令会
生成两个进程，后台proxy进程，前台的进程直接通过proxy连接到目标主机。这样从命令行窗口看来我只
是打开了一个会话。同时，你可以链接很多个主机，如`ssh user@bastion/targetA/targetB/targetC`。
依次通过前一个主机建立的proxy连接到后面的主机上。

这个方法有一些局限：

1. 不能在主机链上指定不同的端口；
2. 不能对不同的主机使用不同的登录用户名；
3. 不同链上建立的连接不能重用已经建立的连接，这可能会导致连接的速度减缓；
4. 其实还有个问题就是不能很容易的从`targetC`退出到 `targetB`…… (我想的)

为了解决这些问题，大神想出了终极解决方案:
```
Host */*  
    ControlMaster auto  
    ControlPath   ~/.ssh/.sessions/%r@%h:%p  
    ProxyCommand /bin/sh -c 'mkdir -p -m700 ~/.ssh/.sessions/"%r@$(dirname %h)" && exec ssh -o "ControlMaster auto" -o "ControlPath   ~/.ssh/.sessions/%r@$(dirname %h):%p" -o "ControlPersist 120s" -l %r -p %p $(dirname %h) -W $(basename %h):%p'
```

1. `Host */*`: 匹配ssh到`A/B/X`这样的主机类型，然后递归的ssh到链中的主机；
2. `ControlMaster auto`: 这个指令的意思是指ssh应当复用已有的control channel连接远程主
机，如果这样的channel不存在，则重新创建，以便以后的链接复用；
3. `ControlPath ~/.ssh/.sessions/%r@%h:%p`: 这个指令告诉ssh control channel socket
文件的位置。对于每个远程主机，socket文件应该是唯一的，如此我们可以重用已有连接并且跳过验证。所
以我们用`%r`(remote login name),`%h`(remote host name)和`%p`(端口)作为文件名的部分。
唯一的问题是因为路径中的`/`，这里会在`%h`被当成一个目录，但是ssh不会自动创建目录；
4. `ProxyCommand blah`: 命令开始时就先创建了所有必须的目录。 `ControlPersist`的意思是如果
control channel 2分钟内没有活动则停止ssh进程。如果你有两个会话`bastion/HostA`和
`bastion/HostB`，如果不配置`ControlPersist`，结束第一个进程时第二进程也会同时被干掉。


所以，当你用上面的配置去`ssh user@bastion/A/B/C`时:

1. ssh 匹配到了`*/*`模式
2. ssh 尝试重用`~/.ssh/.sessions/user@bastion/A/B/C:22`的socket，如果成功则建立连接，
没有则继续执行
3. ssh执行`ProxyCommand`中的内容， 创建目录同时递归的ssh到最终的主机C
4. 然后ssh在主机C上进行身份验证，成功则创建`~/.ssh/.sessions/user@bastion/A/B/C:22`的
control channel socket文件，并且成为control channel的master
5. 显示命令行提示符

你现在有没有和我一样晕，在和大神交流一番后，大神告诉我一个改进版的配置:

```
Host */*  
        ControlMaster auto  
        ProxyCommand    /usr/bin/ssh -o "ControlMaster auto" -o "ControlPath ~/.ssh/.sessions/%%C" -o "ControlPersist 120s" -l %r -p %p $(dirname %h) -W $(basename %h):%p

Host *
        ControlPath     ~/.ssh/.sessions/%C
```
这个配置要简单些，不过他假设你已经创建了`~/.ssh/.sessions`目录。

荣耀归于Dmitry大神，虽然那个ssh keypair我还没有删除……。
