---
layout: post
title: "Learning Scala: Function"
description: "scala"
category: "scala"
tags: [scala]
---
函数在Scala中是一等公民，这意味着你可以在任何时候定义函数，而且它不需要依附于类，在Class/Object中定义的函数叫做方法。

###语法
函数由函数名称，参数以及函数体组成，未声明函数返回类型的，默认返回Unit。

```java
scala> def printName(name: String) = println("Hello, " + name)
func: (name: String)Unit

scala> def double(num: Int):Int = num * 2
double: (num: Int)Int


scala> def iter(num: Int) = { if(num == 0) 1 else 1 + iter(num -1) }
<console>:7: error: recursive method iter needs result type
       def iter(num: Int) = { if(num == 0) 1 else 1 + iter(num -1) }
                                                      ^
```
对于递归函数，必须指定返回类型，否则编译无法通过。

###默认参数和带名参数
可以给定函数的参数一个默认的值，调用函数时可以不用传入该参数。

```java
scala> def printName(congrats:String, name: String = "default") = println(congrats + " " + name)
printName: (name: String, congrats: String)Unit

scala> printName("hello")
hello default
```
不过，如果函数有多个参数并且默认参数是第一个，只传入一个参数时，编译无法通过，应该是没有特别指明，直接被当成了第一个参数。解决的办法是通过带名参数。

```java
scala> def printName(name: String = "default", congrats:String) = println(congrats + " " + name)
printName: (name: String, congrats: String)Unit

scala> printName(congrats = "hello")
hello default
```
由此我们可以看出，带名参数不需要和参数列表的顺序相同。


###call by name / call by value
Scala中有两种参数替换模型，`call by name`和`call by value`，直译就是传名或者传值。对于下面的函数

```java
def f(x:Int, y:Int) = { x }
```
如果是传值的方式，那么 f(2, 2 * 2) 首先会被转换为f(2, 4),也就是说在函数体中未调用之前就已经计算好了。如果是传名的方式，那么2 * 2始终不会被计算，除非该参数被调用。Scala中支持两种传参方式，上面的例子中实际上是`call by name`。`call by value`是用下面的方式声明函数。

```java
def g(y: int) = { 2 * y}
def f(x:Int, y:Int => Int) = { x }
```
只要传入于参数一致的签名的函数即可。


###变长参数
和其他编程语言一样，Scala也可以接受变长参数。函数会将变长参数转换参数序列。

```java
scala> def sum(args: Int*) = {
        var result = 0
        for (arg <- args) result += arg
        result
      }
sum: (args: Int*)Int

scala> sum(1 ,2, 3, 4, 5)
res4: Int = 15
```
但是直接传入Seq会出错……，因为类型不匹配。

```java
scala> val s = sum(1 to 5)


scala> sum( 1 to 5)
<console>:9: error: type mismatch;
 found   : scala.collection.immutable.Range.Inclusive
 required: Int
              sum( 1 to 5)
                     ^
```
所以还是需要转换称参数序列。

```java
scala> sum(1 to 5: _*)
res1: Int = 15
```

###匿名函数
举个简单的栗子好了。

```java
scala> List(1, 2, 3).map {(x:Int) => (x * 2)}
res4: List[Int] = List(2, 4, 6)
```

###过程

过程是返回Unit的函数，利用的是函数的副作用。从语法上来说，和函数的区别就在于它没有等号。

```java
def process(job: String)  { run; println("running job" + job) }
```

当然也可以加上等号，显示的申明返回类型为Unit。

```java
def process(job: String):Unit =  { run; println("running job" + job) }
```
