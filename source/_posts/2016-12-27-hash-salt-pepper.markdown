---
layout: post
title: "hash salts pepper"
date: 2016-12-27 09:34:24 +0800
comments: true
categories: ["security", "salt", "pepper", "hash"]
---

通常用户登录验证的方式是用户名和密码，用户和网站都拥有登陆名和密码，这个时候，对于用户密码的存储就变成了一个安全问题。
早期的验证框架的实现是把密码以明文的方式保存到数据库，但是一旦数据库被SQL injection攻击，密码信息泄露，用户的信息就会赤裸裸的暴露在互联网上面。典型的例子就是几年前的CSDN。这种情况下，很难能找到一种合适的方式，让我们有足够的时间通知用户来重新设置密码修复安全问题。
为了给我们多争取点时间，一种应对的策略是采用加密哈希函数(cryptographic hash function)加密用户密码后保存到数据库，其哈希后的值也被称为签名、校验或者哈希值。
这样用户在发送密码到服务器时，服务器只需要计算下它的哈希值是否和表中的密码值一致就可以判断是否可以合法登陆了。
比如在MySQL中保存的密码:

```bash
 ~> mysql -u root -e 'select PASSWORD("password");'
+-------------------------------------------+
| PASSWORD("password")                      |
+-------------------------------------------+
| *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 |
+-------------------------------------------+
```
用户在登陆时，输入密码`password`, 应用通过同样的哈希函数计算密码的哈希值并与数据库中的密码字段比较，匹配则登陆成功，反之则失败。这样如果密码泄露，在黑客使用暴力破解或者字典攻击破解密码之前，我们也许可以有稍多点的时间来通知用户修改密码。
当然，黑客也可以加密哈希函数预先计算值并生成彩虹表(Rainbow Table)。了解到实现，黑客就可以以空间换时间的方式，先计算出密码的哈希值，然后反查密码，随着这个表的增长，破解的难度可能会降低，时间也会减少。
比如MySQL中它是通过sha1加密的:

```bash
 ~> mysql -u root -e  "select password('password'), concat('*', ucase(sha1(unhex(sha1('password')))));"
+-------------------------------------------+---------------------------------------------------+
| password('password')                      | concat('*', ucase(sha1(unhex(sha1('password'))))) |
+-------------------------------------------+---------------------------------------------------+
| *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 | *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19         |
+-------------------------------------------+---------------------------------------------------+
```
如果黑客以同样的算法，计算了`password`的哈希值，那它就可以在O(1)的时间内知道密码。

要增加破解的难度，让我们在密码泄露时多争取点时间，其中的一种方式就是给密码中加点盐(salt)，在生成密码的哈希值时，加入一个随机的字符串(salt)，然后保存在数据库中。这样对于相同的密码，在数据库中的保存的记录也是不同的。这样对于使用彩虹表破解的黑客来说，破解的成本很高，因为他需要猜测混合salt的算法，同样，暴力破解也变的不太可能。
在密码比较时，计算用户输入的哈希值，然后和数据库中去掉salt的部分记录比较，就可以验证是否是合法用户了。我们以python下`bcrypt`为例:

```python
pip install bcrypt

>>> hashed_password = bcrypt.hashpw("password", bcrypt.gensalt())
>>> print hashed_password
$2b$12$K947InrSXM6XvNoErbAcj.K5YQ/OSVvJ802MxSWNgXdrjmru8Grs2
```
这是用Blowfish密码生成的哈希值字符串再进行base64编码的结果，共分为4个部分，`$2b$`表明这是`bcrypt`格式的哈希，第二个是成本(cost)值，默认是12，第三部分是22位的字符串，也就是salt的值，剩下的部分就是密码哈希后的base64编码的值。在比较的时候，只需要把哈希后的密码当做salt传进去就可以了。

```python
>>> hashed_again = bcrypt.hashpw("password", hashed_password)
>>> hashed_again == hashed_password
True
```
`hashpw`的[源码](https://github.com/pyca/bcrypt/blob/master/src/bcrypt/__init__.py#L82-L83)以及它调用的C类库的[`bcrypt_hasspass`](https://github.com/pyca/bcrypt/blob/ecc3d9642a7f6d862b4cac77e73d67266f5763b8/src/_csrc/bcrypt.c#L64-L173)可以看出，在实际的比较中，它进行了版本的校验以及截断，重用了`hashed_password`中的salt值，所以最后生成的字符串是一样的。
Blowfish是一个symmetric-key块分组密码，对称性key的意思用同样的加密秘钥去加密解密，就像以前谍战片中用相同的密码本去解密消息。块分组的意思就是将明文分为固定长度的块，用秘钥分别加密后再拼起来，并且密文应该和明文的长度相同。Blowfish的分组块大小为64bit，key的长度范围从32bits到448bits，它使用的是16轮变换的[Feistel cipher](https://en.wikipedia.org/wiki/Feistel_cipher)，以及基于PI生成的[S-boxes](https://en.wikipedia.org/wiki/S-box)，从wiki看内容描述，一直不知道为什么8bits的输入最后产生的是32bits的输出，后来看了伪码后才发现它不是用的S-boxes描述的二维数组去取值的:(……。

进一步加强密码的强度的方式是给密码再撒一把胡椒(pepper)，简单来说就是服务器用户的密码在哈希之前加入额外的字符串(pepper)，这样可以变相的增强简单密码的强度，像下面这样:

```python
>>> bcrypt.hashpw("password*{abcd&", bcrypt.gensalt())
'$2b$12$2dVYv2o5vw6uMYe2IT9V9uWfIR2zdkpKDagNRZ8eFOpS4nyNHJuz.'
```
`*{abcd&`就是pepper，它必须保存在服务器中，主要针对的场景数据库暴露但是应用服务器安全，可以拖延字典攻击的时间，让用户及早更改密码。

PS. Coursera上面斯坦福大学的[密码学](https://www.coursera.org/learn/crypto/home)课程开始了，有兴趣的可以去学习下，第二周的课程就在介绍块加密.

### References
----
1. https://en.wikipedia.org/wiki/Cryptographic_hash_function
2. https://en.wikipedia.org/wiki/Rainbow_table
3. https://en.wikipedia.org/wiki/Brute-force_attack
4. https://en.wikipedia.org/wiki/Blowfish_(cipher)
5. https://en.wikipedia.org/wiki/Feistel_cipher
6. https://en.wikipedia.org/wiki/S-box
