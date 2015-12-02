---
layout: post
title: "running containers in Docker"
description: "Docker containers"
category: docker
tags: [docker]
---

既然有了boot2docker,不妨下几个容器来玩玩。先看下本地的docker服务的信息。

```bash
docker@boot2docker:~$ docker info
Containers: 0
Images: 0
Storage Driver: aufs
Root Dir: /mnt/sda1/var/lib/docker/aufs
Dirs: 0
Execution Driver: native-0.2
Kernel Version: 3.16.7-tinycore64
Operating System: Boot2Docker 1.4.1 (TCL 5.4); master : 86f7ec8 - Tue Dec 16 23:11:29 UTC 2014
CPUs: 4
Total Memory: 1.961 GiB
Name: boot2docker
ID: L5JR:QYWN:KEDD:FQ4O:66IY:FUE6:OXTA:SWAE:UV2R:LFCF:7H6H:PTS5
Debug mode (server): true
Debug mode (client): false
Fds: 10
Goroutines: 11
EventsListeners: 0
Init Path: /usr/local/bin/docker
Docker Root Dir: /mnt/sda1/var/lib/docker

docker@boot2docker:~$ ps aux | grep [/]usr/local/bin/docker
690 root     /usr/local/bin/docker -d -D -g /var/lib/docker -H unix:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/var/lib/boot2docker/tls/ca.pem --tlscert=/var/lib/boot2docker/tls/server.pem --tlskey=/var/lib/boot2docker/tls/serverkey.pem
```
从docker的后台进程可以看到，后台服务监听2376端口，同时支持tls加密协议。

