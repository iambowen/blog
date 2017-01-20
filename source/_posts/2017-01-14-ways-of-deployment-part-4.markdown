---
layout: post
title: "ways of deployment: part 4"
date: 2017-01-14 09:20:05 +0800
comments: true
categories: ["deployment", "serverless", "aws Lambda"]
---
在[前一部分](http://www.jianshu.com/p/6d569f0861a6)我们提到使用Docker部署应用同时采用12factors原则时，服务器对于容器来讲，只依赖它的计算资源，那么如果有一个提供运行时环境的黑盒功能，让我们可以直接部署代码去运行，那敢情真的太棒了。其次，从资源利用率的角度来说，在单块架构下，如果应用的某个功能模块需要水平扩展，那么整个应用都得和它一起水平扩展，这是一种资源的浪费。微服务架构各个功能模块分解成单独的微服务，每个服务独立部署，独立扩展，一定程度上降低了资源的浪费。但是我们仔细看一下，假设一个用户管理的微服务，请求`/login`的endpoint要比请求`register`的endpoint多的多，如果因为`/login`的请求数量多需要扩展，那么`/register`得和它一起扩展，那`/login`不禁就要发问，你`/register`有何德何能去承受这些资源。同样的道理可以延伸到Restful endpoint的每个函数。

|Restful API   | RPM  |
|---|---|---|
| `/login`  | 1000  |
| `/register`  | 100  |

如果你的微服务粒度很小，容器和PaaS一定程度上可以解决上面的问题，如果我完全不想关心应用服务器或者容器，怎么办。Here comes Serverless。

## 什么是Serverless
---
鉴于这是一个比较新鲜的事物，喜欢定义的我司给出了Serverless的[概念](https://martinfowler.com/articles/serverless.html)，Serverless原本有两个概念:

1. 描述严重或完全依赖第三方应用程序/服务（比如在云平台）管理服务器端逻辑和状态的应用程序。举个例子，对于Single Page的应用，我们一般都是使用前后端分离的架构，前端部署在AWS S3上面，你可以把S3看成一个文件存储服务，对于前端应用的部署，只是上传新的文件而已，同时S3的服务器对我们来说都是不可见的，我们也不用担心任何的维护压力，也不用担心如何扩展(大多数情况下)。
2. 开发者实现的服务器端的应用逻辑(微服务甚至粒度更小的服务)以event trigger的方式运行在无状态的临时的容器中，并且这些容器、计算资源都是由第三方去管理。你也可以称之为FaaS(函数即服务)，AWS的Lambda服务就是目前最好的一个实现。

所以可以看到核心的点在于

1. 除了代码外全托管，无任何服务器维护、扩展的负担
2. 更小的实现粒度，微服务粒度可以小到一个Restful的endpoint，这里可以小到一个函数

## Serverless的使用场景
---
Serverless的使用场景，目前能看到的有三种:

1. 传统的MVC架构的应用
2. 事件驱动的应用
3. 定时任务
其中我们目前主要在2、3的业务场景下使用Serverless也就是AWS Lambda。而对于1，我们很早就进行了尝试，但是更多的是[纯技术性质](https://github.com/realestate-com-au/sshephalopod)、hackday的spike，并没有在核心应用上去使用。最主要的考虑是它耦合了太多的AWS服务，失去了可移植性，而非不能做。

### MVC架构的应用
---
假设我们有一个MVC架构的单页面应用宠物店，那么当浏览器请求服务器时，它会按照请求，从数据库读取具体的数据，render html，然后返回给浏览器。
![pet store](https://martinfowler.com/articles/serverless/ps.svg)
在这个例子中，我们需要维护宠物店应用的服务器以及数据库服务器，以及其对应代码、schema等，还有如何部署、扩展的问题。但是如果切换到Serverless架构的话，在AWS上看起来会是这样的:

![]()

1. 对宠物店应用做前后端分离，前端部署在S3上面，后端的逻辑拆分到函数级别，部署在Lambda上，状态和数据保存在Dynamodb中(Dynamodb是一个全托管的NoSQL数据库)，API-Gateway可以作为http proxy以及single sign-on验证入口
2. 有请求到来时，首先返回前端的静态页面，然后根据其中请求的API，由API-Gateway经过验证(Oauth或者SAML)后trigger对应的Lambda函数，比如`/search`对应的是`Search Function`。在函数中它会访问Dynamodb，获取数据并返回。

我们在这里面用到的服务`API-Gateway`、`S3`、`Lambda`以及`Dynamodb`都是全托管的服务，他们基本上都可以通过`Cloudformation`实现基础设施即代码以及部署(某些region不能使用cloudformation去管理API-Gateway，S3的部署不需要使用cloudformation，同样`Dynamodb`也不存在部署的问题)，维护的压力很小，水平扩展由这些服务自己实现，比如Dynamodb、以及Lambda，S3可以通过CDN去提升性能。同时我们可以发现，原本单块架构下部署在一起的三层结构，现在被彻底的分开部署了。是不是非常的amazing(Amazoning)。

这样的好处在于:

1. 不需要为一个一直在线的服务器付费，以及维护、升级
2. 按照请求的数量去付费，而不是服务器类型、运行的数量去付费
3. 完全不用去关心水平扩展的问题
4. 部署非常简单，而且速度都比较快

使用这样的架构对于我们这种长期坚持微服务(粒度到一个Restful endpoint一个API)、前后端分离、持续交付等实践的公司来讲是比较容易的，到现在还没有在核心业务上使用主要是个政治问题，也许过几个月CTO觉得这个也不错，可能立马就采用了。
同时，我们可以发现，传统的只做部署或者服务器维护的运维人员在这个架构下已经没有什么位置了……。所以也许DevOps的下一步就真的是NoOps了……。

### 事件驱动的架构(Event-Driven Architecture)
---
事件驱动的架构比如[Serverless Architectures](https://martinfowler.com/articles/serverless.html)提到的这个应用场景，广告服务收集到信息发布到消息系统中，然后通知注册到具体topic下的`click processor`服务，进行处理，保存到数据库。
![event driven](https://martinfowler.com/articles/serverless/cp.svg)
其中，`click processor`服务可能得运行在单独的服务器上，需要一直在线，同时，如果系统中的消息/事件数量突然增多，对于`click processor`的水平扩展，只能是基于容器甚至于服务器级别，虽然这是可以做到的，但是对于整个架构的设计和使用的服务要求比较高，同时，资源浪费也是不可避免的(比如，可以通过ASG去监控`click processor`的服务器，如果它的CPU使用率过高，就启动新的`click processor`服务器去处理，使用容器的方式同理)。
使用无服务架构的方式的话就变成下面这样:

![]()

1. 用户点击广告服务时，其请求通过http proxy写入到Kinesis中
2. 在`click processor`的Lambda函数上可以绑定要处理的Kinesis Stream，同时指定每次处理的batch大小
3. Lambda轮询Kinesis Stream，有新的事件则立即启动`click processor`函数去处理。

这样就不需要一个一直在线的服务去poll消息系统，同样如果需要扩展的话可以通过在`click processor`函数中去invoke其它的`click process`函数来实现。
Lambda也可以用push模型，比如，它可以将事件源设定为S3的某个bucket中文件上传，当有新的文件上传，它会触发该Lambda函数来处理新上传的文件。除了S3之外，SNS、Congnito等服务也可以以push的方式去触发Lambda函数。

## 定时任务(Scheduled Job)
---
定时任务在企业里面是一个很常见的场景，我见过有一些把定时任务和应用放在一个服务器上跑的，恕我直言这种方式不可取，首先它无法自己scale，处理的不好还会导致服务器的性能问题，同时你在部署的时候可能还得考虑它。比如每周一企业都会给用户发一个简报邮件，如果你用定时任务来跑，可能得像下面这样:

![]()

1. Cron服务器上邮件发送cron定时运行，首先从数据库获取要发送的邮箱的list
2. 该cron将邮箱发送给邮件发送服务集群，集群前面的loadbalancer可以将请求平均到每个服务器上
这种方式除了资源的浪费还有水平扩展成本高之外，还有就是得维护额外的cron服务器以及代码。以前的惯例是把所有的cron任务放在专门的一台/多台机器上运行，维护的成本很高，而且服务器出问题，很难定位到原因，即便你把它们的运行日志都上传到中心化服务器中。
如果使用Lambda的话就简单了很多:

![email Function]()

1. 首先`email function`将`CloudWatchCloudWatch Events - Schedule`作为事件源，并选定执行该函数的时间，可以用cron的标准格式去定义
2. 其次在`email function`运行时，它会从数据库中获取列表，从水平扩展的角度考虑，你可以把列表分为n份，在这个函数中去invoke多个(成千上万个)真正的邮件发送函数去处理。实际上要做的事情很简单，像下面这样:

```javascript
    var aws = require('aws-sdk');
    var lambda = new aws.Lambda({
        region: 'ap-southeast-2' 
    });
    #fake the mail list
    var mailers = {mailer1: "someone@gmail.com", mailer2: "somebody@qq.com"};

    console.log("call email sending function");
    lambda.invoke({
        FunctionName: 'arn:aws:lambda:ap-southeast-2:AWS_ACCOUNT_ID:function:email-sending-function',
        Payload: JSON.stringify(mailers) 
            }, 
        function(error, data) {
                if (error) {
                        context.done('error', error);
                        console.log("invoke failed");
                        }
                    if(data.Payload) {
             context.succeed(data.Payload);
             console.log("invoke success");
        }
    });
```
这样前面所说的痛点基本都解决了，没记错的话我们目前使用Lambda最多的场景就是定时任务，比如ETL。

## 关于AWS Lambda
---
目前Lambda只支持三种语言的运行时环境:

1. java
2. nodejs
3. python

其中nodejs和python可以直接部署代码，而java是部署编译后的jar包。虽然看上去支持的语言不多，但实际上如Ruby语言，可以通过JRuby去开发部署，同理还有基于JVM的scala等语言。

Lambda函数运行的容器最多只能保留5分钟，所以一定要保证函数在5分钟内执行完毕。同时不要在上保存状态，状态可以保存在S3或者数据库中。

Lambda的部署方式有三种

1. 如果没有很多外部依赖，直接上传代码即可
2. 如果有外部依赖，可以打包成zip文件上传
3. 将打包后的文件上传到s3上，通过cloudformation去做部署
同时，Lambda支持的`Alias`功能可以很容易让我们实现蓝绿部署。
Lambda的监控比较粗粒度，AWS没有提供它在容器中运行的状况监控，虽然这点没有太大的必要，你也可以通过比较hack的方式，比如用newrelic来实现对它的监控。
出错报警也是需要注意的地方，可以通过创建cloudwatch alarm监控错误数量，出问题时报警，当然对应的你代码得在错误时抛出异常。
Lambda的日志是传到cloudwatch中的，想要传到日志服务器中得需要些额外的步骤。还有一种方式就是将日志作为时间写入到Kinesis Stream中，以统一的方式写入日志服务器，同时可以在检测到很多错误返回时报警。
目前Lambda最大的问题在于测试，单元测试自然是没有问题，但是集成的话会稍微比较麻烦，每个函数都得有一套对应的测试环境，比如关联的数据库表、S3 bucket、Kinesis Stream等，维护的成本比较高。调试也只能部署上去后在线调试，想想都觉得刺激。有了Lambda，线上改代码不是梦。突然间就想起来P神以前和我结对调试问题的时候说的一句话，"whenever I debug, I debug in production"。现在想想还是P神高。当然这些都是`trade off`，看技术层面的选择了。

在有一次的AWS Summit上，有一位台湾的朋友提到说Lambda背后实现的原理主要有两点:

1. 启动艹快的定制化容器,并且可能会有部署你代码的standby的容器
2. 高速网络
实际上我们回头看它的使用场景，用现有的工具，比如Mesos等，我们在本地或许就能实现event driven或者定时任务的场景，因为它们对于时效性的要求不高。但是对于实时的业务场景，需要立刻返回的就不好做了。不过也许等[Unikernel](http://unikernel.org/)成熟了，这点就不再是问题了。

## 总结
---
不说了，得赶紧把我创建的lambda函数删掉，免的下个月账单💥。

## Reference
---
1. [Serverless Architectures](https://martinfowler.com/articles/serverless.html)
2. [AWS Lambda](https://aws.amazon.com/lambda/)
3. [Event Driven Architecture](http://www.slideshare.net/jboner/event-drivenarchitecture-6097206)
