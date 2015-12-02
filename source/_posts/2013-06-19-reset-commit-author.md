---
layout: post
title: "reset commit author"
description: ""
category: git
tags: ["git"]
---

在如下两种情况下，你需要重新配置git commit 的author。

* 在一个新的repo上面工作;
* 用别人的机器(或者pair machine上)提交代码，而关于你的author信息并未添加在git配置中;

假设你global git config ~/.gitconfg 中配置信息如下:

```
[user]
     name = example
     email = example@example.com
```
别人的机器(或者pair机器)的repo中git 配置信息如下:

```
[user]
     name = pair
     email = pair@pair.com
```
而你期望的提交信息为:

```
name = serious
email = serious@serious.com
```
在第一种情况下，当你修改完代码提交后，log中对应的author信息为:

```
Author: example <example@example.com>
```
在第二种情况下，当你修改完代码提交后，log中对应的author信息为:

```
Author: pair <pair@pair.com>
```
两种情况下，想要修改提交中的author信息，都可以通过:

```
git commit --amend --author=“serious <serious@serious.com>”
```
来重新设置。

第一种情况，如果你需要将来长期在这个repo上工作的话。首先，修改repo下的.git/config文件，添加

```
[user]
     name = serious
     email = serious@serious.com
```
然后重新设置提交的author信息:

```
git commit --amend -C HEAD --reset-author
```

Done
