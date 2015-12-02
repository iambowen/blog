---
layout: post
title: "Learning Shell: Navigation File Content"
description: "shell commands"
category: shell
tags: [shell]
---
Linux系统中浏览文件的命令很多，比如`less`,`more`, `head`,`tail`, `cat`等，其中`less`命令最为丰富，对于二进制文件和压缩文件，也有对应的命令去浏览。我整理了下，在组内做了一次分享，如下为主要内容。

##less

这个命令可以查看一个文件的内容，和编辑器的打开要读取整个文件的方式不一样，`less`只读取部分，因此，它占用的内存较少。如果是在产品环境下追踪问题，需要打开很大的文档，用这个命令要好过编辑器。

####移动
  - Ctrl + F(orward)/U(p)/B(ackward)/D(own)  或者 space 滚动
  - j/k   向上或者向下移动一行
  - g/G/q   跳到文件头/尾部， 退出浏览

####Search
  - ?         反向搜索
  - /         正向搜索
  - n/N       跳到下一处/前一处匹配行
  - &pattern  只显示匹配 pattern 的行

####有用的命令
  - :v         使用缺省的编辑器进入编辑模式，可以通过`EDITOR`环境变量修改
  - :F         类似`tail -f`的效果，等待文件即时的输出，但是tail命令是不能搜索的
  - ma         标记，使用a跳到标记处，``跳到上次浏览位置
  - !          执行shell命令
  - :s file    将管道中的内容保存在file中
  - :h         显示所有快捷方式及命令

####浏览多个文件
  - less file1 file2 file3
  - :n   查看下一个文件
  - :p   查看前一个文件
  - :e   打开一个文件

##cat
查看文件内容，每次输出全部内容。`tac`命令和`cat`命令作用相反。

####常用参数
  - -T 显示文件中的tab
  - -n 显示文件的行数
  - -b 显示行数，不包括空行

####重定向
  - 清空文件:  `cat /dev/null > empty_the_file.txt`
  - 合并文件:  `cat -n file1 file2 > file3`
  - 重定向:    `cat ~/.ssh/id_rsa.pub | ssh root@remote “cat >> ~/.ssh/authorized_keys”`


##head/tail
取文件前/尾部 10(n)行内容，用法简单直接。
  - head -n 10 file
  - tail -f -n 10 file
  - tail -n 10
  - tail -r file(reversely)

##strings
浏览二进制文件，`strings`命令可以抽取二进制文件中所有的字符串，在二进制文件没有提供很有效的信息的清空下，可以用这个命令获取到更多的信息，用法很简单。
  - `strings filename`

## `Z`命令
z系列命令特指z开头的一系列命令，包括:

  - zcat
  - zless/zmore
  - zgrep
  - zdiff(zcmp)

这些命令可以看做是变异版的cat/less等，其针对的对象是压缩文件，如`tar.gz`结尾的文件，在不解压的情况下，浏览/查找其内容，对比文件内容等。需要注意的是，这些命令只能操作单个文件压缩后的文件，如果是整个目录那就达不到效果了。
对于 bz2 结尾的压缩文件，也有对应的命令群，`bz*`，以此类推。


###本次session的ppt:

<script async class="speakerdeck-embed" data-id="b70bb090096e01326f7c2216803d3d2d" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
