---
layout: post
title: "Delete old VMs from Vagrant"
description: ""
category:
tags: [vagrant]
---

想升级Vagrant配置文件中的Ubuntu，但是不想改变box的名字，因此需要删除旧的box，以及VM。

```bash
 ~> vagrant box list
centos  (virtualbox)
debian  (virtualbox)
lucid64 (virtualbox)
ubuntu  (virtualbox)   #需要删除的box
```
从Vagrant中删除box

```bash
 ~> vagrant box remove ubuntu
Removing box 'ubuntu' with provider 'virtualbox'...

 ~> vagrant box list
centos  (virtualbox)
debian  (virtualbox)
lucid64 (virtualbox)
```
从VirtualBox中删除虚拟机

```bash
#列出Virtualbox中当前的虚拟机
 ~> VBoxManage list vms
"centos_default_1389778387460_58997" {1b6e8415-ce81-4690-b229-981cf5c1f09f}
"debian_1386125850" {b9dd3091-d036-4fab-b9f4-5b6c6b2faf16}
"ubuntu_default_1394781826776_87151" {c1487df0-119a-411d-9d84-fcb6f983b200}
"VagrantChefHubot_default_1390458896900_28199" {d55d7983-3ab4-45b6-a20c-cb2125b4eff8}
"hubotVagrant_default_1390464057664_43197" {bc429ec4-0bf9-4486-a460-097329382135}
"graphite_default_1390985918603_95953" {e87ba34a-a00a-44ec-99f0-f19c37f35c18}
"riemann_default_1391836269341_83882" {e967375e-7498-4f48-a063-7641eddd53b1}
#unregister想要删除的虚拟机，加--delete可以删除vm
 ~> VBoxManage unregistervm ubuntu_default_1394781826776_87151 --delete
 0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
#double check下虚拟机已经被删除
  ~> VBoxManage list vms
"centos_default_1389778387460_58997" {1b6e8415-ce81-4690-b229-981cf5c1f09f}
"debian_1386125850" {b9dd3091-d036-4fab-b9f4-5b6c6b2faf16}
"VagrantChefHubot_default_1390458896900_28199" {d55d7983-3ab4-45b6-a20c-cb2125b4eff8}
"hubotVagrant_default_1390464057664_43197" {bc429ec4-0bf9-4486-a460-097329382135}
"graphite_default_1390985918603_95953" {e87ba34a-a00a-44ec-99f0-f19c37f35c18}
"riemann_default_1391836269341_83882" {e967375e-7498-4f48-a063-7641eddd53b1}
```
Nicely done!
