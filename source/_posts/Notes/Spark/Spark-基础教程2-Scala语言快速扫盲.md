---
title: Spark-基础教程2-Scala语言快速扫盲
tags:
  - spark
categories:
  - Notes
date: 2015-06-05 11:24:06
---
## 任性的教授：Martin Odersky
Spark是用Scala（发音为 /ˈskɑːlə, ˈskeɪlə/）语言开发的。Scala是一种多范式的编程语言，设计意图是要集成 面向对象编程和函数式编程的各种特性。由于不满Java语言复杂的语法，瑞士洛桑联邦理工学院奥德斯基教授带领小组在2001年创建 了Scala语言（任性~）。

scala运行在Java虚拟机之上，也就是说scala会被编译为和java编译后的class一样的字节码。 这也代表scala和java是可以互相调用并且它们可以联合编译，不过实际上来说scala调用java很容易，而java调用 scala有时会遇到一些问题。

## 环境准备

### 启动Hadoop和Spark

Spark环境中输入：

``` bash
cd /usr/local/spark
/bin/spark-shell
```

### 启动scala环境

## 定义变量与常量

在Scala中定义一个变量，使用`var`关键字：

``` scala
scala> var name = "steve"
name: java.lang.String = steve
```
``` scala
scala> name = "marius"
name: java.lang.String = marius
```

如果要定义一个常量，使用`val`关键字，常量在首次赋值后不可以改变：

``` scala
scala> val two = 1 + 1
two: Int = 2
```

请定义变量 a ， 值为 123 ; 常量 b ， 值为 “demo” 。

## 函数定义与调用

在Scala中，使用`def`关键字定义一个函数，等号左侧是函数名、参数列表和返回值，右侧是函数体实现的表达式。

下面定义的函数对传入整型参数加1并返回：

``` scala
scala> val two = 1 + 1
two: Int = 2
```

使用函数名，并传入参数进行函数调用：

``` scala
scala > var x = addOne(123)
x : 124
```

如果函数体需要多个表达式才能实现，可以使用`代码块`将多个表达式包起来：

``` scala
scala > def sum(n:Int):Int = {
           for (i <- 1 to 10)
           r = r*i
           r // return r 也可以
        }
```

如果在函数体中不使用`return`返回函数值，那么`最后一个表达式的值就是函数返回值`。

请定义函数 multiInt ， 有两个Int型参数，返回这两个参数的乘积。

## 超级重要的匿名函数

Scala支持匿名函数，创建匿名函数不需要使用`def`关键字，符号`=>`的左侧定义 参数列表，右侧定义函数体实现。

下面的匿名函数为名为x的变量加1并返回结果：
``` scala
scala> (x: Int) => x + 1
res2: (Int) => Int = ...
```

函数可以`赋值`给变量或常量，这是`函数式编程`的一个特点。下面将匿名函数赋给addOne常量，后续代码中就可以 使用addOne进行调用了：

``` scala
scala> val addOne = (x: Int) => x + 1
addOne: (Int) => Int = ...
```
``` scala
scala> addOne(1)
res4: Int = 2
```
如果匿名函数体实现包含多行表达式，可以使用`{}`来包围代码，例如：

``` scala
scala> { i: Int =>
         println("hello world")
         i * 2
       }

res0: (Int) => Int = ...
```
请定义匿名函数，实现两个Int型参数相乘的功能，并将该函数赋值给变量 `myFunc`。

## 对象定义

Scala支持面向对象编程，使用`class`定义一个类，在类定义中使用`val`定义成员变量，用`def`定义成员方法：

``` scala
scala> class Calculator {
          val brand: String = "HP"
          def add(m: Int, n: Int): Int = m + n
       }

defined class Calculator
```

使用`new`关键字创建一个对象，使用.调用对象的属性和方法：

``` scala
scala> val calc = new Calculator
calc: Calculator = Calculator@e75a11

scala> calc.add(1, 2)
res1: Int = 3

scala> calc.brand
res2: String = "HP"
```

请定义类 Person ，有两个属性 name 和 age ， 分别为String类型和Int类型；一个方法grow，当 grow被调用时，将age加1。