Docker的官方镜像的入口在[Docker Hub Registry](https://registry.hub.docker.com/)，镜像文件放在了亚马逊AWS上，这对于国内的用户就比较悲剧了，因为要么是访问不到镜像文件，要么就是下载太慢。万幸的是，国内现在已经有了docker registry 的mirror - [docker.cn](https://docker.cn/)。
所以在pull或者运行容器的时候，需要在镜像前加上registry的ip或者主机名。我觉得更好的方式是可以将registry写在配置文件中，这样可以避免这个麻烦，不过目前还没有看到这样的解决方案。
So

``` bash
docker@boot2docker:~$ docker run -i -t docker.cn/docker/ubuntu /bin/bash
Unable to find image 'docker.cn/docker/ubuntu:latest' locally
Pulling repository docker.cn/docker/ubuntu
8eaa4ff06b53: Download complete
511136ea3c5a: Download complete
3b363fd9d7da: Download complete
607c5d1cca71: Download complete
f62feddc05dc: Download complete
Status: Downloaded newer image for docker.cn/docker/ubuntu:latest
root@0b8874d6f2c8:/#
```
这条命令的意思是保持标准输入打开状态`-i`，并且Docker要为容器分配一个伪终端(tty)`-t`。Docker daemon首先在本地寻找名为`docker.cn/docker/ubuntu`的镜像，找不到的话，在去docker.cn下载。
从镜像下载的方式我们大概可以判断整个镜像是分层的，像git一样有版本管理，这个就是`aufs`文件系统的作用。镜像下载完成，容器启动后执行了bash命令，可以认为是来到了容器运行的操作系统中。
之后就可以像使用一个Ubuntu系统一样去使用这个容器了。


```bash
root@0b8874d6f2c8:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
valid_lft forever preferred_lft forever
6: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
inet 172.17.0.2/16 scope global eth0
valid_lft forever preferred_lft forever
inet6 fe80::42:acff:fe11:2/64 scope link
valid_lft forever preferred_lft forever
```
可以看到容器有自己的eth0网络设备以及由Docker分配的一个ip地址，和host machine没太大区别。


```bash
root@0b8874d6f2c8:/# vim
bash: vim: command not found
root@0b8874d6f2c8:/# apt-get install -y vim

```
没有vim，赶快装一个。

随之退出控制台，容器也停止运行。

``` bash
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
docker@boot2docker:~$ docker ps -l
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS                          PORTS               NAMES
0b8874d6f2c8        docker.cn/docker/ubuntu:14.04   "/bin/bash"         15 hours ago        Exited (0) About a minute ago                       naughty_darwin
```
使用`docker ps`命令可以查看运行时的容器，对于停止的容器，可以通过 `docker ps -l` 或者 `docker ps -a`可以看到所有的容器。
容器启动时通过`--name`选项可以指定容器的名字，缺省会随机给出一个名字，很诡异的一个设计，不能理解。
有名字的好处就是就是容易记忆或者查看，否则如果只能按照containerid那串hash code使用，想来就有些头大。

重新启动停止的容器

```bash
docker@boot2docker:~$ docker start 0b8874d6f2c8
0b8874d6f2c8
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS              PORTS               NAMES
0b8874d6f2c8        docker.cn/docker/ubuntu:14.04   "/bin/bash"         15 hours ago        Up 3 seconds                            naughty_darwin
```
通过`docker attach`命令可以重新连接到容器上。

```bash
docker@boot2docker:~$ docker attach 0b8874d6f2c8
root@0b8874d6f2c8:/# which vim
/usr/bin/vim
```
上次安装的vim依然在，可见docker保存了容器的状态(persistence)。
前面所创建的容器是交互式的容器，而实际的使用当中，我们需要容器去运行程序或者服务，这种又称为daemonized container. 下面是一个简单的例子:

```bash
docker@boot2docker:~$ docker run --name awesomeness -d ubuntu /bin/sh -c "while :; do echo hello; sleep 5; done"
1bda4fdc3a3b869d1dc58bb4564b08e1bc0c0f24372f48afb8bcc4291cc94929
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
1bda4fdc3a3b        ubuntu:latest       "/bin/sh -c 'while :   55 seconds ago      Up 54 seconds                           awesomeness
```
使用`docker logs`命令可以查看容器的日志:

```bash
docker@boot2docker:~$ docker logs awesomeness
hello
hello
hello
hello
hello
hello
```
加上`-t`选项可以输出时间戳，`-f`跟踪最新的日志:

```bash
docker@boot2docker:~$ docker logs -ft awesomeness
2015-01-17T13:17:43.827474614Z hello
2015-01-17T13:17:48.842826627Z hello
2015-01-17T13:17:53.847642897Z hello
2015-01-17T13:17:58.856081585Z hello
2015-01-17T13:18:03.863587257Z hello
```

查看容器内的进程:

```bash
docker@boot2docker:~$ docker top awesomeness
PID                 USER                COMMAND
922                 root                /bin/sh -c while :; do echo hello; sleep 5; done
1454                root                sleep 5
```

通过Docker在容器内部启动新的进程，执行新的任务:

```bash
docker@boot2docker:~$ docker exec -d awesomeness touch /tmp/testfile
docker@boot2docker:~$ docker exec -t -i  awesomeness  /bin/bash
root@1bda4fdc3a3b:/# ls -al /tmp/testfile
-rw-r--r-- 1 root root 0 Jan 17 14:03 /tmp/testfile
```
不得不说docker的发展真的很快，我最开始使用的时候docker的版本大概是在0.6，当时的容器技术还是使用lxc，
只能用`lxc attach`命令才能连接到运行的容器的控制台。现在使用这么简单，必须要点个赞。

停止运行的容器:

```bash
docker stop awesomeness   # 用containerid 1bda4fdc3a3b 效果相同
```

在容器意外退出时，自动重启容器:

```bash
docker@boot2docker:~$ docker run --restart=always --name awesomeness -d ubuntu /bin/sh -c "while :; do echo hello; sleep 5; done"
```
还可以设置重启的次数，`--restart=on-failure:5`。这个功能也很不错，让容器拥有了一定的自恢复(self-healing)能力，

查看容器的信息:

```bash
docker@boot2docker:~$ docker inspect awesomeness
[{
  "AppArmorProfile": "",
  "Args": [
  "-c",
  "while :; do echo hello; sleep 5; done"
  ],
  "Config": {
    "AttachStderr": false,
    "AttachStdin": false,
    "AttachStdout": false,
    "Cmd": [
    "/bin/sh",
    "-c",
    "while :; do echo hello; sleep 5; done"
    ],
    "CpuShares": 0,
    "Cpuset": "",
    "Domainname": "",
    "Entrypoint": null,
........
```
inspect可以返回JSON格式的详细配置信息，不过这个信息略多，通过`--format`可以格式化输出的内容，假设我只想看到容器的IP地址:

```bash
docker@boot2docker:~$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' awesomeness
172.17.0.3
```
有点像调用了`jq`命令。

删除容器的命令是`docker rm CONTAINER_ID or CONTAINER_NAME`，这个操作只能针对已经停止运行的容器。删除所有容器的命令是:

```
docker rm `docker ps -a -q`
```
这个命令会返回所有容器的ID，然后执行删除操作。实际的使用中我们可能只是删除部分的容器，可能就得结合`cut`,`grep`或者`awk`等命令来拿到要删除的容器ID/名字，然后再去执行删除操作。
