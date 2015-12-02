---
layout: post
title: "Learning Yum(1): Config Files Explained"
description: ""
category: linux
tags: [yum, centos]
---


[Yum](http://en.wikipedia.org/wiki/Yellowdog_Updater,_Modified)是RedHat系列中(如Centos, Fedora)等软件包管理工具，全称为Yellow Dog Updater, Modified, 在RPM的基础上，它可以自动解析安装软件的依赖。最早接触Redhat的时候时的版本是Redhat9, 实在是不堪忍受各种坑爹的依赖，所以后来接触Debian/Ubuntu之后才有大喜过望的感觉。最近想多熟悉几个系统，Centos/Arch Linux都在之列(没有Fedora的原因它发布比较快，不是特别稳定)，在都使用相同内核的前提，这些发行版本的最大的不同也许就是包/软件管理器了，这就是为什么我要先开始熟悉yum了…….

从[这里](http://www.vagrantbox.es/)下载Centos的virtual box镜像,添加到[Vagrant](http://vagrantup.com/)中,你就可以拥有一个Centos的环境了，然后深度接触yum。

开始之前，得先知道 yum 相关的配置文件都在什么地方，

```bash
	man yum.conf
```
在帮助文件的底部我们就可以看到几乎所有的yum相关的配置文件:
```bash
FILES
   /etc/yum.conf
   /etc/yum.repos.d/
   /etc/yum/pluginconf.d/
   /etc/yum/protected.d
   /etc/yum/vars
   /etc/yum/version-groups.conf
```

yum 主配置文件:

```bash
	/etc/yum.conf
```
如下为yum.conf的示例文件:

```bash
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=1
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```
可以看到配置文件中有大量的选项，比如选择是否在本地保存cache，cache目录在那里，log文件的位置，是否enable yum的plugin等等，详细的配置项的解释可以查看man yum.conf.
yum repos 配置文件:

```bash
[vagrant@localhost ~]$ tree /etc/yum.repos.d/
/etc/yum.repos.d/
|-- CentOS-Base.repo
|-- CentOS-Debuginfo.repo
|-- CentOS-Media.repo
`-- CentOS-Vault.repo
```
配置文件中有两个section，一个是[main]，包含全局的yum配置, 一个是[repository]，包含repository的配置，定义软件仓库。 通常在主配置文件中定义main，repository定义通常放置在/etc/yum.repos.d/*.repo 文件中，扩展性非常好，如果你想添加新的软件源或者镜像，在/etc/yum.repos.d/目录下新创建一个.repo文件就好了。
yum 依赖libcurl, 所以下载的时候，是单线程的，在不使用插件的情况下，寻找一个合适的软件源是很重要的。 从[列表](http://mirror-status.centos.org/)中找到中国的镜像，还是很多的，教育网公网服务器一半一半的样子,最奇怪的是台湾的镜像，竟然显示是台湾省，呵呵。我选择的是[163的镜像](http://mirrors.163.com/)，步骤如下:

```bash
sudo cp CentOS-Base.repo Centos-163.repo
sudo sed -i 's/^mirrorlist/\#mirrorlist/g' Centos-163.repo
sudo sed -i 's/^\#baseurl/baseurl/g' Centos-163.repo
sudo sed -i 's/mirror.centos.org/mirrors.163.com/g' Centos-163.repo
```
之后运行yum update metadata，就可以更新metadata/index文件了。
.repo文件中配置细节目录区别:

```bash
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
               ftp://path/to/repo
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
enabled=1
```
其中base为repo的名字，baseurl支持多个url、协议，指向repo的地址。gpgcheck选项表示要不要对下载的package做校验。enabled选项表示默认情况下是否使用该repo。mirrorlist为后备的repo镜像url列表。相关的配置选项还很多，比如time_out, http_caching等，可以man yum.conf查看。

yum 的插件配置文件:

```bash
[vagrant@localhost ~]$ tree /etc/yum/pluginconf.d/
/etc/yum/pluginconf.d/
|-- aliases.conf
`-- fastestmirror.conf
```
拿fastestmirror.conf作为例子看看:

```bash
[main]
enabled=1
verbose=0
always_print_best_host = true
socket_timeout=3
hostfilepath=timedhosts.txt
maxhostfileage=10
maxthreads=15
```
其中enable选项可以打开或者关闭yum的插件。

/etc/yum/vars/*，自定义yum中可以引用的变量:
在yum.conf或者.repo中可以看到一些yum build-in变量，这些变量包括:

```bash
$basearch
$releasever
$arch
$YUM0-9
```
$basearch读取的是yum.conf中distroverpkg的值，$arch读取的是系统的CPU的architecture, $releasever是发行版的版本号，比如Centos-6。$YUM0-9读取shell中的变量，有则取之，无则为空。如果想从系统的环境变量中fetch一些值，可以在bash中设置$YUM0-9。自定义yum变量还有一种方法是在/etc/yum/vars/下新增变量文件，比如:

```bash
echo "It's not Ubuntu" > /etc/yum/vars/declare
```
在yum.conf或者.repo文件中就可以调用$declare了，

/etc/yum/version-groups.conf: 定义软件组， 用yum version查看

```bash
[yum]
#  These are the top level things to do with yum, we don't list Eg. libselinux
# even though that's require by rpm(-libs).
run_with_packages = true
pkglist = glibc, sqlite, libcurl, nss,
          yum-metadata-parser,
          rpm, rpm-libs, rpm-python,
          python,
          python-iniparse, python-urlgrabber, python-pycurl
```

/etc/yum/protected.d: 保护某些软件不被卸载，default只有yum，可以添加新的conf文件，比如:

```bash
[vagrant@localhost ~]$echo "vim-common" > /etc/yum/protected.d/vim.conf
```
此时如果remove vim-common这个package，就会得到失败的信息:

```bash
[vagrant@localhost ~]$ sudo yum remove vim-common
Setting up Remove Process
Resolving Dependencies
--> Running transaction check
---> Package vim-common.x86_64 2:7.2.411-1.8.el6 will be erased
--> Processing Dependency: vim-common = 2:7.2.411-1.8.el6 for package: 2:vim-enhanced-7.2.411-1.8.el6.x86_64
--> Running transaction check
---> Package vim-enhanced.x86_64 2:7.2.411-1.8.el6 will be erased
--> Finished Dependency Resolution
Error: Trying to remove "vim-common", which is protected
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

关于yum config文件大概就是这些，我们下一篇介绍yum的utils以及一些有用的plugins。
