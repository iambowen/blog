---
layout: post
title: "Essential SSH"
description: "essential skills on ssh"
category: 'ssh'
tags: [ssh]
---
# 什么是SSH
----
 [Secure Shell (SSH)](http://en.wikipedia.org/wiki/Secure_Shell) 是一种加密网络协议，主要用作以下用途:

 - secure data communication  (安全数据传输)
 - remote command-line login  (远程命令行登陆)
 - remote command execution   (远程命令执行)
 - other secure network services between two networked computers  (节点间安全网络服务)

相比telnet，ftp这些明文传输的协议，SSH要安全的多。

# 基本工作原理
----
SSH是传输层协议，有不同的版本(有SSH1和SSH2)，SSH协议框架分为三部分:

1. 传输层协议: 提供正向加密(如果一次会话被破解，不会影响到前面会话的安全性)的服务器验证，数据安全性，数据完整性。还可以提供压缩功能。
2. 用户认证协议: 连接服务器的用户验证。
3. 连接协议: 在底层的SSH链接基础上的多路复用的多个逻辑链路。

SSH连接过程如下图：
![ssh connection](http://upload-images.jianshu.io/upload_images/2419194-68856e3d2e267700.gif?imageMogr2/auto-orient/strip)

大致可以分为如下的步骤：

1. TCP 3次握手建立连接
2. 版本号协商，让客户端和服务器端使用相同的协议版本通信
3. 客户端和服务器端为使用公钥算法，加密算法等协商，得出最终要使用的算法
4. 秘钥交换阶段，首先客户端会在 `~/.ssh/known_hosts`等文件中查找服务器主机信息
5. 如果没有找到，则提示客户端是否接受服务器签名，接受服务器的public key(`/etc/ssh/ssh_host_rsa_key.pub`),该key会被保存到`~/.ssh/known_hosts`
6. 如果找到，服务器和客户端用算法(如Diffie–Hellman算法)交换秘钥，生成会话秘钥和会话ID，其中ID用于认证过程，会话秘钥用于数据的加密解密
7. 用户认证阶段，有两种方式，一种是public key的验证方式，如果服务器在自己的`~/.ssh/authorized_keys`等文件中没有找到客户端的public key，验证失败，反之则成功；
8. public key 认证失败后，会回退到密码认证，这个过程是加密的
9. 认证完成后就是会话请求以及交互会话阶段，数据双向加密传输，服务器端执行从客户端传输的命令，然后将结果返回给客户端

以上只是ssh会话过程的一个简单描述，其实过程远比这个复杂，如果对细节比较关心，除了查阅材料，还可以让ssh打印冗余信息，来获取交互过程中具体发生的事情。

``` bash
[vagrant@bogon ~]$ ssh -vvv localhost
OpenSSH_5.3p1, OpenSSL 1.0.1e-fips 11 Feb 2013
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: Applying options for *
debug2: ssh_connect: needpriv 0
debug1: Connecting to localhost [::1] port 22.
debug1: Connection established.
debug1: identity file /home/vagrant/.ssh/identity type -1
debug1: identity file /home/vagrant/.ssh/identity-cert type -1
debug1: identity file /home/vagrant/.ssh/id_rsa type -1
debug1: identity file /home/vagrant/.ssh/id_rsa-cert type -1
debug1: identity file /home/vagrant/.ssh/id_dsa type -1
debug1: identity file /home/vagrant/.ssh/id_dsa-cert type -1
debug1: Remote protocol version 2.0, remote software version OpenSSH_5.3
..........
```

传输过程中数据一致性的保证，是通过计算所传输数据的校验码，md5等，传输完成后比对，就可以知道数据是不是完整的，有没有被更改。

## 生成key pair
----
用 `ssh-keygen`命令就可以生成一对key/pair，加密的算法和长度都是可选的，也可以选择对private key做密码保护，默认生成的key pair是在home目录的的`.ssh`目录下。

```bash
[vagrant@bogon ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): something
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in something.
Your public key has been saved in something.pub.
The key fingerprint is:
a5:0d:9d:f7:9a:c7:55:3b:8f:d3:6e:73:92:4e:b1:aa vagrant@bogon.something.com
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         . .     |
|        . + .   .|
|         = . .  o|
|        S .   oo.|
|             + *o|
|            o B.o|
|             +o+o|
|          E....o+|
+-----------------+
```

默认生成的key pair为`~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`
## 服务器端配置文件
----
对于服务器端来说，配置文件在`/etc/ssh`目录下，其中`sshd_config`包含所有的配置，选项很多，举个简单的例子，企业版的Github的ssh服务运行在不同的
端口，如61000，那么修改配置文件`sshd_config`:

```bash
Port 61000
```
重启sshd

```bash
service sshd restart
```
之后就可以通过61000端口进行ssh连接

```bash
ssh -p 61000
```

## 客户端配置文件
----
客户端的配置文件默认在`~/.ssh/`目录下，其中`known_hosts`放置remote host及其public key，`authorized_keys`文件保存允许以public key验证方式登陆的主机。
主要的配置文件是`~/.ssh/config`，其中可以设置remote host相关信息等， 如下:

```bash
Host remotehost
  User user
  Port 7777
  HostName remotehost.example.com
```
如果没有以上的配置，那么想ssh到`remotehost.example.com`就需要以下的操作:

```bash
ssh -p 7777 user@remotehost.example.com
```
加入以上配置后，就只需像下面一样:

```bash
ssh remotehost
```

## 远程命令执行
----
其实比较简单了，将要执行的命令放在最后就可以了

```bash
ssh user@remotehost 'yum install -y package'
```
这种用法常见于自动化脚本中，或者登录到服务器时不需要执行交互式命令，如重启服务，安装包之类的。

## 以public key验证方式登陆远程服务器
----
其实就是将客户端的public key加入服务器端的`~/.ssh/authorized_keys`文件中，有以下两种方式:

```bash
ssh-copy-id user@remotehost
```
或者土一点

```bash
cat ~/.ssh/id_rsa.pub | ssh user@remotehost 'cat >> ~/.ssh/authorized_keys'
```
这样就可以实现无密码登陆服务器。我们的测试环境在亚马逊的AWS上，为了使用统一的工具去远程登陆所有的主机，在用亚马逊的API生成instance时候，同时将预先生成的public key放在instance的`authorized_keys`文件中，在工具远程登陆instance时，使用了下面的命令:

```bash
ssh -i private_key user@instance
```
既可以实现对测试环境下所有主机的访问。

## Forward Agent
----

现实中有如下的一种使用场景，本机可以public key验证方式登陆remotehost1和remotehost2，但是从remotehost1到remotehost2必须以密码验证的方式登陆。这个问题可以通过forward agent解决，一种是用`ssh-add`添加private key:

```bash
ssh-add ~/.ssh/id_rsa
```
然后在ssh到remotehost1时启用forward agent:

```bash
ssh -A user@remotehost1
```
如此在登陆到remotehost1时，可以直接ssh到remotehost2而不需要输入密码，在remotehost1上面输入`ssh-add -l`可以发现本机的private key被带到了remotehost1上。

另一种是修改本机的ssh配置文件，针对remotehost1开启forward agent:

```bash
Host remotehost1
  User user
  HostName remotehost1
  ForwardAgent yes
```
就不再需要加 `-A`的选项了。

## 建立socket proxy
----
ssh强大的地方在于它可以的很容易建立起隧道或者socket代理，如果你有一个在国外的vps，分分钟就可以建立一个安全的代理，访问到墙外的网站。

```bash
ssh -D 8888 user@remotehost
```
执行完这个命令后，一个socket代理就搭建完成，在Firefox的网络设置中，配置socket代理，服务器为 127.0.0.1 端口为8888，就可以使用了。注意，
这个socket代理在Chrome下的的一些proxy插件如SwitchSharp下不工作，原因不太清楚。

## SSH 端口转发
----
为了方便访问内网的产品环境的服务器，通常我们都会设置一些跳转机(bastion)，只有登录到这台机器上时才能去访问内网的其他服务器。有这样的一种应用场景，
我希望访问在内网的staging环境的mysql_server服务器，但是我只有bastion机器的访问权限，怎么办呢，可以通过port forwarding来解决。

```bash
ssh -L 33306:mysql_server:3306 bastion -luser
```
之后用mysql客户端就可以连接本地的33306端口来访问数据库，所有的请求通过bastion机器转发给mysql_server.

```bash
mysql -h 127.0.0.1 -p 33306 -u user -p
```

除了远程的端口转发，本地也可以做类似的事情。比如现在有这么一种需求，我希望本地开发环境的服务器运行在80端口，而通常这些服务器在开发模式都运行`>1024`的端口下，这个也可以通过端口转发来完成:

```bash
python -m SimpleHTTPServer   # 启动在8000端口

sudo ssh -L 80:127.0.0.1:8000 user@127.0.0.1
```
如此就可以在浏览器输入`localhost`去访问服务，不需要再加端口号。当然，实现这个需求的方式有很多种，如iptables等，这里讲的只是其中一种方法。

## SSH 反向端口转发
----
基于IPv4地址数量以及安全的考虑，公司或者学校的网络都是通过NAT管理的，出口的IP地址可能只有几个。可以用:

```bash
curl ifconfig.me/ip
```
获取自己的NAT ip地址。
如果我想在自己家中访问到公司的网络，那么可以用反向端口转发的方式在NAT上打个“洞”。假设家里的路由器已开通了DDNS服务，域名为`home.oicp.net`，那么我可以在公司的主机上做如下的事情:

```bash
ssh -R 8888:127.0.0.1:22 user@home.oicp.net
```
如此，我就建立了一个反向连接的隧道，从家中的路由器服务器上，我做如下的操作:

```bash
ssh -p 8888 user@127.0.0.1
```
就可以访问到公司的主机了。 同理，可以通过类似的方式访问到公司内部的资源或者一些服务，前提是公司没有提供VPN，否则不推荐这种方式，不符合安全守则。

## 文件传输
----
用ssh进行文件传输，一般我们听到这个时候都会想到`scp`或者`rsync`，其实ssh也可以，只不过方式略微hack一些。

```bash
tar cvf file | ssh user@remotehost 'tar xv '
```
其实就是用tar命令将文件打包然后写到标准输入，再经由管道在远程机器解开文件。虽然这是通过加密方式传输，但始终觉得这种方式不甚靠谱，一旦传输因为网络原因断开，就得重新传输，不能续传。

## 总结
ssh常见的用法大概就是这些，详细的原理以及其他用法可以参见后面给出的一些链接。


## references
------
1. https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys
2. http://www.cisco.com/web/about/ac123/ac147/archived_issues/ipj_12-4/124_ssh.html
3. http://newfdawg.com/docs/HP-SSH_Explained.PDF
4. http://www.wildlee.org/2011_03_1305.html
5. http://zh.wikipedia.org/wiki/%E8%BF%AA%E8%8F%B2%EF%BC%8D%E8%B5%AB%E5%B0%94%E6%9B%BC%E5%AF%86%E9%92%A5%E4%BA%A4%E6%8D%A2