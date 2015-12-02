---
layout: post
title: "在Skype中使用Hubot"
description: ""
category:
tags: ["hubot"]
---



[Hubot](http://www.36kr.com/p/55962.html) 是一个github开源的[机器人项目](http://hubot.github.com/)，可以集成到众多的聊天工具中，如IRC, Hipchat, Skype，QQ等。你可以用它来搞笑，也可以用它来进行自动化的管理，比如自动化部署等。这几天花了点时间，在OSX和Centos上分别尝试搭了下Hubot，详细过程如下:

安装Hubot, 有两种方式，懒人的方式就用现成的[Vagarant Script](https://github.com/RafaelGorski/VagrantChefHubot), 有几点要注意的是:

1. OSX必须得安装[X11](http://xquartz.macosforge.org/downloads/SL/XQuartz-2.7.5.dmg)支持
2. Oracle OCI 不是必须的

在Centos 6.4 上安装Hubot。

环境准备, 包括桌面支持，开发工具, skype等等.

```bash
yum install http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
yum groupinstall -y "Desktop"
yum groupinstall -y "Development Tools”
yum install qtwebkit.i686 webkitgtk.i686 —y
yum install tigervnc-server -y   #for vnc connect
yum install wget telnet -y
yum install sqlite-devel -y
yum install nodejs npm -y
yum install redis -y     
yum install python-setuptools python-setuptools-devel -y
yum update
```

安装Skype:

```bash
wget http://download.skype.com/linux/skype-4.2.0.13.tar.bz2
tar skype-4.2.0.13.tar.bz2 -C /opt/
ln -s skype-4.2.0.13 skype
ln -s /opt/skype/skype /usr/bin/skype
```

安装Skype4py: Skype api 的python wrapper

```bash
easy_install pip
pip install Skype4Py
```

安装coffee-script hubot:

```bash
npm install -g coffee-script hubot
```
配置服务:

```bash
echo -e "VNCSERVERS="1:vnc"\n VNCSERVERARGS[1]="-geometry 1024x768" >> /etc/sysconfig/vncservers
useradd vnc
su - vnc
passwd

service iptables stop
service vncserver start

chkconfig redis on    # auto start
chkconfig vncserver on   #auto start
service redis start
```

centos默认iptables的服务是打开的，邪恶一点的方法是直接关掉iptables服务，上进的方法就是在iptables里面添加新的规则，还是邪恶吧……，因为hubot-skype也可能存在被iptables block的情况。
VNC到虚拟机上，启动Skype并登陆。

```bash
skype &
```

安装配置hubot-skype:

```bash
mkdir hubot && cd hubot
hubot -c .
```

自动生成如下目录结构:

```bash
[root@localhost hubot]# tree -L 1
.
|-- bin
|-- external-scripts.json
|-- hubot-scripts.json
|-- node_modules
|-- package.json
|-- Procfile
|-- README.md
`-- scripts
```

`package.json` 里面放的hubot运行依赖的插件，`hubot-scripts.json` 里面写的是hubot的脚本。修改 `package.json`:

```javascript
  "dependencies": {
    "hubot":         ">= 2.6.0 < 3.0.0",
    "hubot-scripts": ">= 2.5.0 < 3.0.0",
    "hubot-skype": "0.0.2"
  }
npm install
```

修改`hubot-scripts.json`, 添加`karma.coffee`:

`["redis-brain.coffee", "shipit.coffee", "karma.coffee”]`

运行:

```bash
./bin/hubot  -a skype &
```
Skype会弹出是否允许第三方的应用控制，勾选就可以了。

把hubot添加到Skype对话中，就可以用命令来调戏hubot了, 下面是一些例子:

```bash
hubot image me keyword  #google 搜索keyword相关的图片，随机挑选一张返回图片链接
hubot animate me keyword  #google 搜索keyword相关的gif图片，随机挑选一张返回图片链接
hubot youtube me keyword  #youtube 搜索keyword相关的视频，随机挑选一个返回视频链接
rebot mustache me keyword #给google搜索返回的图片上加个胡子。。。。
```
添加了`karma.coffee`脚本-人品计算器的话，可以用`user++/user—` 来增加和减少讨论组内成员的人品值。
鉴于youtube国内无法访问，所以正准备扩展下脚本，提供 `hubot youku me `的命令。另外，如果把这个做成微信服务号部署到heroku上拿来玩应该也很有趣。

最后，写了一个[脚本](https://gist.github.com/iambowen/8642990)，配置过程中绝大多数操作都放到里面了, 不能保证100%自动化。

####Note:

1. 有个rails的工程[hubot-control](https://github.com/iambowen/hubot-control)可以更加方便的进行hubot的管理， 我做了些许修改;
2. 一定要VNC到虚拟机上去启动hubot-skype，不然会报这个错误:

```bash
  ......
  File "/usr/lib/python2.6/site-packages/Skype4Py/api/posix_x11.py", line 254, in __init__
    raise SkypeAPIError('Could not open XDisplay')
Skype4Py.errors.SkypeAPIError: Could not open XDisplay
```
原因是X11的环境变量 DISPLAY没有设置成0，非常之烦人，下次会考虑使用dbus去运行，虽然不知道怎么弄……
