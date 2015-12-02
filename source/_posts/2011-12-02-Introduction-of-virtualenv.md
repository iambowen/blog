---
layout: post
title: Introduction of virtualenv
tag: python
categories: ["python"]
---

什么是 *virtualenv*:
virtualenv 是一款用来创建隔离的Python开发环境的工具，有点类似ruby下的rvm.通过virtualenv，可以为不同的python项目定制所需要的环境.

OSX下安装的过程:
本来想自己写，但是这位阿三哥写的太好了，就节省下体力……: "猛击这里":http://blog.praveengollakota.com/47430655
OSX系统自带的python版本是2.6，而我们开发使用的版本是2.7，直接升级很容易，但是其中有些配置会稍微麻烦一些。
主要的步骤:

# 安装 *Python 2.7*
#_ 下载 *easy_install*,安装对应python2.7的 easy_install：
@sh setuptools-0.6c11-py2.7.egg@
#_ 使用easy_install 安装 pip, pip是一个python下的包管理工具:  
@easy_install pip@
#_ 使用pip 去安装 *virtualenv* 以及 *virtualenvwrapper* :

``` bash
pip install virtualenv
pip install virtualenvwrapper
```

#_ 配置 *.bashrc/.bash_profile* 文件使得 *virtualenvwrapper* 的命令可以在bash下运行:

``` bash
mkdir ~/.virtualenv
export WORKON_HOME=$HOME/.virtualenvs
source /Library/Frameworks/Python.framework/Versions/2.7/bin/virtualenvwrapper.sh
```

之后 `source ~/.bash_profile` ，就可以在命令行中使用 `vritualenvwrapper` 中的命令了.

*如何使用virtualenv:*

# 创建一个虚拟环境:

`mkvirtualenv  test`

#_ 查看当前的虚拟环境的列表:

`workon`

#_ 使用某一个虚拟环境:

`workon test`

之后就回看到shell的提示符前面出现了环境的名称，这里就可以大胆的去做各种环境配置了，用pip去安装所有你需要的python包。
想退出的话，可以直接 `command + d` 或者用 `deactivate` 命令即可.
