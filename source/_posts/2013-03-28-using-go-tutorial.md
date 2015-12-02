---
layout: post
title: using go tutorial
tag: go tutorial
categories: ["go"]
---

Just trying to use Go for I find it looks like C, and the cost of writing a hello world is quite small, plus the tutorial is petty easy to fetch. Just list what I've done to get the tutorial and write the hello world example, which already posted thousands time in the internet.

*Install hg:*


  # brew install hg  (mac)

  #_ apt-get install mercurial (ubuntu)


*Download go:*

  # wget https://go.googlecode.com/files/go1.0.3.darwin-amd64.tar.gz

  #_ wget https://go.googlecode.com/files/go1.0.3.linux-amd64.tar.gz

*Install go:*

```bash
   *  tar -C /usr/local/ -zxf go1.0.3.darwin-amd64.tar.gz
```
*Export path:*

```bash
   * export PATH=$PATH:/usr/local/go/bin
```

*Get go tutorial:*

```bash
   * go get code.google.com/p/go-tour/gotour
```

*Run go tour:*

```bash
   *  gotour
```

After run the hello world program, you can leave this alone and declare that you can actually play with go now.........................
