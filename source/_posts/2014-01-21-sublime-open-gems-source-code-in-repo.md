---
layout: post
title: "Sublime open gems source code in repo"
description: ""
category: 
tags: [ruby]
---


Jetbrains的产品很赞，不过就是太耗资源了，比如Rubymine。有时候会用轻量点的编辑器Sublime去打开/编辑ruby工程代码，一个麻烦就产生了，sublime无法关联到工程内所用到的rubygems的源码。

google搜索了下，有两种解决的办法，一种是通过修改EDITOR环境变量.

```bash
export EDITOR='subl' >> ~/.bash_profile && source ~/.bash_profile

bundle open rubygem_name
```

相信和很多人一样，我们都已经把 EDITOR 设置为 `vim` 或者 `emacs`。那么可以采用第二种方案.

```bash
 ~> bundle show rake # 返回rubygem的完整路径
 /Users/iambowen/.rvm/gems/ree-1.8.7-2012.02/gems/rake-0.9.6
 ~> bundle show rake | xargs subl   # xargs do the trick
```

这次应该解决了问题，不过每次输入这么多命令不是很方便，可以写一个函数的函数来解决这个问题:

```bash
function sublg ()
{
  bundle show $1 | xargs subl
}
```
把这个函数加入`~/.bash_profile`后，就可以直接使用`sublg gem_name`来打开gem的源码了。
