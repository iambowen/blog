---
layout: post
title: "Specifying Amazon Credentials in Packer"
date: 2015-12-02 20:50:33 +0800
comments: true
categories: ["packer", "AWS"]
---
[Packer](https://packer.io/)是一个用一份配置构建跨平台镜像的工具，它支持EC2 AMI，Vmware，
QEMU，Virtualbox，docker等多个平台。

我们使用AWS作为基础设施平台，通过EC2 AMI作为镜像部署，构建AMI工具基于Packer，加上了一些定制
的东西以减少配置，简化使用的方式。

Packer在构建AMI时，需要创建临时的key pair,security group以及EC2实例，因为需要对应的API
credentials,像这样:

``` bash
access key id:     AKIAIOSFODNN7EXAMPLE
secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
所以Packer需要在配置文件中hard code credentials或者通过环境变量传入，如果这些都没有找到，
Packer会通过如下的步骤去自动查找credential:

1.查找AWS相关环境变量，如:

```bash
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
2.寻找AWS配置文件`~/.aws/credentials`或者`AWS_PROFILE`环境变量

3.查找运行Packer的EC2实例中[instance profile](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)中的role，然后用AWS STS(Security Token Service)来获取临时的credential.该role的policy至少要允许如下的Actions:

```json
{
  "Statement": [{
      "Effect": "Allow",
      "Action" : [
        "ec2:AttachVolume",
        "ec2:CreateVolume",
        "ec2:DeleteVolume",
        "ec2:CreateKeypair",
        "ec2:DeleteKeypair",
        "ec2:CreateSecurityGroup",
        "ec2:DeleteSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateImage",
        "ec2:RunInstances",
        "ec2:TerminateInstances",
        "ec2:StopInstances",
        "ec2:DescribeVolumes",
        "ec2:DetachVolume",
        "ec2:DescribeInstances",
        "ec2:CreateSnapshot",
        "ec2:DeleteSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DescribeImages",
        "ec2:RegisterImage",
        "ec2:CreateTags",
        "ec2:ModifyImageAttribute"
      ],
      "Resource" : "*"
  }]
}
```
这几种方式中，使用instance profile的的方式是最好的，原因大致如下:

1. 固定credential的方式会带来安全隐患，一旦泄露，会导致巨大的损失。
2. 从安全的角度考虑，API credential需要rotate，这样会带来维护成本。读取配置文件也有同样的
问题。
3. 在持续集成流水线AMI构建任务中，credential必须通过环境变量的方式传入，任务本身增加了额外的
依赖，维护成本提高。

之所以总结Packer在构建AMI时指定credential的方式，是因为最近在把一个用packer构建AMI的流水线
从一个Region移到另一个Region时遇到了原有的credential不工作的情况，后来发现，其实运行这个任务
的Agent(EC2实例)初始化时就绑定了instance profile，不用设置任何额外的credential，packer可以
自动去读取，于是在移除了原有的各种credential之后，顺利的构建出了镜像。
