---
layout: post
title: "Learning Scala: Some Basics"
description: ""
category:
tags: [scala]
---


简单介绍Scala一些基本的概念，主要参考书籍为高宇翔翻译的[《快学Scala》](http://url6.org/ovN)第一章。

####声明变量:val和var的区别
`val` 是Scala中用来申明常量(常量变量)的关键字，对应的 `var` 是用来申明变量的关键字。当我们知道这个概念的时候，对它们区别应该就比较清晰了。Scala中的常量字面量必须是全大写字母。有例为证:

```bash
#申明常量变量
scala> val a = "hello"
a: String = hello

scala> a = "hello, world"
<console>:8: error: reassignment to val
       a = "hello, world"
         ^
#声明变量
scala> var b = "hello"
b: String = hello

#声明常量字面量
scala> b = "hello, world"
b: String = hello, world
```

```bash
scala> val A = "hello"
A: String = hello
```

Scala会根据初始化的值，自动推断出常量/变量的类型，这个叫做[“类型推断”](http://scala.wisdomfish.org/scala-research/type-inference)，能想到的类似的概念是Ruby的duck typing，不过Scala是静态语言，并且类型安全(TypeSafe)。
当然你也可以显式的指明变量的类型，用过pascal的同学应该很熟悉这种方式:

```bash
scala> val c:String = "hello, world"
c: String = hello, world
scala> val c:String = null
c: String = null
```
Scala支持同时为多个变量赋同一个值，或者像Ruby一样并行赋值，只不过语法略有不同，:

```bash
scala> val d, e = "hello"
d: String = hello
e: String = hello
#类型不匹配会报错
scala> val d, e:Int = "hello"
<console>:7: error: type mismatch;
 found   : String("hello")
 required: Int
       val d, e:Int = "hello"
                      ^
```
并行赋值的例子，实际上是模式匹配:

```bash
scala> val (a: String, b: Int) = ("hello", 100)
a: String = hello
b: Int = 100
```
对比下Ruby的代码:

```bash
irb(main):004:0> a, b = "hello", 100
=> ["hello", 100]
irb(main):005:0> a
=> "hello"
irb(main):006:0> b
=> 100
```

####常用类型
和Java一样，Scala有7种基本的数值类型(Byte, Char, Short, Double, Int, Long, Boolean)，不同之处在于它们不是基本类型(Primary Type)，全部都是Class。

```bash
scala> 1.getClass
res4: Class[Int] = int
scala> "hello".getClass
res1: Class[_ <: String] = class java.lang.String
scala> 1.5.getClass
res2: Class[Double] = double
scala> 1.to(5)
res6: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5)
```
Scala对这些数值类型做了扩展，功能远比Java的对应类型强大，比如在处理String对象的时候，Scala可以将String对象隐式转换成StringOps对象，使用一些高级的方法：

```bash
scala> "hello".intersect("world")
res3: String = lo
```
Scala中用方法进行类型转换，而不是和Java一样去强制类型转换。

```bash
scala> 1.1.toInt
res7: Int = 1
scala> 107.toChar
res8: Char = k
```
####算术符和操作符重载
就结果和使用来说，Scala中的算术符(+-*/%&|^>><<)和Java/c/c++/ruby没什么区别，不过Scala的算术/操作符本质上是方法，刚开始觉得奇怪，后来想想这样做可以保证一致性，也很不错：

```bash
scala> 1 + 2
res9: Int = 3

scala> 1.+(2)
res10: Double = 3.0
```
Scala也没有提供++或者--操作符，+=和-=是等价的代替方式。为什么没有呢？书中给出的理由是实现这个特性不值得……不能理解，是觉得会像c++或者Java一样带来副作用么？

```bash
scala> var b = 1
b: Int = 1

scala> b += 1

scala> b
res20: Int = 2
```
在Scala中，操作符重载功能没有实现，给出的理由比较牵强，不明白为什么那么想。
####定义和使用方法
Scala中用`def`关键字定义方法，可以显示的制定输入参数以及返回的类型。

```java
def max(a: Int, b: Int): Int = {
  if (a > b) a
  else b
}
max: (a: Int, b: Int)Int
```
调用方法

```java
scala> max(1, 2)
res0: Int = 2
```
##定义和使用类

```java
class Hello {
  val str = "hello, world"
}
object Hello {
  def world = println("hello, world!")
}
```

类和伴生类必须同时定义，所以，可以进入`paste`模式，将代码拷贝到Scala的解释器中。

```bash
scala> val a = new Hello
a: Hello = Hello@62150f9e

scala> a.str
res5: String = hello, world

scala> Hello.world
hello, world!
```
能想到的(其实是读过的……)基础就是这些了，下一章介绍控制结构好了。
