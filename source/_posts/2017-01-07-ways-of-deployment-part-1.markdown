---
layout: post
title: "ways of deployment: part 1"
date: 2017-01-07 19:59:49 +0800
comments: true
categories: ['deployment', 'IaC', 'chef', 'puppet', 'ansible']
---

### 什么是基础设施
---

在IT领域，当我们谈论基础设施时，我们都在谈论什么呢？一般来讲，我们会直觉的认为服务器就是基础设施，甚至在Infrastructure as Code的[wiki](https://en.wikipedia.org/wiki/Infrastructure_as_Code)页面也是这么举例的，不过我不是很认同，我觉得基础设施应该包括提供给业务相关的应用所有基础保障的服务和设施，比如:

1. DNS/CDN 
2. 防火墙/Load Balancer
3. 应用服务器、数据库(物理机/虚拟机)
4. 日志、监控、报警服务

### 基础设施管理面临的挑战
---

业务的快速发展要求基础设施的灵活性，更快的部署速度，更快的上线时间，自恢复的系统，但是传统的IT运维的方式在基础设施管理面前给我们带来的很多的挑战:

1. `服务器蔓延(Server Sprawl)`。在单块架构下，服务器的数量和需要配置的种类都比较少，然而随着业务发展，或者微服务拆分等，服务器数量，所需配置的种类可能会爆炸式增长，沿用传统的管理方式挑战很大，而且对于相同的服务器可能会导致配置的差异。
2. `配置漂移(Configuration Drift)`。服务器的配置可能会随着时间增加。比如有人为了解决一个特定用户的问题，修改了其中一台服务器的配置，这样他们之间就存在了差异。 很有可能会发生，只有在某个环境里面的台服务器上，应用才能正常运行的情况。
3. `雪花服务器(Snowflake Servers)`。雪花服务器的意思是该服务器和你的网络中任意其它的服务器都不同，特殊到无法复制。比如，在别的服务器上升级ruby语言后，应用可以运行，但是在某台机器上就是不可以。
4. `脆弱的基础设施(Fragile Infrastructure)`。总有一些服务器，在你on-call的时候，你需要对着它们拜一拜，祈祷它们不要出问题。
5. `自动化恐惧症(Automation Fear)`。缺乏对自动化的信心因为我的服务器配置不是一致的。我的服务器不一致是因为我没有频繁和一致的运行自动化。
6. `侵蚀(Erosion)`。侵蚀就是问题随着时间的推移蔓延到正在运行的系统的意思。比如，服务器磁盘被日志文件塞满，操作系统升级，内核补丁，以及基础设施软件（如Apache，MySQL，SSH，OpenSSL）升级去修复安全漏洞等。

### 基础设施即代码定义及原则
---

针对以上的问题，解决的办法是将基础设施作为代码，版本管理起来。基础设施即代码是基于从软件开发实践的基础设施自动化的方法。它强调配置和改变系统及其配置的一致性，可重复的程序。变更转化为定义，然后通过包括彻底的验证的无人值守过程应用到系统中。其原则如下:

1. `容易重现的系统`。能够毫不费力且可靠地重建基础设施中的任何元素。
2. `可任意处理系统`。可以轻松创建、销毁、替换、更改以及移动资源。
3. `一致的系统`。假设两个基础设施元素提供相似的服务，比如同一个集群中有两个应用程序服务器。这些服务器应该几乎完全相同。它们的系统软件和配置应该是一样的，除了一丁点配置（比如IP地址）用于区分彼此。
4. `可重复的过程`。基于可再生原则，对基础设施执行的任何行为都是可以重复的。也就是说`Duang`了之后，对于所有人的效果应该是一样的。
5. `变化的设计`。确保系统能够安全地改变，迅速的频繁做出变化。

### 基础设施即代码的实践
---

1. 使用定义文件。如Chef cookbook, Ansible Playbook等。
2. 一切版本化。所有的配置管理文件都用CVS工具如git管理起来。
3. 持续测试系统和流程。自动化测试和持续交付/部署流水线。
4. 小的变更而不是批量变更。小的变更，测试和回退的难度更小。
5. 让服务持续可用。通过冗余/DR等方式保持服务的可用性。

### 基础设施即代码的工具
---
市面上的基础设施、配置管理的工具很多，我用过的有`chef`， `puppet`，`ansible`以及`cloudformation`。其中`chef`是很多年前用于管理测试环境的工具，`puppet`用于管理数据中心遗留系统的工具，`ansible`用于少量的系统如日志系统Splunk的工具，`cloudformation`是我们在AWS上统一的配置管理、部署工具。
我在[github](https://github.com/iambowen/microservice-training-lesson-9-deployments)做了关于这几个工具的小demo，大家可以感受下几种工具的差别。它们完成的事情都是在一个Ubuntu14.04的虚拟机上，配置和安装docker，并且用Docker运行一个简单的应用，访问时可以返回`Ciao mondo.`(意大利语，世界你好的意思)。
直接运行的效果就是这样:

```bash
docker run -d -p 8080:80 iambowen/ciao:alpine

curl localhost:8080
```
让我们分别看看用`chef`、`puppet`、`ansible`来实现的过程。

#### Chef
----
[Chef](https://www.chef.io/chef/) 是用基于Ruby实现的自动化配置管理工具。它有两种运行模式，Server-Client以及Chef-Solo的模式。其中Server-Client模式中，必须有chef的agent驻守在node上，并将节点注册在Chef Server中，同时同步node上相关的cookbook，并编译应用到节点上。另一种是以Chef Solo的方式，将所需的Cookbook等配置文件下载/上传到node，然后编译运行，不依赖Chef Server。

![arch](http://www.dayshaconsulting.com/wp-content/uploads/2015/01/chef-block-diagram.jpg)
[这里](https://github.com/iambowen/microservice-training-lesson-9-deployments/tree/master/chef)我们使用chef solo的方式，将所需的docker相关的cookbook放在固定的路径下，由vagrant来完成chef client的安装，之后再用Chef根据recipe的配置去做部署，包括docker的安装、pull镜像以及运行容器。

vagrantfile的配置

```ruby
 config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "docker"
    chef.add_recipe "ciao"
  end
```
`ciao`应用的recipe文件

```ruby
docker_service 'default' do
  action [:create, :start]
end

docker_image 'ciao' do
  repo 'iambowen/ciao'
  tag 'alpine'
  action :pull
end

docker_container 'ciao' do
  repo 'iambowen/ciao'
  tag 'alpine'
  port '8088:80'
end
```
Chef是基于Ruby的DSL实现，所以写Chef的脚本，比起写Ruby的脚本要稍微简单一点点。Chef还提供了一些其他的工具，比如Knife，公司的虚拟化解决方案用的是VMware VShpere，曾经使用Knife来管理虚拟机(创建/销毁)。我们使用是在几年前在AWS上构建端到端的测试环境时使用Chef来做不同应用/服务器的配置管理和部署的，规模接近500台。

### Puppet
---
[Puppet](https://puppet.com/)是另一个自动化配置管理工具。和Chef类似，也是两种运行模式，Master-Agent模式，以及Standalone模式。Master-Agent模式里面，agent需要创建client side certificate和Puppet Master通过SSL通信，获取该节点的catalog/manifests，然后编译运行。Puppet Master以前是用Ruby实现的，好像后来用Scala重写了，但是它的配置文件的格式还是类似于Ruby的DSL。Puppetlabs每年还有年度DevOps报告，内容不错，咨询师或者需要和领导吹比的可以看看。
![puppet arch](http://www.slashroot.in/sites/default/files/Working%20of%20Puppet%20Configuration%20Mangement%20tool.png)

[这里](https://github.com/iambowen/microservice-training-lesson-9-deployments/tree/master/puppet)是利用puppet配置和部署的代码。因为这个`ubuntu14.04`的基础镜像不包括puppet，同时我想偷懒，直接用vagrant运行inline的shell脚本将puppet docker module安装，之后再用puppet的agent去应用配置。

```ruby
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    gem install puppet -v '3.7.5'
    gem install facter
    puppet module install --modulepath /etc/puppet/modules garethr-docker 
  SHELL

  config.vm.provision "puppet" do |puppet|
    puppet.manifests_path = "puppet/manifests"
    puppet.manifest_file = "ciao.pp"
  end
```

实际的puppet脚本是这样的

```ruby
include 'docker'

docker::run { 'ciao':
  image   => 'iambowen/ciao:alpine',
  ports   => "8088:80"
}
```
因为我们的数据中心的遗留系统都是在用puppet去做管理，所以接触的稍微多些，也踩过一些坑，比如[这个](http://iambowen.github.io/puppet/2015/04/16/puppet-cron-pitfall/)，这样的工具在方便你使用的同时也隐藏了具体的实现，一旦出现问题，debug的成本比较高。我们用puppet管理两个数据中心大约在2000-3000台的服务器。

###  Ansible
----
前面提到Chef和Puppet都是需要在节点/服务器上安装代理(chef client/puppet agent)，以这种pull的模式去获取配置文件，应用。这就意味着你的节点服务器上会存在额外的依赖，举个例子，如果你的应用基于Ruby2.3，但是提供部署的puppet agent只能运行在ruby1.9.3下面，你就得在同一套环境下准备两个ruby的环境，有没有觉得很膈应，管理的难度也加大了。至少在几年前，对我们造成了比较大的伤害，当时在开发时流行用rvm或者rbenv去管理ruby的环境，但是在生产环境的服务器用这些东西是比较奇怪和不靠谱的。
相比之下，[Ansible](https://www.ansible.com/)做的事情非常简单，写yaml格式的配置文件，ssh到应用服务器，应用具体的配置。服务器上只需要有Python的环境以及一些相关的包。
![ansible arch](https://d1cg27r99kkbpq.cloudfront.net/static/extra/43-ansible-multi-node-deployment-workflow.png)

[这里](https://github.com/iambowen/microservice-training-lesson-9-deployments/tree/master/ansible)是利用ansible配置和部署的代码。

```ruby
 config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.limit = "all"
    ansible.inventory_path = './inventory'
  end
```

部分的[playbook](https://github.com/iambowen/microservice-training-lesson-9-deployments/blob/master/ansible/playbook.yml)如下:

```yaml
---
- hosts: all
  sudo: yes
  user: vagrant
  tasks:

    - name: install docker-py package
      pip:
        name: docker-py
        state: latest

    - name: running ciao app
      docker_container: 
        image: 'iambowen/ciao:alpine'
        name: ciao
        expose: 80
        ports: 
        - 8088:80
        pull: true
```
如果看完整的playbook，会发现它多做了很多事情，比如添加额外的docker源，安装docker等，比起Chef和Puppet脚本要长些，但是它们的格式都是yaml，而且对应的文档都可以通过`ansible-doc`获取到，学习的成本比起Chef和Puppet来讲要低很多，同时，也省去了Chef Server和Puppet Master维护的成本，比较省心。因为我们的基础设施以及全面向AWS移植，AWS提供了更好的配置管理和部署工具`Cloudformation`,所以我们只有在很少的情况下使用了Ansible，比如管理基础镜像、日志服务器更新等。

### 基础镜像 + 包(RPM/Deb) + Config Service
---
回顾上面的几种工具，他们在配置管理时做的事情，大致有两种，首先是依赖管理，如安装应用运行所需的依赖，第二是配置管理，根据具体的环境(staging/production)应用不同的配置文件。如果服务器通过基础镜像生成，基础镜像中包含了运行基础设施相关的组件，如日志客户端、监控客户端等等，应用代码可以通过RPM/Deb打包(依赖自包含)，安装时yum/aptitude会帮你安装依赖。而应用启动的配置，可以用过环境变量传入，不同环境依赖的配置，可以连接config service(zookeeper等)获取。

[rpm](https://github.com/iambowen/bbs_team_b/blob/master/bbs.spec) 打包的例子:

```bash
BuildRoot:  %{_tmppath}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)
BuildArch:  noarch
Requires:   java-1.7.0-openjdk tomcat6
%description
This package installs the Resi REST Services with embedded server.
%pre
%install
rm -rf %{buildroot}
mkdir -p %{buildroot}/usr/share/tomcat6/webapps/bbs_team_b/
cp -r ../SOURCES/* %{buildroot}/usr/share/tomcat6/webapps/bbs_team_b/
```
基于基础镜像，RPM包和配置服务器的示例图:
![arh](images/rpm_config_service.png)

这样做的好处在于：
1. 打包的脚本和配置文件可以和生产代码放在一起，开发人员对于生产环境拥有了可见性，同时具体的配置值放在config service中，对大部分人是不可见的
2. 免去了自动化配置工具如Chef、Puppet等的学习、维护的成本，同时这种方式配置部署，其代码库对于大部分人是不可见的
3. 对于系统的安全补丁，只需要更新基础镜像既可，之后重新部署即可，维护的成本大大降低

不好的地方在于有额外的依赖，比如需要维护yum源，我们使用Koji去维护内部的yum源，每次新的rpm包push到koji时，koji需要重新index，更新metadata，花费的时间会比较多，无法在持续交付流水线上立即部署。其次维护config service也会有额外的成本。当然现在也可以在打包rpm的时候生成metadata，同步到s3上作为YUM源。

### Cloudformation
----
前面提到的三种工具，看上去都是对服务器做配置管理和部署，但是实际上，对于其他的基础设施，比如网络配置等也可以做到代码化。这里我们以AWS的`cloudformation`为例来介绍。
AWS的[cloudformation](https://aws.amazon.com/cloudformation/)可以让你通过`json`或者`yaml`格式的模板，来管理AWS几乎所有的基础设施资源，同时对于应用提供了`immutable deployment`的零宕机时间部署。
[这里](https://github.com/iambowen/microservice-training-lesson-9-deployments/blob/master/aws/cloudformation/template/vpc.json)是我用来生成自己的VPC网络的cloudformation模板（感觉钱包在滴血-_-!），

比如下面的模板片段，在我新建的VPC中，会包含两个private subnet，两个public subnet，分别对应两个az(数据中心)的网络，以及其它的一些网络配置，nat、网络的acl等。如果我有新的配置修改，比如acl的变更等，可以直接通过cloudformation模板更新，不需要在aws console上做任何手动的修改，数据中心的网络直接就被代码化了。

```json
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "A vpc template",
    "Parameters": {
    },
    "Metadata": {
    },
    "Resources": {
        "NetworkAclPrivate": {
            "Type": "AWS::EC2::NetworkAcl",
            "Properties": {
                "VpcId": {
                    "Ref": "Vpc"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": {
                            "Fn::Join": [
                                "",
                                [
                                    {
                                        "Ref": "AWS::StackName"
                                    },
                                    "-acl-private"
                                ]
                            ]
                        }
                    }
                ]
            }
        }
    }
}
```

cloudformation还支持yaml格式的配置，比如[例子](https://github.com/iambowen/microservice-training-lesson-9-deployments/blob/master/aws/cloudformation/template/ciao.yml)中的配置。如果我通过命令行调用`aws cloudformation create-stack --stack-name ciao --template-body file://aws/cloudformation/template/ciao.yml --capabilities CAPABILITY_IAM`，那么我可以生成一个ELB，一台基于Ubuntu的EC2实例，该实例通过一个AutoScaling Group去管理，上面运行了`iambowen/ciao:alpine`的容器，并且该ELB只接收http请求，该实例绑定的安全组只接收来自ELB的请求。

```yaml
loadBalancer:
    Type: AWS::ElasticLoadBalancing::LoadBalancer
    Properties:
      Scheme: internet-facing
      Subnets:
      - subnet-d7c254b3
      - subnet-90d866e6
      SecurityGroups:
      - Ref: loadBalancerSecurityGroup
      Listeners:
      - Protocol: HTTP
        LoadBalancerPort: 80
        InstancePort: 80
      HealthCheck:
        Target: HTTP:80/
        HealthyThreshold: 2
        UnhealthyThreshold: 4
        Interval: 10
        Timeout: 8
      CrossZone: true
      ConnectionDrainingPolicy:
        Enabled: true
        Timeout: 30
launchConfiguration:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      IamInstanceProfile:
        Ref: iamInstanceProfile
      ImageId: ami-8ed6eaed
      AssociatePublicIpAddress: false
      InstanceType: t2.micro
      InstanceMonitoring: true
      SecurityGroups:
      - Ref: instancesSecurityGroup
      UserData:
        Fn::Base64:
          Fn::Join:
          - "\n"
          - - "#!/bin/bash -eu"
            - docker run -d --name ciao -p 80:80  -e MESSAGE='Goodbye!' iambowen/ciao:alpine
            - docker ps
            - ''
            - echo; echo --- SUCCESS
            - RESOURCE_STATUS=0
            - ''
```
每次AWS的配置、容器或者配置需要更新时，只需要修改cloudformation模板中对应的配置，运行下update stack操作即可. 如 `aws cloudformation update-stack --stack-name ciao --template-body file://aws/cloudformation/template/ciao.yml --capabilities CAPABILITY_IAM`

我们对于AWS的环境中的资源，几乎都通过cloudformation生成，这种基础设施即代码的程度，大大降低了我们维护、迁移的成本。
### 总结
---
文中介绍的几种工具都比较成熟，对于配置管理、部署等，都没有太大问题，选择的时候根据具体的情况，比如虚拟化解决方案等。下一节我将介绍部署的几种模式，如蓝绿部署、红黑部署、immutable部署等。

### Reference
---
1. 文中部分内容参考了我参与翻译的[《Infrastructure as Code: Managing Servers in the Cloud》](https://www.amazon.cn/dp/1491924357) 以及 [《DevOps实践》](https://www.amazon.cn/dp/B01LWLRQF3/)
2. 基础设施即代码的工具部分的代码[链接](https://github.com/iambowen/microservice-training-lesson-9-deployments)