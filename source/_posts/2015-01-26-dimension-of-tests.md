---
layout: post
title: "Dimensions of Test"
description: "tests"
category: "tests"
tags: ["test"]
---

前几周给一个客户介绍了关于[持续交付]()的内容，他们特别关注概念以及细节，当时我在介绍自动化测试的时候，
分类比较简单，感觉回答的不是很好，于是重新整理下思路，总结成文。

### 软件测试的定义
----
对于软件测试的定义不尽相同，其经典定义是:在规定的条件下对程序进行操作，以发现程序错误，衡量软件质量，并对其是否能满足设计要求进行评估的过程。

### 软件测试的分类
----

测试的分类很多，可以按照不同的角度去划分，比如

1. 功能/非功能测试
2. 静态/动态测试(是否执行程序)
3. 白盒/黑盒/灰盒测试(是否关心软件内部结构和具体实现)
4. 按照软件开发过程划分

我自己更倾向于按照软件开发的过程去划分，同时，将测试的范围扩大到软件的整个生命周期，不止是开发测试，还要包括部署、产品的监控以及对基础架构的测试。如此，则有以下(不限于)分类:

- Unit Testing
- Integration Testing
- Functional Testing
- Smoke Testing
- E2E Testing
- Regression Testing
- Acceptance Testing
- Exploratory Testing
- Performance Testing
- Infrastructure Testing

以上除接受测试和探索性测试之外，都应当是自动化的。

### Test Pyramid(软件测试金字塔)
----
测试金字塔是Mike Cohn提出的概念，意思是测试从上到下-功能性测试到单元测试，数量由少到多，呈一个金字塔形。

