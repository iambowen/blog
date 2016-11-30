---
layout: post
title: "increase mac docker client vm size"
date: 2016-11-29 21:17:15 +0800
comments: true
categories: ["docker", "resize"]
---
最近在做数据流水线的spike以及直播的一些实验准备，需要在本地运行很多的容器，没过多久就发现有些容器启动失败，提示磁盘不足。
我对这个提示感到困惑，于是，`docker exec `到一个已经运行的容器中，执行`df -h`,结果如下:

```
Filesystem                Size      Used Available Use% Mounted on
none                     19.2G      19.2G     0G   100% /
tmpfs                     3.9G         0      3.9G   0% /dev
tmpfs                     3.9G         0      3.9G   0% /sys/fs/cgroup
/dev/vda2                19.2G      19.2G     41.3G   100% /etc/resolv.conf
/dev/vda2                19.2G      19.2G     41.3G   100% /etc/hostname
/dev/vda2                19.2G      19.2G     41.3G   100% /etc/hosts
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     3.9G         0      3.9G   0% /proc/kcore
tmpfs                     3.9G         0      3.9G   0% /proc/timer_list
tmpfs                     3.9G         0      3.9G   0% /proc/sched_debug
```
真的没有空间了……，我开始不能理解为什么会出现这样的情况。首先想到的是docker mac client的设置，可能会有虚拟机磁盘的大小的配置项……，但是打开一看竟然没有。
其次想到的是[hyperkit](https://github.com/docker/hyperkit)，Docker mac 客户端依赖的hypervisor，结果也没有这样的选项。
搜索了下，查到了这篇[帖子](https://forums.docker.com/t/consistently-out-of-disk-space-in-docker-beta/9438)。Docker在Mac下的vm 磁盘文件的路径是`~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2`, 我查看了下这个文件，大小约20G。那现在的问题怎么把这个磁盘文件增大。
考虑了半天，查询了资料，问了同事也没有找到可行的解决方案。文章里面介绍的resize之后重新分区的方式很容易破坏原有的磁盘。可选的方式就是重新用`qemu-image`创建一个更大的虚拟分区文件，然后删除旧的`Docker.qcow2`文件，。操作如下:

1. `mv Docker.qcow2 Docker.qcow2.bak`
2. `brew install qemu`
3.  创建更大空间虚拟磁盘
```
~> qemu-img create -f qcow2 Docker.qcow2 50G
Formatting 'Docker.qcow2', fmt=qcow2 size=53687091200 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16
```
4. 重启docker

不足的地方就是原有镜像、volume、容器都不见了，还好没什么特别重要的数据.从帖子里面看到每个人的默认Docker vm磁盘大小是不一样的，这可能是根据可用磁盘空间乘以一个比例去设置的，这个时候突然想起应该在公司发的MBP本磁盘大小的邮件，我应该在下面顶一记来着…… 512G++。此外，新的Docker版本如果能支持支持vm disk的auto expand就好了。
