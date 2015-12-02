---
layout: post
title: "flush dns cache in local machine"
description: ""
category:
tags: [network]
---


Under linux:

```bash
sudo /etc/init.d/nsdc restart
```

Under windows:

```bash
ipconfig /flushdns
```

Under Mac:

```bash
dscacheutil -flushcache
```
