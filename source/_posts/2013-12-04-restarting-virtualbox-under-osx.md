---
layout: post
title: "restarting virtualbox under osx"
description: ""
category: 
tags: ["virtualbox"]
---


使用vagrant启动虚拟机的时候遇到了如下的错误:


```bash
 ~> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Available bridged network interfaces: 
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
The guest machine entered an invalid state while waiting for it
to boot. Valid states are 'starting, running'. The machine is in the
'poweroff' state. Please verify everything is configured
properly and try again.

If the provider you're using has a GUI that comes with it,
it is often helpful to open that and watch the machine, since the
GUI often has more helpful error messages than Vagrant can retrieve.
For example, if you're using VirtualBox, run `vagrant up` while the
VirtualBox GUI is open. 
```


错误信息提示我虚拟机所处状态(poweroff)，不能启动。原因未知，不过除了信息中给出的解决办法，还可以通过重启VirtualBox来解决。办法如下:


```bash
 ~> sudo /Library/StartupItems/VirtualBox/VirtualBox restart
Password:
Unloading VBoxUSB.kext
Unloading VBoxDrv.kext
/Applications/VirtualBox.app/Contents/MacOS/VBoxAutostart => /Applications/VirtualBox.app/Contents/MacOS/VBoxAutostart-amd64
………..
Loading VBoxDrv.kext
Loading VBoxUSB.kext
Loading VBoxNetFlt.kext
Loading VBoxNetAdp.kext 
```

然后就可以顺利的通过Vagrant启动虚拟机了。
