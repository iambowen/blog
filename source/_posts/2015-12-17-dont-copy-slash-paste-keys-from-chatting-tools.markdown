---
layout: post
title: "don't copy/paste keys from chatting tools"
date: 2015-12-17 16:58:30 +0800
comments: true
categories: ["ssh", "practice"]
---

今天，别的组的同事过来问我一个关于SSH的问题，问题是这样的:

1. 客户把AWS的ssh instance的private key通过slack拷给了同事；
2. 同事发现用部署工具[fabric](http://www.fabfile.org/)可以使用该key，ssh到EC2的instance上进行部署；
3. 但是如果使用key去ssh(如`ssh -i key user@instance`)到EC2的instance，就会提示输入`passphrase`
4. 客户的Ops很肯定说这个private key没有加`passphrase`

这个问题很有趣，我先查看了下key，在我的印象里，如果在`ssh-keygen`的时候加入密码保护了，private
key 中会有如下的额外信息:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,B88893260B6CCFDC6304101075B74A9F
.....
```
但是同事给我的private key中没有.

在不同的系统下尝试用该key去ssh到EC2 instance得到的结果都是需要输入passpharse,通过输入冗余
ssh信息`ssh -vvvv`也没有看到什么有用信息(其实是我忽略了)。

google搜索，猜测会不会是key generate时候的格式不同导致的，但是觉得这种可能性不高。

在客户的ops channel询问了下，有人给出了这个建议，用
```
openssl rsa -text -noout -in KEYFILE
```
去检查key的完整性,返回结果如下:

```
[vagrant@localhost ~]$ openssl rsa -text -noout -in id_rsa
unable to load Private Key
140516793460640:error:0906D066:PEM routines:PEM_read_bio:bad end line:pem_lib.c:802:
```
实际上这个信息已经比较明显了，另外有人也从ssh debug的信息中指出:

```
debug1: key_parse_private2: missing begin marker
debug1: key_parse_private_pem: PEM_read_PrivateKey failed
```
private key的开始或者结束的marker出问题了，于是客户询问这个key是不是从slack拷贝过去的，因为
聊天工具有时候会自动纠错，把结束的marker `----`自动改成`——`，他曾经就遇到过这种情况。
再看一遍private key，果然是这样……好羞愧。修改后，果然可以顺利ssh 到instance上了。
(更正下，虽然他指出了问题的来源，但是这段debug信息，在private key是完整的情况下仍然存在，所以这不是key出错的绝对证据。)

从这个事情中，我们可以得到一些教训

1. 不要用聊天工具copy/paste private key或者代码之类的东西，很容易引起错误。
2. 同时，这种方式也很不安全，尽量不要这么做，要么在传递完后迅速删除聊天记录
3. 或者使用一个公共的key管理的服务，如[rattic](http://rattic.org/)，或者使用临时生成的credential来ssh，如这个[项目](https://github.com/realestate-com-au/sshephalopod)进一步提高安全性。
