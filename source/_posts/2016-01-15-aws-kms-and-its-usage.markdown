---
layout: post
title: "AWS KMS and its usage"
date: 2016-01-15 09:33:30 +0800
comments: true
categories: ["AWS", "KMS"]
---

##什么是KMS
[KMS](https://aws.amazon.com/kms/)是AWS提供的中心化的key管理服务。你可以用它来创建和加密
的key，同时，AWS会使用Hardware Security Modules (HSMs)来包括你的master key。它还被集成
到了其他的AWS服务中，如CloudTrail会记录所有key的使用以方便审计。

## KMS的优点
基本上来自于[文档](https://aws.amazon.com/kms/)，
