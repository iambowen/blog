---
layout: post
title: "introduction to boot2docker"
description: "simple introduction to boot2docker"
category:
tags: [docker, boot2docker]
---

[boot2docker](http://boot2docker.io/) 是一个轻量级的Linux发行版，它基于[Tiny Core Linux](http://tinycorelinux.net/)，主要用于运行Docker容器。
它的主要特性有:

1. 有AUFS文件系统支持的最新内核, 最新的docker版本
2. 通过自动挂载`/var/lib/docker`来实现容器的持久性
3. 通过磁盘自动挂载实现SSH key的持久性
4. 通过Host-only网络设置，可以很容易的访问到docker映射的端口

除此之外，它运行时只占用27MB左右内存，启动时间大约只要5s，同时它还支持多个平台。因为容器技术是需要linux高版本的支持，所以在Windows和OSX上运行，同时要享受容器带来的资源红利，都是以虚拟机的形式运行。
本质上和Vagrant启动虚拟机运行Docker没有什么太大区别，不过使用更加方便，资源消耗较少(个人浅见)。

### 安装
----
OSX下用`brew`就可以直接安装:

```bash
brew install boot2docker
```
Windows用户请自觉安装Linux发行版。

### 初始化
----
创建一个新的boot2docker虚拟机

```bash
boot2docker init
```

### 启动
----

```bash
~> boot2docker up
Waiting for VM and Docker daemon to start...
....................oooo
Started.
Writing /Users/docker/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/docker/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/docker/.boot2docker/certs/boot2docker-vm/key.pem

To connect the Docker client to the Docker daemon, please set:
export DOCKER_TLS_VERIFY=1
export DOCKER_HOST=tcp://192.168.59.103:2376
export DOCKER_CERT_PATH=/Users/docker/.boot2docker/certs/boot2docker-vm

```

最新的docker支持了tls加密通信方式，要连接Docker后台的Docker客户端，需要export以上的几个环境变量。

### 杂项命令

```bash
~> boot2docker info
{
  "Name": "boot2docker-vm",
  "UUID": "f7cdbcf4-31f5-4dc0-8d4a-83d38a573a50",
  "Iso": "/Users/docker/.boot2docker/boot2docker.iso",
  "State": "running",
  "CPUs": 4,
  "Memory": 2048,
  "VRAM": 8,
  "CfgFile": "/Users/docker/VirtualBox VMs/boot2docker-vm/boot2docker-vm.vbox",
  "BaseFolder": "/Users/docker/VirtualBox VMs/boot2docker-vm",
  "OSType": "",
  "Flag": 0,
  "BootOrder": null,
  "DockerPort": 0,
  "SSHPort": 2022,
  "SerialFile": "/Users/docker/.boot2docker/boot2docker-vm.sock"
}
```
可以看到虚拟机的配置

```bash
~> boot2docker ip

The VM's Host only interface IP address is: 192.168.59.103
```
可以查看boot2docker vm的ip地址。

```bash
boot2docker ssh
```
可以ssh到boot2docker虚拟机中，如果用`free -m`命令查看下当前使用到的内存，发现果然只用了几十兆，nice.

### 共享目录
boot2docker虚拟机缺省的共享目录是host machine的`/Users`目录，想添加新的共享目录的，除了通过VirtualBox的虚拟机配置界面添加之外，还可以通过`VBoxManage`命令

```bash
boot2docker stop # 确保虚拟机处于关闭状态

VBoxManage sharedfolder add boot2docker-vm --name test --hostpath ~/github

boot2docker start

boot2docker ssh

root@boot2docker:~# mkdir -p /mnt/test
root@boot2docker:~# mount -t vboxsf test /mnt/test

```
即可。

----
boot2docker 提供的功能还不止这些，通过 `boot2docker -h`可以查看其他功能命令及用法，至于在虚拟机内使用docker，那就是另外一个问题了。
