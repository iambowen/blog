---
layout: post
title: Using watch to excecute commands in cycle
tag: linux
categories: ["linux"]
---

项目的集成测试环境以及UAT环境都部署在亚马逊的虚拟云节点上面，早上9点之前，一个cron job会自动将这些节点启动。有时候会遇到当前可用的节点不足而导致节点创建失败, 需要不停的尝试启动节点的命令，看看什么时候运气垂青，碰到有其他节点被destroy时，可以抢到这样的“token”去启动我们的节点。
这个effort显然过大了，作为懒人是不能接受的，同事介绍了watch这个工具，它可以周期性的执行一个命令，同时将结果输出，轻松的解决了问题。

*安装:*
大部分的linux发行版应该都包括了这个应用，MAC下面需要自己安装，用brew也很方便。

*+brew install watch+*

*用法介绍:*

```bash
Usage: watch [-dhntv] [--differences[=cumulative]] [--help] [--interval=<n>] [--no-title] [--version] <command>
```

+-n, --interval = <seconds>+ : 设定相隔多长时间重新运行下监控的命令并输出结果
+-d, --differences[=cumulative]+ : 高亮显示监控的命令输出相比上次输出变化的区域
+-v, --version+ : 显示watch的版本号
注: mac 下的watch版本略低，0.3.0的版本提供了更多的功能，比如彩色输出，退出响铃之类的
*使用场景:*

watch 的帮助文档中，给出了一些使用的场景， 比如:


   # 监控当前目录中某些用户的文件变化:   @watch -d ‘ls -l | grep root’@

   #_ 偷窥其他用户的窗口行为:  @watch tty7@ (假设tty7有用户登陆)


能想到的还有一个,当时还想自己实现这样的一个脚本，后来看到watch，就放弃了……:


   #_ 正在编辑haml文件，同时想在浏览器上面看到更改后的html文件: @watch -d 'haml input.haml input.html'@
