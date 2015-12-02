---
layout: post
title: "Math in shell"
description: ""
category: "linux"
tags: ["shell"]
---


最近半年的所作的项目中，接触了些Ops的工作，自动化工具如Chef/Puppet都用过些。这些工具都很不错，但是过程当中会遇到一些问题，需要你基础的系统/shell知识，从而认识到了其重要性，开始系统学习，目前阅读的书为: <linux shell scripting cookbook version> 以及浏览china-unix论坛。

Shell的强大是毋庸置疑的，所以进行数学运算也就是理所应当具有的功能。Shell可以用`declare/let/(( ))/[]`进行一些简单的数学运算。
Shell 支持的运算:

```bash
ARITHMETIC EVALUATION
       The  shell  allows arithmetic expressions to be evaluated, under certain circumstances (see the let and declare builtin commands and Arithmetic Expansion).
       Evaluation is done in fixed-width integers with no check for overflow, though division by 0 is trapped and flagged as an error.  The  operators  and  their
       precedence, associativity, and values are the same as in the C language.  The following list of operators is grouped into levels of equal-precedence opera-
       tors.  The levels are listed in order of decreasing precedence.

       id++ id--
              variable post-increment and post-decrement
       ++id --id
              variable pre-increment and pre-decrement
       - +    unary minus and plus
       ! ~    logical and bitwise negation
       **     exponentiation
       * / %  multiplication, division, remainder
       + -    addition, subtraction
       << >>  left and right bitwise shifts
       <= >= < >
              comparison
       == !=  equality and inequality
       &      bitwise AND
       ^      bitwise exclusive OR
       |      bitwise OR
       &&     logical AND
       ||     logical OR
       expr?expr:expr
              conditional operator
       = *= /= %= += -= <<= >>= &= ^= |=
              assignment
       expr1 , expr2
              comma
```

可以看到涵盖了通用的编程语言中的数学运算方式，那么在shell中可以通过那些命令来进行计算呢？

`declare`: 声明变量同时定义其属性。

```bash
#!/bin/bash
n=1*2*9/3
echo $n
declare -i n
n=1*2*9/3
echo $n

result:
1*2*9/3

6
```

从上面的例子可以看出，只有将变量声明为整数，shell才会认为你是在尝试进行数学运算而不是单纯的字符串变量。

`expr`: 数学表达式运算

```bash
z=`expr 5 + 1`
echo $z
a=`expr 7 % 2`
echo $a
d=`expr 1 \& 0`
echo $d
#error
b=`expr 7*2`
echo $b

result:
6
1
0
7*2  
```

expr后所带的表达式运算符必须以空格分开，不然会被当做字符串，不会进行运算，一些meta字符比如& %等，必须转义才可以执行。

`let`:  后面跟随数学表达式并进行运算

```bash
[vagrant@localhost shell]$ a=2
[vagrant@localhost shell]$ b=2
[vagrant@localhost shell]$ let result=a+b
[vagrant@localhost shell]$ echo $result
4
[vagrant@localhost shell]$ let result++
[vagrant@localhost shell]$ echo $result
5
[vagrant@localhost shell]$ let result--
[vagrant@localhost shell]$ echo $result
4
```

同样，在计算的时候需要用空格分隔。

`(( ))` 和`[]`也可以起到类似的作用。参加如下的例子:

```bash
[vagrant@localhost shell]$ a=1
[vagrant@localhost shell]$ b=2
[vagrant@localhost shell]$ result=$[ a + b ]
[vagrant@localhost shell]$ echo $result
3


[vagrant@localhost shell]$ result=$[ $a + b ]
[vagrant@localhost shell]$ echo $result
3


[vagrant@localhost shell]$ result=$(($a + 50))
[vagrant@localhost shell]$ echo $result
51

[vagrant@localhost shell]$ result=$(($a - 50))
[vagrant@localhost shell]$ echo $result
-49

[vagrant@localhost shell]$ echo $((16#2a))  #进制转换,将16进制的”2a”转换为十进制的数字, 相当酷帅吊炸天
42  

[vagrant@localhost shell]$ echo $((2#1111))
15
```

`(()) `或者`[] `中的变量也可以加或者不加$符号从表达式外获取。以上这些命令只能处理整数运算，如果想要进行浮点数运算，就得求助`bc`命令了。

```bash
[vagrant@localhost shell]$ echo "5 * 0.86" | bc
4.30

[vagrant@localhost ~]$ n=40
[vagrant@localhost ~]$  echo "$n * 0.86" | bc
34.40

[vagrant@localhost ~]$ echo "scale=3;4/7" | bc   #指定十进制浮点运算的精度
.571

[vagrant@localhost ~]$ umask
0002
[vagrant@localhost ~]$ echo "obase=8;$(( 8#666 & (8#777 ^ 8#$(umask)) ))" | bc
664

[vagrant@localhost ~]$ no=100
[vagrant@localhost ~]$ echo "obase=2;$no" | bc   #进制转换, 10进制-> 2进制
1100100

[vagrant@localhost ~]$ echo "sqrt(100)" | bc #开方
10

[vagrant@localhost ~]$ echo "10^10" | bc   #次方计算
10000000000
```

`bc`的功能远超上面所列举的例子，想仔细研究的话就 `man bc`…., enjoy。

references:[1](http://www.sal.ksu.edu/faculty/tim/unix_sg/bash/math.html), [2](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=218853&page=7#pid1617953), [3](http://www.amazon.com/Linux-Scripting-Cookbook-Second-Edition/dp/1782162747)
