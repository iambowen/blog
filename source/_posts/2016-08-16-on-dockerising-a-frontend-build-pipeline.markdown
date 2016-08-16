---
layout: post
title: "on dockerising a frontend build pipeline"
date: 2016-08-16 13:08:58 +0800
comments: true
categories:["docker","ci"]
---
最近花了一段时间把主站的build pipeline docker化了，时间长到感觉自己的reputation都要被毁了。
在此总结下这个过程以及碰到的问题，希望对大家能有所帮助。

### 背景
这是一个纯前端的项目，两年前前后端分离的时候的项目，Grunt workflow，测试框架使用Karma，用Phantomjs`1.8.2`运行headless的测试，开发环境使用Chrome/Safari做功能性测试。开发环境基于node `0.12`，一些基础设施的更新，部署的脚本，smoke test是基于Ruby的，版本为`2.0`。

这个前端的工程部署在两个不同的AWS Region的S3上，互为fail over，最前面有Akamai为它们做负载均衡。

持续集成的工具使用Bamboo，其agent需要有`ruby 2.0`，`node 0.12`，`Phantomjs 1.8.2`的环境才可以运行具体的任务。整个过程已经做到了持续部署，一个完整的build过程如下：

1. 提交代码
2. trigger build，执行单元测试和集成测试
3. 自动部署staging 环境
4. 自动部署production 环境
5. 对部署后的产品做性能测试
6. 上传工程中依赖的第三方类库信息到S3 bucket(出于安全的考虑)

### 存在的问题
太长时间没有人做技术上的升级，导致下面的一些隐患和问题：
1. 开发的工具版本落后，node当前版本已经是`6.3`了，ruby 2.0的版本应该已经不维护了，同样，对应的Karma，Phantomjs都以及更新了很多
2. 运行build依赖的agent是共用的，如果有人对agent的环境进行修改，会影响该项目的持续集成
3. 未来需要将CI工具从Bamboo迁移到Buildkite，用pipeline as code的方式去构建，每个组自己去管理build agent，使用Docker会更加方便迁移

### 过程以及遇到的一些难点

测试部分通过的过程及问题
1. 首先做的事情是构建一个基础的docker 镜像，包含最新的node `6.3.1`，phantomjs `2.1.1`，后来发现其实不用Phantomjs，这个有点多余了。 成果在这里: https://github.com/iambowen/node_on_docker，因为这样的环境更加通用些，所以才publish到官方的docker repository里面。
2. 在这个镜像的基础上，构建一个我们工程依赖环境的基础镜像，额外安装了Ruby `2.3`，最新的Chrome，git以及一些git的配置，因为需要从企业版github上pull代码。
3. 本地升级node版本，以及相关的grunt，karma，Phantomjs的版本，运行测试通过。
4. 将工程mount容器中，然后运行测试，`npm install`失败，原因是安装`fsevent`出错。查看了下这个包，原来只是给OSX下使用的。删除`npm-shrinkwrap.json`后重新运行可以通过。原因是有人在OSX下运行了`npm shrinkwrap`去生成的这个锁定版本的文件，真是烦人。于是反其道行之在容器里面生成`npm-shrinkwrap.json`，在host上运行测试一切完好，就这样解决了这个问题。
5. 在Bamboo创建一个branch，然后针对我的分支代码运行测试
6. 测试里面的一个步骤是做`bower install`安装第三方js类库，但是比较恶心的是，有些第三方类库是以`git`的协议去下载，而不是`https`。本地运行一切都好，但是在Bamboo Agent上运行的时候却出现了连接超时的问题，很有可能是Bamboo所在AWS的network ACL或者是security group没有允许`9418`端口的TCP访问。不过最后解决的方式并不是修改防火墙或者将协议改为`https`，而是直接把类库checking到git中，这样对应的修改Gruntfile，不用再运行`bower install`。check in之后在Bamboo上运行还是失败，本地却可以通过，仔细检查，原来是一部分bower module目录名为`dist`被git ignore掉了。

通过测试后，接下来就是部署了。部署要解决的问题是，如何让容器拿到AWS role的动态权限去做文件的上传更新操作。ECS好像是支持容器去assume role的操作，但是我们没有用ECS，所以只能考虑其它方式。

我想到的方式在bamboo 的 docker agent上 `assume role`，拿到对应的credential后，将其作为环境变量传入到容器中。实验证明这样的方式是可行的，万幸bamboo的docker agent支持aws cli命令，不过没有`jq`稍微增大了点提取credential的难度，脚本如下：

```
if [ "$DEPLOY_ENV" = "staging" ]; then
  AWS_ACCOUNT_ID='1111111111'
elif [ "$DEPLOY_ENV" == "production" ]; then
  AWS_ACCOUNT_ID='2222222222'
fi

credentials=$(aws sts assume-role  --role-arn       arn:aws:iam::"$AWS_ACCOUNT_ID":role/roleName \
          --role-session-name roleSessionName \
            --query 'Credentials.[SecretAccessKey, SessionToken, AccessKeyId]'  \
            --output text)

SecretAccessKey=$(echo $credentials | cut -d' ' -f1)
SessionToken=$(echo $credentials | cut -d' ' -f2)
AccessKeyId=$(echo $credentials | cut -d' ' -f3)

docker run  -e BUILD_VERSION="$BUILD_VERSION" \
    -e DEPLOY_ENV="$DEPLOY_ENV" -e AWS_SECRET_ACCESS_KEY="$SecretAccessKey" \
    -e AWS_SESSION_TOKEN="$SessionToken" -e AWS_ACCESS_KEY_ID="$AccessKeyId" --rm docker_image bash -c 'grunt deploy'
```
因为部署是用aws node 的sdk，所以读取的环境变量名字不太一样，要稍微注意下。

在CI上运行后，staging部署通过，手动在bamboo的docker agent上测试下是否能assume产品环境的部署的role，结果可以，那就是说产品环境的部署应该也可以通过了。

### 总结
1. `npm` sucks，更糟糕的是程序员在引入依赖的时候缺乏考虑，我在`package.json`里面见到了不少无人维护的component，后续的升级维护是一个问题，联想以前的ruby项目也是一样。一旦有版本升级，碰到无人维护的gem时会非常痛苦。
2. 一个工程里面用了太多的语言，也是一件很糟糕的事情，明明可以用node的aws sdk来做到所有的部署，不知道为何用ruby去实现，无形中增大了维护的成本。
3. 一般来说，我们认为docker可以保证不同环境的一致性，但是由于一些特殊原因，如我上面提到的防火墙问题，bower module被git ignore掉的问题，在CI环境下才能暴露出来。所以在PR被merge到master之前，一定要保证修改在CI上也运行通过。
