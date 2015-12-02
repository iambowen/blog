---
layout: post
title: "Deploy Rails 4 app to Heroku"
description: ""
category: 
tags: [deploy, heroku]
---



今天参加Railsgirls的活动，被抓壮丁去做了Rails应用在Heroku部署的演示，本来以为很简单，照着[教程](http://rgcn.github.io/heroku/) 走一遍就好了，但是没想到其中还是有些坑的，做了个简单的ppt，解释下里面的坑。

<script async class="speakerdeck-embed" data-id="29a27f305cc901317bda563746ea3d55" data-ratio="1.2994923857868" src="//speakerdeck.com/assets/embed.js"></script> 


####为什么要安装rails_12factor

>
By default Rails 4 will return a 404 if an asset is not handled via an external proxy such as Nginx. While this default behavior may help you debug your Nginx configuration, it makes a default Rails app with assets unusable on a twelve-factor platform. ….  The rails_serve_static_assets gem enables your Rails server to deliver your assets directly, instead of returning a 404.  
>

产品环境里面，rails应用可能是在类似Nginx的服务器后面，静态的assets，比如对/public/assets/application.css
的请求是由Nginx直接处理的，但是Heroku并没有使用类似Nginx这样的proxy，而是在routing layer
做的loadbalancing。所以当你请求静态的assets的时候，会得到404的错误。rails_12factor包含了rails_serve_static_assets这个gem，
可以把对assets的控制权交给Rails应用本身。这样就不会出现拿不到更新的css的情况了。