![pyramid](http://martinfowler.com/bliki/images/testPyramid/test-pyramid.png)

上图引自Martin Fowler的测试金字塔的文章，其实不用他说，写过测试的都知道，单元测试的成本最低，运行
速度最快，但是到功能性测试，需要启动UI/浏览器去测试，成本很高，所以单元测试可以多考虑edge case，而
功能性测试只考虑happy path。曾经参与过一个项目，基本上严格执行BDD的开发方式，用[Cucumber](http://cukes.info/)写了大量的测试，
但是每个产品的功能测试，运行完成都会超过一个小时，最后只能通过切分测试，多加几个Jenkins Slave去并行运行，才将构建的时间降到半个小时左右。

### Unit Testing(单元测试)
----
对于每个开发人员来说，对于单元测试来说应该都很熟悉。在我看来，写单元测试的好处在于:

1. 可重复自动运行，验证成本很小
2. 代码为文档，测试可以帮助理解程序或者业务逻辑
3. 单元测试一般粒度较小，因此发现问题，定位会比较快

当然，也有人反对单元测试，认为大多数的单元测试都是废柴，比如[这位](http://www.rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf),可能很多认为在代码或者逻辑不太改变的情况下，单元测试在运行之后就没有什么太大的用处，但是一旦代码有任何调整，有单元测试是你重构的信心来源。
单元测试做到什么粒度，也是有争议的，我司曾经提出过功能性单元测试(Functional Level Unit Testing)，可以认为是单元测试粒度放大，针对具体的功能。
单元测试的覆盖率应该是多少，也有争议，我当时的项目做到了后端代码100%测试覆盖率，js代码做到了70%-90%的测试覆盖率。有些人认为只针对关键功能，比如算法写单元测试就可以了，80%
就是很不错得测试覆盖率了。不过，上次苹果的一个安全漏洞，就是一段很简单的代码，如果有一个test case覆盖到的话，应该就不会出问题了。
单元测试的代码往往要多于产品代码，有时候维护也是个问题。

### TDD(测试驱动开发)
----

提到单元测试就不得不说一下TDD测试驱动开发了，测试驱动开发就是在写实现代码之前先写测试，遵循下面的步骤:
1. 写测试，运行，失败(红)
2. 实现代码，运行测试,成功(绿)
3. 重构，运行测试，成功(绿)

这也就是TDD的两顶帽子，功能实现和重构，每次只能戴上其中一顶。
上面的描述缺少了TDD实践中一个很重要的环节，tasking，也就是将story分解成粒度更小的test case，然后划定实现的步骤，小步快跑。

从实际的项目中来看，TDD对于理清需求是有帮助的，有助于业务实现的正确性，保证了测试覆盖率，而且一旦建立
这种节奏，稳步向前带来的自信感非常好。
目前TDD的使用在国内外也是有争议的，我觉得很大程度上和敏捷社区如何推动这个技术实践的态度有关系，过于
强势，“不做TDD就怎么怎么了”，很容易让人反感，说实话我也很反感这样。用不用TDD和你的业务成功与否没有
直接的关联，加之TDD实际上降低了对程序员的要求(责任感>能力)，所以切实实践过或者是大神们认为它有缺点
是很正常的一件事情，不过水平一般且没有实际的体会就不要附和了。

去年这个话题炒得很热，一定程度上因为[DHH](http://david.heinemeierhansson.com/)(Rails框架的创始人)的一篇文章[TDD is Dead](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html),之后他和Uncle Bob以及我司的Martin Fowler展开了一系列的[论战](http://martinfowler.com/articles/is-tdd-dead/)。这种论战过程有趣，结果没什么意思，因为大神都比较顽固，不容易放弃自己的想法，不过不论他们怎么说，"Long Live The Testing"，这点是要承认的。

我个人对于TDD，觉得还是要宽容些，保持责任心和实现对功能的测试更加重要，不用纠结于是不是T-DD出来的代码。

### Integration Testing(集成测试)
----
集成测试就是各个模块组装进行测试，比如A依赖B模块，作为单元测试可能我只需要Mock或者Stub，但是集成测试会输入合适数据，真实的让A去调用B服务，测试是否会满足期望的结果。

现在微服务的这种架构方式很流行，一个简单的例子就是前后端分离，前端与不同的微服务交互，如此部署，维护，
扩展都会更加容易。在这样的架构下，集成测试是一个问题，[Pact Testing](https://github.com/realestate-com-au/pact)是一个可选的方式。Pact是基于[CDC - Consumer driven contracts](http://martinfowler.com/articles/consumerDrivenContracts.html)的一个实现。

我们可以将前端理解为Consumer，后端理解为Provider，完整的测试过程如下:

1. Consumer提出要请求的Provider的API格式，传入的数据以及期望的返回，运行测试后生成pact(json)文件并将其push到Provider的repo中
2. Provider CI运行Pact测试，因为API还没有实现，测试会失败
3. Provider的开发人员根据Pact文件中约定的Restful API以及Input/Output数据，实现并且通过测试。

目前Pact已经有多种语言的实现，比如[ruby](https://github.com/realestate-com-au/pact), [Java/Scala](https://github.com/DiUS/pact-jvm)等。

### Functional Testing
----
功能测试，就是对产品的功能进行验证。对于web项目来说，就是启动服务，通过浏览器访问测试。曾经做过的Ruby
项目是使用cucumber去进行测试，其实依赖的工具是Capybara和Webdriver/Selenium。讲到这里就不得不提
[BDD - Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior-driven_development)了

#### BDD
---
BDD更关注从业务的角度出发，用[Business Readable DSL](http://www.martinfowler.com/bliki/BusinessReadableDSL.html)的方式去描述一种实例行为，期望应用可以满足预期。
[Cucumber](http://cukes.info/)是其一种实现，其测试的基本的格式如下:

``` cucumber
Given I am a super user
When I log in the website
Then I should see "Admin" on the top right
```
通过在每个step中写一些Assertion，可以判断功能是否实现等。
我的第一个项目是严格遵循BDD的流程，基本顺序如下:

1. 在接到一个需求时，按照描述，先写Cucumber测试
2. 运行测试，失败，然后编写对应的单元测试
3. 运行单元测试用例失败，实现之并让其通过
4. 运行Cucumber测试，通过
5. 重复1-4直至完全满足需求

做的好的话，整个开发的节奏会非常流畅。BDD原本期望是让BA或者产品经理去读或者写测试描述，但是很少有BA
或者产品经理回去阅读这些测试，写就更不可能了。程序员自己写的话，表达很难贴近正确的业务行为，加上不正确的
添加测试(通常Cucumber测试只检查Happy Path)，导致测试越来越多，运行越来越慢，所有测试跑完的时间，远超过
我seed一些测试数据，启动浏览器手工测试快些。
以前的团队有很好的BA，在Story的Acceptance Criteria中，会将用例写的很清晰，这样基本上不用怎么修改就可以
写好Cucumber测试的step，现在倡导全功能团队，BA和QA这两个Role几乎都没有了，客户也不怎么提BDD了……。

### Smoke Testing(冒烟测试)
----
冒烟测试是确认软件的基本功能是否正常的测试。

### E2E Testing(端到端测试)
----
端到端测试，可以理解为整个系统在现实世界中的走完一个完整流程的测试。领域不同，端到端测试的概念不尽相同。
我们当时的端到端测试只cover了与其他系统的接入点功能的验证以及基本的流程的验证。因为要访问不同的服务，
而这些服务的可用性不一定很高，同时在运行测试时需要保证其他的系统都使用了合适的版本，当时的E2E测试经常
会出问题，修复也很麻烦，一定程度上得怪亚马逊EC2机器性能或者网络的问题-_-!

### Regression Testing(回归测试)
----
回归测试，确保新加入的功能没有破坏已有的功能。讲到这里就得介绍下，在持续交付的前提下，我们通常用两种方式:

1. Feature Toggle
2. Feature Branch

用来保证实现新功能同时不影响旧功能。

Feature Toggle译为特性开关，就是通过配置文件/数据库来判断是否使用新特性的代码，其实就是读入配置文件，然后用if/else去判断用新或者是旧的代码。
这个的好处在于你可以只在主干上开发，保证不破坏现有的功能，同时还能保证持续交付/部署。要付出的代价就是写针对toggle打开或者关闭情况下，
两种状态不同功能的测试，如果有两个toggle，那么可能得针对四种状态写测试。在回归测试的时候会关闭所有Toggle，对应的还有Progression测试，
既打开enable所有toggle，新功能的测试应该都通过。如此测试的数量会大大增加，流水线构建的速度也会随之减慢。

Feature Branch就是在新的Branch上开发新功能，坏处就是与主干merge太晚，可能会出现merge hell，很难一下修复太多冲突。
相比之下Feature Toggle要更好些，但是注意最好不要加太多Toggle。

### Acceptance Testing(接收测试)
----
就是开发或者测试将实现好的功能，在测试环境中，给business people做show case，然后证明所有的需求都正确实现了。只有他们说OK，story卡才能认为是done了。

### Exploratory Testing(探索性测试)
----
新产品上线前，找一群对产品完全不了解的人，让它们试用，以期发现一些诡异的bug或者使用不合理的地方。做
过一次，还真的发现了bug。

### Performance Testing(性能测试)
----
dark release 的时候，针对产品环境，使用典型的应用场景，进行性能测试，比如并发数等等。常用的工具有
`ab`,[Jmeter](http://jmeter.apache.org/), [Gattling](http://gatling.io/)等。Jmeter
感觉配置比较麻烦，Gattling是基于Scala的，用起来还不错。

### Infrastructure Test(面向基础架构的测试)
----
面向基础架构的测试是近年来提出的一种概念，应该上过我司的[Tech Radar](www.thoughtworks.com/radar/techniques)。
随着DevOps运动的兴起，"Infrastructure as Code"概念深入人心, 既然“基础架构即代码”，那么针对
这些代码，我们也可以做基本的单元测试等。比如[puppet](http://puppetlabs.com/) , [chef](https://www.chef.io/)。甚至有[Babushka](http://babushka.me/)这样的为sysadmin准备的TDD工具。
除了开发环节，对线上的产品环境也做测试, 比如:

1. [Chaos Monkey](https://github.com/Netflix/SimianArmy)是NetFlix推出的帮助测试系统
稳定性的工具，它会随机的干掉产品环境在AWS上一些节点，如果系统的弹性很好，那么它会自动恢复这些节点。
这是一种防御性的测试，同时通过测试的反馈，逐步的改进产品的稳定性和弹性。
2. [Nagios](http://www.nagios.org/)是一个监控工具，它可以测试产品环境的应用的健康状况以及服务器的监控状况。
3. 用类似[Gomez](http://gomeznetworks.com/)的工具去检测应用的可用性状况。
4. 针对第三方的平台，如CDN，通常都是得手工配置，可以通过写单独的测试来验证配置正确，如CDN服务商提供的一些特殊的Header或者返回码。

### 那么问题来了
----

如果产品环境出现一个bug，该如何处理？下面是我觉得一个不错的处理方式。

1. 定位问题
2. 根据bug的场景，写一个失败的测试
3. 修复这个测试
4. 打包/部署测试环境/接受测试/部署产品环境

### 结论
----

场景不同，测试的方式方案不尽相同，但如果能保证最大程度的自动化，那将是极好的。

### 免责申明
----

本人不是测试专精，如果有误导或者描述不准确的地方，请轻喷。

### References
----
- [TestPyramid](http://martinfowler.com/bliki/TestPyramid.html)
