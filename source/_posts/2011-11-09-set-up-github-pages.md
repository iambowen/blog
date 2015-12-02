---
layout: post
title: set up github page
tag: blog
categories: ["tools"]
---

# 创建一个[github](www.github.com)账户
# 设置好用户名，email和github的token
# 创建一个新的repository，命名规则为username.github.com, 比如我创建的就是iambowen.github.com
# 设置好remote的repos后在本地创建同样的repos，然后添加README和index.html文件,本地提交
#  @git push -u origin master@,大约10分钟之后访问username.github.com,就可以看到新添加的index页面，比如[我的](http://iambowen.github.com)
# 安装[jekyll模版](https://github.com/mojombo/jekyll/wiki/install), @gem install jekyll@
jekyll 是一个文本转换引擎(text transformation engine),
它可以将Texttile/Markdown的标记语言转换为html的layout.通常的site基本结构如下:

```
|-- _config.yml
|-- _includes
|-- _layouts
|   |-- default.html
|   `-- post.html
|-- _posts
|   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
|   `-- 2009-04-26-barcamp-boston-4-roundup.textile
|-- _site `-- index.html
```

`_config.yml`

保存配置的文件.

*_includes*
这里保存公用的patilal文件，在 _layouts 和 _posts 的文件中可以复用这些文件。

*_layout* 这里保存文章的模版文件

*_posts* 这里保存blog的文章,文件的格式通常使用YEAR-MONTH-DATE-title.MARKUP,然后github会自动的将这些文件转换.
*_site*
这里的文件可以由命令 @jekyll --pygments@生成之后在命令行启动 jekyll的服务器,@jekyll --server@.通过访问http://127.0.0.1/4000去预览post._site文件不需要push到remote repos上,把它加到.gitignore文件中就可以了.

写完一篇blog，预览后，就可以本地提交，push到远程主机上了。过几分钟，应该就能看到博文^_^了.
