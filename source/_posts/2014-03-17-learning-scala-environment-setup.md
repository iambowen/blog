---
layout: post
title: "Learning Scala: Prepare Development Environment"
description: "Learning Scala Series"
category:
tags: [scala]
---


最近开始翻译[Scala Cookbook](http://shop.oreilly.com/product/0636920026914.do), 希望能为Scala这门语言的推广做出些贡献。就我自己的学习语言和阅读教材经验来看，教材无法覆盖很多实际的使用过程中碰到的问题，比如环境问题等。于是想写一些列总结性blog，也算是自己的总结了。

##Scala的安装:
安装前先确保机器上已经有java的环境，因为Scala同样需要JVM虚拟机。
[Scala](http://www.scala-lang.org/)最新的版本是2.10.x, 安装的方法很简单，下载解压安装包，把`scala/bin`目录加入到`PATH`环境变量中即可，如下的脚本在OSX和*nux下适用。

```bash
link="http://www.scala-lang.org/files/archive/scala-2.10.3.tgz"
curl -O $link && tar xvzf scala-2.10.3.tgz ~/ && echo "export PATH=~/scala/bin/:$PATH"  >> ~/.bash_profile && source ~/.bash_profile
```

在命令行敲入`scala`就可以进入Scala的console(解释器)了。

```bash
 ~> scala
Welcome to Scala version 2.10.3 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_51).
Type in expressions to have them evaluated.
Type :help for more information.
scala>
```

还有一种办法是安装[SBT](http://www.scala-sbt.org/), Scala工程的构建工具，OSX下直接用[Homebrew](http://brew.sh/)就可以安装.
`brew install sbt`
可能需要修改java虚拟机的启动参数，否则可能出错:

```bash
echo "export JAVA_OPTS=\"-XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:PermSize=256M -XX:MaxPermSize=512M\"" >> ~/.bash_profile && source ~/.bash_profile
```

用`sbt console`进入Scala的REPL console。

```bash
 ~> sbt console
[info] Starting scala interpreter...
[info]
Welcome to Scala version 2.10.3 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_51).
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

##Scala REPL 的使用

`:help`可以告诉你想要的关于Scala REPL的一切信息，个人觉得几个比较有用的命令有`:paste`, `:load`, `:sh`, `:history`等。
`:paste`,直接在Scala console里面编辑代码相对来说不是很方便，你可以选择自己熟悉的编辑器，完成代码，使用`:paste`命令，`Command + V`将代码拷贝到Scala REPL中，之后按`Ctrl + D`退出，对应的代码就被load到当前的进程当中了。

```bash
scala> :paste
// Entering paste mode (ctrl-D to finish)

def hello() = println("hello, world")

// Exiting paste mode, now interpreting.

hello: ()Unit

scala> hello
hello, world
```
在Scala console中也可以执行shell的命令，结果会被保存到一个list中，如下

```bash
scala> :sh ls
res15: scala.tools.nsc.interpreter.ProcessResult = `ls` (6 lines, exit 0)

scala> res15.toArray
res16: Array[String] = Array(README.md, cn, code_examples, conventions.yml, en, target)
```

用`:load`命令+文件路径，可以加载Scala文件，导入你自定义的类/代码。

```bash
scala> :load ./code_examples/chap1/hello.scala
Loading ./code_examples/chap1/hello.scala...
hello, world

scala>
```
Scala的console支持和shell类似的history功能，你可以用`Ctrl + R`去搜寻前面执行过的代码再来一次，用`:history`查看所有的曾经运行过的代码。比较尴尬的是，虽然SBT文档里面注明是可以用shebang的方式去执行历史代码，我在REPL中却没有试验成功。Anyway，这个不是特别重要了。
还有些命令诸如`:implicits`，显示当前scope中的隐式定义，`:imports`可以显示导入类库的记录。
我好像没有说明console的最重要的作用。。。，那就是解释scala代码并返回结果，这个时候的Scala代码就像脚本一样，支持自动补齐，不过功能不是很强大。

```bash
scala> val x = 1 + 2
x: Int = 3
scala> "hello " * 5   #如果没有显式的赋值，scala解释器会给把结果指定一个变量名
res1: String = "hello hello hello hello hello "
scala> println(res1)
hello hello hello hello hello
```

##编译运行一个Scala文件
用`scalac`和`scala`命令，和使用`javac`和`java`命令很像，都会生成class文件。

```scala
/*helloworld, example, helloworld.scala*/
object HelloWorld extends App {
   println("Hello, World");
}
```
用`scalac`命令编译

```bash
 ~> scalac helloworld.scala
```
用`scala`命令运行

```bash
~> scala HelloWorld
Hello, World
```

##Scala IDE
Intelij, NetBeads和Eclipse都提供了Scala支持，看个人喜欢，可以随意选择。Netbeans的Scala插件是由国内的一个程序员邓草原开发的。当然，你也可以考虑其他的工具比如Emacs，Vim，Sublime或者Atom。

* [eclipse](http://scala-ide.org/)
* [intelij + Scala plugin](http://www.jetbrains.com/idea/download/)
* [netbeans scala](https://github.com/dcaoyuan/nbscala  )
* [VIM with Scala](https://github.com/derekwyatt/vim-scala )
* [Sublime with scala](https://github.com/sublimescala/sublime-ensime)

关于开发环境，基本就是这些了。
