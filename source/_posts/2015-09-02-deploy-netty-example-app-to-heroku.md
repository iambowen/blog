---
layout: post
title: "deploy netty example app to heroku"
description: ""
category:
tags: [heroku,deploy]
---


作为一名java的弱鸡，为了在公共的服务器上重现netty的一个[bug](http://iambowen.github.io/2015/08/25/tracing-and-production-bug-about-netty/)，我也是很努力的自己setup了一个简单的netty的[java](https://github.com/iambowen/netty-example)工程，然后计划
部署在heroku上面，这样在github上提交issue的说服力更强一些。

###过程
----

###准备netty的项目
----

比较简单，就是在netty的代码库中抄一些[例子](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example)就可以了.要注意的有下面几点:

1. 申明对netty的依赖，如下:

```xml
<dependencies>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.0.24.Final</version>
    </dependency>
</dependencies>
```

2. 将依赖的netty jar包copy到target目录下:

```xml
<executions>
    <execution>
        <id>copy-dependencies</id>
        <phase>package</phase>
        <goals><goal>copy-dependencies</goal></goals>
    </execution>
</executions>
```
3. 声明应用启动时的main class:

```xml
<configuration>
    <archive>
        <manifest>
            <mainClass>com.tw.httpserver.HttpHelloWorldServer</mainClass>
        </manifest>
    </archive>
</configuration>
```
4. app的启动端口需要去读取heroku预设的`PORT`随机端口的环境变量，否则前面的代理服务器无法绑定到后端的netty.
我个人觉得这是一个非常坑爹的设定，同时在官方的文档中没有特别说明。很容易引发下面的错误:

```
Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
```
解决的办法就是这样

```java
static final int PORT = Integer.parseInt(System.getenv("PORT"));
```
5. 声明启动web服务所需命令,在`Procfile`中加入以下内容：

```bash
web: java  $JAVA_OPTS -cp target/classes:target/dependency/* com.tw.httpserver.HttpHelloWorldServer
```

6. 声明系统运行的环境依赖，在 `system.properties`加入:

```
java.runtime.version=1.7
```

### 本地测试
-----

用`mvn clean install`打包到`target目录下`，然后运行`heroku local web`,测试应用在本地是否工作正常。

### 发布和部署
-----

1. 用`heroku create`命令新建一个应用。
2. 本地提交，并将代码提交到heroku，`git push heroku master`。
3. `heroku ps:scale web=1`保证至少有一个实例在运行这个应用。
4. `heroku open`可以打开部署后的服务页面。

### 结论
----
可能是因为java水平太差，感觉在heroku上部署java应用比部署ruby应用要难一些。部署完成后，发现还是不
能重现bug，感觉好受伤。
