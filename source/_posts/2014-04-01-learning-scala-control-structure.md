---
layout: post
title: "Learning Scala: Control Structure"
description: ""
category: scala
tags: [scala]
---


任何语言的控制结构应当都是相似的，语言处理现实世界的问题，理应和现实世界对应。Scala的控制结构以及关键字与Java没什么不同，有区别的地方在于Scala中表达式可以有返回值，比如条件表达式。

###条件表达式(if/else)
Scala中典型的条件表达式如下:

```scala
scala> val x = 1
x: Int = 1
scala> if (x > 0) 1 else -1
res0: Int = 1

scala> val y = -1
y: Int = -1
scala> if (y > 0) 1
res1: AnyVal = ()
```
需要注意的有如下几点:
1. Scala中，表达式不需要分号表示结束;
2. Scala中没有?:三元表达式;
3. 条件表达式都有返回值;
4. 返回值的超类是Any，if语句没有输出值时，返回值为Unit;
5. 表达式过长或者，可以将其放在大括号中，如下:

```scala
if (n > 0) {
  n = n * n
  n -= 1
}
```

###循环(for/while)
Scala中的*for/while*循环和Java中没什么太大区别，但是Scala中没有类似Java中*for(initialize; condition; update variable)*的结构。

for循环的例子:

```scala
for(i <- 1 to 10)
  println(i)

for(c <- "hello, world")  
  println(c)
```
*变量<-表达式* 这样的语法被称作生成器， 变量i/c为集合元素类型，循环用其依次获得集合中的元素，并处理。

while的例子:

```scala
var a = 10
var b = 1
while (a > 0) {
  b *= a
  a -= 1
}
```
Scala中没有*break*或者*continue*关键字，如果要中途退出循环，需要自己设置Boolean型变量或者使用Breaks对象的break方法。

###for推导
for推导是包含卫语句(*guard clause*)的for表达式，比如:

```scala
for (i <- 1 to 3; j <- 1 to 3 if i != j) println((10 * i + j) + " ")
```
如果i 等于 j，表达式不会打印结果。





###模式匹配
个人认为Scala最好用的特性之一，等理解透彻了专门写一篇，下面只是个简单的例子，可以认为是可以用来匹配类对象的*switch*。

```scala
val a = true

a match {
  case true => println("yes, PPG!")
  case _ => println("failed!")
}
```
