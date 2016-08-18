---
layout: post
title: "how to write useful git commit message"
date: 2016-08-17 11:20:33 +0800
comments: true
categories: ["git"]
---
相信大家在自己项目的历史提交里面看到类似的提交记录
![](http://imgs.xkcd.com/comics/git_commit.png)
我还见过更加糟糕的，类似这样
```
53ee0c7 fix build again
7a63a11 fix build
```
这样的提交信息的问题在于不表意，没有简要的说明修改的内容，为什么要这样的修改，别人只能去查看具体的代码改动才能知道发生了什么，但是可能无法知道为什么这样修改。当然，这样的提交我自己也写过，原因包括
1. 语法、拼写错误，羞于示人
2. 解释原因得写很长，懒的敲键盘
3. 无法解释为什么这样的修改就能work
这其实算是一种比较不负责任的行为，估计别人看到会比较崩溃，幸运的是还没有领导看到，所以至今没有被开除。举个例子，假设某个提交引起了产品环境的错误，别人需要迅速定位是哪个提交引起的问题，但是如果提交都是类似`Perfectly complete a new story`，而且每次代码修改的量都比较庞大，那就得花很多时间才能定位。相反如果提交信息很清晰，`BAU-1008 add xxx form in xxx page. :pear: Kevin`。你用`git log --oneline --after "Aug 10 2016" `可以迅速看到对应的提交，进一步的可以查看修改内容再查找具体的问题。

今天早上客户跟我们一起做了一个关于如何有效的提交`git commit`信息。他提到了`git commit message`的7个[规则](http://chris.beams.io/posts/git-commit/)。他认为从项目维护性的角度考虑，应当注意提交的信息以及规范。一个项目的提交信息首先得从下面三个方面达成一致:

1. 格式。消息体的格式，如Markdown，语法应该是什么样子，大写的规则等。
2. 内容。提交的信息中应当包含什么，不应当包含什么。
3. 元数据。问题跟踪的ID（Jira，Leankit等）要不要引用，PR的sha code要不要引用。

具体的规则有下面7点：

1. 用空行将内容和主题分开
2. 提交的主题限制在50个字符
3. 主题首字母大写
4. 主题结尾不要使用句号
5. 主题需要使用祈使/肯定语气
6. 内容每72个字符换行
7. 在内容中解释清楚修改的原因及方式

第一条，如果内容和主题没有分开，`git log --oneline`主题和下面的内容会一起显示。
第二条，主题超过50个字符时超过的部分在github上显示为`...`，提交PR的时候超过的部分会被折断到comments中，很烦人。用vim去编辑提交信息的时候，如果看到主题的字的颜色变化了，就说明超过了50个字符。
第三、四条不评价，感觉更多是从美观和规范上统一的。
第五条，感觉这样可以少写一些字，而且和git 缺省的提交信息，如revert的提交信息一致。
```
Revert "Add the thing with the stuff"

This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```
第六条，因为git不会帮你wrap文字，所以得手动的来做这个事情，这里可以借助一些编辑器，如VI的帮助。
第七条，个人觉得这个才是最重要的，解释清楚修改的原因以及方式，引用别人文章里面的一个例子:
```
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.

   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).

   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().

   fail(), clear(n) and exceptions() are just never called. Delete
   them.
 ```
业务相关的代码修改，可以将story ID加在最前面，方便issue track。

 为了让大家统一提交的格式，可以新建一个提交的template，配置git[使用](https://robots.thoughtbot.com/better-commit-messages-with-a-gitmessage-template)。 过程如下:

 1. 在`~/.gitconfig`中加入下面的内容：
 ```
 [commit]
  template = ~/.gitmessage
```
2. 新建`~/.gitmessage`这个template文件并且填入自定义模板：
```
Brief here:

Reason to change:
*

Way to change:

*
```
配置完成后在项目中做修改，`git ci -a`就可以在模板的基础上修改了。

举一个项目中的一个例子，里面包含了业务需求的github issue的链接：
```
commit 2196e866261dee6d7c17f266cc15987f
Author: Alex Jin <alex.jin@example.com>
Date:   Wed Aug 17 13:24:41 2016 +0800
    Make trend bigger and modify link color. :pear: Luke

    * Reason to change:
      see [lob/project#208]
```

如果是用Pull Request方式工作的话，麻烦的地方在于修改的原因可能还得在comments里面再写一遍。解决的办法是创建issue/PR的template，参考[这里](https://github.com/blog/2111-issue-and-pull-request-templates)。

什么，你问我为什么还没有被开除么？
因为领导不看提交 :)
