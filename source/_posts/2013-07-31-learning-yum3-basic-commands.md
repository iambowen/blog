---
layout: post
title: "Learning Yum(3): Basic Commands"
description: ""
category: "linux"
tags: ["linux"]
---

对于包管理工具来说，安装/删除/查看包信息/升级软件包/搜索查找/清除metadata, 都是必须提供的功能。yum的基本命令操作同样也可以概括为:

```
install/(remove|erase|downgrade)/(list|info)/(update|upgrade)/search/clean
```

**安装package:**  
yum安装package有两种方式，一种是安装单个/多个的package，一种是安装软件组。

```bash
yum install package_names
yum install  -y(--ausumeyes) package_names   
yum install --enablerepo=drivers nvidia-driver
```
install的时候可以指定或者不指定具体的版本号，正常安装的时候会提示是否确认安装。为了偷懒或者在自动化部署的时候，想在跳过提示，可以加上个``` -y ```的参数，就可以了。有些repo可能默认被disable了，只想在单次的yum install的时候enable，可以加上 ``` --enablerepo=<repo name> ```.

**组安装:**  
想知道当前repo都提供了哪些package group，可以用`yum -v grouplist `，可以知道组名及组id。

```bash
[vagrant@localhost ~]$ yum -v grouplist kde\*
Config time: 0.009
Yum Version: 3.2.29
Setting up Group Process
rpmdb time: 0.001
group time: 0.159
Available Groups:
   KDE Desktop (kde-desktop)   # KDE Desktop 组的id 就是(kde-desktop)
Done
```
组安装的有几种不同形式，但是结果都是相同的，如下的几个命令都会安装KDE Desktop的groupyum groupinstall "KDE Desktop"

```bash
yum groupinstall kde-desktop
yum install @kde-desktop
```

**删除package:**

```bash
yum remove package_name
yum remove glibc*
yum erase package_name/glob expression
yum groupremove  #删除软件组
```
开始我以为remove和erase是有区别的，但是后来在fedora社区论坛上发现其实它们没什么区别，很汗啊，上次分享的时候我还信誓旦旦的说了来着，都是被apt搞混了。

**查看package信息:**

```bash
yum info package_name   等同于rpm -q --info package_name
vagrant@localhost ~]$ yum info vim-common
Installed Packages
Name        : vim-common
Arch        : x86_64
Epoch       : 2
Version     : 7.2.411
Release     : 1.8.el6
Size        : 17 M
Repo        : installed
From repo   : base
Summary     : The common files needed by any version of the VIM editor
URL         : http://www.vim.org/
License     : Vim and GPLv2+ and BSD and LGPLv2+ and Open Publication
Description : VIM (VIsual editor iMproved) is an updated and improved version of the
            : vi editor.  Vi was the first real screen-based editor for UNIX, and is........
```
可以看到package相关的所有信息，

**罗列packages信息:**  

```
yum list [available|installed|extras|updates|obsoletes|all|recent] [pkgspec]
```

```bash
yum list available 列出当前enable的repo中所有可用package
yum list installed 列出当前系统中已经安装的package
yum list extras 列出已安装的package中不在enable的repo中的package
yum list updates  列出当前enable的repo中可以提供更新的package
yum list obsoletes 列出enable的repo中或者已安装的package中被淘汰的package
yum list all 列出所有enable的repo中的package
yum list recent 列出所有enable的repo中最近一周添加的package
yum list installed "krb?-*" 以上所有的list命令都支持通配符，罗列相关的package信息
yum list package spec  为特定的package列出相关信息，比如列出glibc的信息
[vagrant@localhost yum.repos.d]$ yum list glibc
Installed Packages
glibc.x86_64                                                       2.12-1.107.el6                                                            @anaconda-CentOS-201303020151.x86_64/6.4
Available Packages
glibc.i686                                                         2.12-1.107.el6_4.2                                                        updates
glibc.x86_64                                                       2.12-1.107.el6_4.2                                                        updates

yum list  \*.i686  #glob expression
```

**罗列组信息:**

```bash
yum grouplist
yum -v grouplist  "KDE*"
```

**查看repo信息:**

```bash
yum repolist
```

**更新package:**

```bash
yum check-update  #检查有更新的软件，返回的值分别为可以更新的package所在repo已经最新的版本号
yum check-update glibc*   #检查单一的package，支持通配符
[vagrant@localhost yum.repos.d]$ yum check-update
glibc.x86_64                                                                            2.12-1.107.el6_4.2                                                                    updates

yum update   #升级所有已安装package有更新的package
yum update glibc #升级某一个package
yum update glibc*  #升级某一票package

yum upgrade  #和update类似，但是会考虑淘汰的package
```

**搜索软件包:**  
` yum search ` yum 提供了软件包搜索的功能，可以通过关键字(package描述|package名)搜索到你想要的软件包。

```bash
vagrant@localhost yum.repos.d]$ yum search vim
=================================================================================== Matched: vim ====================================================================================
vim-X11.x86_64 : The VIM version of the vi editor for the X Window System
vim-common.x86_64 : The common files needed by any version of the VIM editor
vim-enhanced.x86_64 : A version of the VIM editor which includes recent enhancements
vim-minimal.x86_64 : A minimal version of the VIM editor

[vagrant@localhost yum.repos.d]$ yum search "vi editor" #只需要描述软件提供的功能，便可以查找出提供该功能的软件包
============================================================================== N/S Matched: vi editor ===============================================================================
vim-X11.x86_64 : The VIM version of the vi editor for the X Window System
```

**清理本地cached metadata 和package:**  
减少占用的磁盘空间，或者处理metadata出错的情况:

```bash
yum clean  #默认清理cached package和metadata，cache的目录是/var/cache/yum/
yum clean packages #只清理cached package
yum clean metadata #只清理metadata
yum clean dbcache   #清理yum的sqllite数据库文件
yum clean all     #清理
```

常用的yum 命令大概就有这么些，我们可以发现:  
1. yum基本的命令使用模式比较接近, 而且都是正常人的使用逻辑;  
2. yum的命令一般都会支持通配符的操作，用法比较统一；  
下一次我们将介绍一些平常不太用yum但是还有点意思的命令。
