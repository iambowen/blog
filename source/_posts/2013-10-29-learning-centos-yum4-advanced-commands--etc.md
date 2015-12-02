---
layout: post
title: "Learning Centos: yum(4) advanced commands & etc"
description: ""
category: "linux"
tags: ["linux"]
---



[前一篇](http://iambowen.github.io/2013/07/31/learning-yum3-basic-commands/ ) YUM基本命令，介绍了工作中比较常用的一些命令。除此之外，YUM还有些不太常用的但是还算比较有趣的命令，在此我们稍微探索发现下。

`yum shell `: yum内建的交互式shell

在这个shell中，我们可以进行各种操作，比如罗列所有repo，搜索/查看软件信息等等。eg:

```bash
[vagrant@localhost ~]$ sudo yum shell
Loaded plugins: fastestmirror
Setting up Yum Shell
> search vim-common
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * extras: mirrors.stuhome.net
 * updates: mirrors.btte.net
N/S Matched: vim-common ==============================================================================
vim-common.x86_64 : The common files needed by any version of the VIM editor
```

好处非常明显，就是免掉了每次去多输入”yum”。实际的测试发现，除了``` Install/Remove ```之外，大多数的命令都可以在这个shell下执行。其中的原因不太明白，也没有找到合理的解释。

`yum history `: 记录所有yum transaction的历史
yum 的transaction 记录都被保存在/var/lib/yum/history/ 下的sqlite db文件中，通过history命令，我们可以查看到软件安装的详细记录，执行者，具体操作，影响的package等。

```bash
     [vagrant@localhost ~]$ sudo yum history
Loaded plugins: fastestmirror
ID     | Login user               | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     7 |  <vagrant>               | 2013-10-26 06:30 | Install        |   58
     6 |  <vagrant>               | 2013-10-26 05:08 | Install        |    3
     5 |  <vagrant>               | 2013-09-25 08:25 | Install        |    3
     4 |  <vagrant>               | 2013-09-24 07:04 | Install        |    1
     3 |  <veewee>                | 2013-03-09 15:50 | Install        |   16  <
     2 |  <veewee>                | 2013-03-09 15:49 | I, U           |   40 >
     1 | System <unset>           | 2013-03-09 15:37 | Install        |  199
```

通过ID可以查找Transaction发生时候的相关信息。

```bash
[vagrant@localhost ~]$ sudo yum history info 3
Loaded plugins: fastestmirror
Transaction ID : 3
Begin time     : Sat Mar  9 15:50:00 2013
Begin rpmdb    : 239:6f5a8ecd22e6f0a663940801ef56d66b2ad40228
End time       :            15:50:07 2013 (7 seconds)
End rpmdb      : 255:92e8267085fc364911baebaf646b14856fb20498
User           :  <veewee>
Return-Code    : Success
Command Line   : -y install puppet facter
Transaction performed with:
    Installed     rpm-4.8.0-32.el6.x86_64                       @anaconda-CentOS-201303020151.x86_64/6.4
    Installed     yum-3.2.29-40.el6.centos.noarch               @anaconda-CentOS-201303020151.x86_64/6.4
    Installed     yum-plugin-fastestmirror-1.1.30-14.el6.noarch @anaconda-CentOS-201303020151.x86_64/6.4
Packages Altered:
    Dep-Install augeas-libs-0.9.0-4.el6.x86_64        @base
    Dep-Install compat-readline5-5.2-17.1.el6.x86_64  @base
    Dep-Install dmidecode-1:2.11-2.el6.x86_64         @base
    Install     facter-1:1.6.17-1.el6.x86_64          @puppetlabs
    Dep-Install hiera-1.1.2-1.el6.noarch              @puppetlabs
    ...
```

既然可以记录Transaction，那么势必也会提供undo和redo的功能:

```bash
   sudo yum history undo 7  #将Transaction 7中安装的所有软件都删除
   sudo yum history redo 8  #将Transaction 7中删除的软件重新安装
```

对于系统管理员来说，这应该算得上是个比较有用的命令。前面提到yum的交易记录数据库文件保存在 ` /var/lib/yum/history/ `下面，如果想将Transction记录到新的db文件中，可以用` yum history new `，它会清空` /var/lib/yum/history/ `目录，然后新建db文件。想打开某个history记录文件可以用`yum load-transaction file_name`。

`yum provides` :  通过软件安装后的可执行文件，反查安装它的package

```bash
vagrant@localhost ~]$ yum provides /usr/bin/vim
1:vim-enhanced-7.2.411-1.8.el6.x86_64 : A version of the VIM editor which includes recent enhancements
Repo        : installed
Matched from:
Other       : Provides-match: /usr/bin/vim

[vagrant@localhost ~]$ yum provides wget
wget-1.12-1.8.el6.x86_64 : A utility for retrieving files using the HTTP or FTP protocols
Repo        : installed
Matched from:
Other       : Provides-match: wget

[vagrant@localhost ~]$ yum provides /*/vi
1:vim-minimal-7.2.411-1.8.el6.x86_64 : A minimal version of the VIM editor
Repo        : installed
Matched from:
Other       : Provides-match: /bin/vi

```

这个命令还是略有用处的。

`yum downgrade`: 版本回滚

在持续部署出错时候，使用downgrade可以顺利进行版本的回滚。

好吧，熟悉了这些个命令和相关配置，我相信，从使用的角度讲，已经相当的足够了。下一篇将尝试搭建自己的yum repo服务器。
