---
layout: post
title: "GPG and keybase.io"
date: 2016-12-23 15:39:39 +0800
comments: true
categories: ["security", "keybase.io", "GPG"]
---
[GPG](https://en.wikipedia.org/wiki/GNU_Privacy_Guard)(GnuPG)是一种使用混合加密技术(对称/非对称，对称加密提升加密速度，公钥加密用来保证key交换时的安全性)的应用。它的主要作用在于:
1. 身份验证。对自己的工作签名，比如git提交，邮件等。
2. 加密。对交流的过程进行加密保护，防止被别人窃取。

如果是用`gpg`工具来生成和发布key的话，流程是这样的:

#### 生成key
---

```bash
~> gpg --gen-key
gpg (GnuPG) 1.4.19; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Sat Dec 23 16:20:33 2017 CST
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"
Real name: Rich Dickson
Email address: dickson.rich@richdick.com
Comment: richdick
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.   ##用密码保护master key

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
..+++++
...+++++
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
..........+++++
........+++++
gpg: key C5071A6F marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   4  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 4u
gpg: next trustdb check due at 2017-08-02
pub   2048R/C5071A6F 2016-12-23 [expires: 2017-12-23]
      Key fingerprint = 1D98 85B1 7D28 942B A265  3BEA AFD0 0C00 C507 1A6F
uid                  Rich Dickson (richdick) <dickson.rich@richdick.com>
sub   2048R/DDAF894F 2016-12-23 [expires: 2017-12-23]
```
把这个key导出保存在一个安全的地方，比如保险柜 :)
```bash
 ~> gpg --export-secret-keys --armor "Rich Dickson (richdick) <dickson.rich@richdick.com>" > richdick-private-key.asc
```

### 把public key发布出去
----
公共的key server可以选`hkp://keys.gnupg.net`或者[keybase.io](https://keybase.io). Keybase用起来比较方便，下面以它来举例:

```bash
brew install keybase

keybase login

keybase pgp select --multi
#    Algo    Key Id             Created   UserId
=    ====    ======             =======   ======
2    2048R   AFD00C00C5071A6F             Rich Dickson <dickson.rich@richdick.com>
Choose a key: 2
▶ INFO Bundle unlocked: AFD00C00C5071A6F
▶ INFO Generated new PGP key:
▶ INFO   user: Rich Dickson (richdick) <dickson.rich@richdick.com>
▶ INFO   2048-bit RSA key, ID AFD00C00C5071A6F, created 2016-12-23
▶ INFO Key AFD00C00C5071A6F imported
```
我有个朋友叫Logan，我在keybase上搜索下它:

```bash
~> keybase search logan
loganhan twitter:logan_han github:logan-han coinbase:loganhan dns://han.life
```

假设我现在需要给它传递一段加密信息，那么可以这样:

```bash
 ~> echo 'Merry Xmas, Logan!' > secrets.txt
 ~> keybase encrypt -i secrets.txt -o secrets.txt.asc loganhan
▶ INFO Identifying recipient loganhan
✔ <tracked> public key fingerprint: BFC6 69B1 861B 8A71 044A 70F1 5E6A 5830 E81F 7EA2
You last followed loganhan on 2016-08-03 10:09:33 CST
 
```
加密后的内容如下:

```bash
BEGIN KEYBASE SALTPACK ENCRYPTED MESSAGE. kiQrUWZvE8pSlIf jYX6ZmEkk4Gba7a 
....
3gZJb6yMWYBnlCU VCVZhPh2MvRiWne lhaaSZViJOXr407 Wt7NXT9RU6Mb96M 28XkWJMHHE60iwD ASc8SRp0HDuMCMg CXx6bMqIWf68ZfT yL19g5vgAGaQi8b yyY7tFMpJ5d5GJe eTa0ZP4AiI20Ez5 4eAZKoy9D4L3R77 ezDwAo5OscyQTUY voRvduVQoWhET3p w9eF13YSk2tajBq 7OuopZZOS1RjE9y sKnmYXLEvaiOKw0 . END KEYBASE SALTPACK ENCRYPTED MESSAGE.
```
我的同事如果收到这个加密后的文件的话，可以用下面的命令解密:

```bash
 ~> keybase decrypt -i secrets.txt.asc
Message authored by iambowen
Merry Xmas, Logan!
```

### 验证身份
----
我觉得如果希拉里的团队知道用邮件加密的功能，就不会被钓鱼，导致邮件泄密，最终败选。GPG可以用在多个地方去验证身份，比如Twitter、DNS、github等。以github为例，在个人的setting里面可以添加GPG的public key。
首先，导出GPG的public key，并在setting页面添加:
```bash
gpg --export --armor iambowen.m@gmail.com | pbcopy
```

本地配置git的signing key:

```bash
 git config --global user.signingkey iambowen.m@gmail.com
```

提交的时候附加身份信息:

```bash
 ~> git commit -S

You need a passphrase to unlock the secret key for
user: "Bowen Ma (My GPG key) <iambowen.m@gmail.com>"
2048-bit RSA key, ID A02C6030, created 2016-06-21

[master 0a06550] Why: Testing commit with GPG
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

在github检查这次提交
```
https://github.com/iambowen/microservices-training-lesson-8-security/commit/0a06550f2738b780544af839662fd9f6dfc506b4
```
可以看到提交者的身份已经被验证过了。

其实我个人使用GPG(或者keybase这个工具)去加密的场景很少，不过如果以后对于安全的要求加强，这个应该是可以用到的。

