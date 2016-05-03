---
title: Spark-基础教程4-RDD动作Action
tags:
  - spark
categories:
  - Notes
date: 2015-06-07 11:24:06
---
## count ：计数

使用`count`成员函数获得RDD对象的成员`总数`，返回值为`长整型`。

** 语法 **

``` scala
def count(): Long
```

** 返回值 **

长整型，表示成员总数量。

** 示例 **

下面的示例返回文件的总行数（记录数）：

``` scala
scala> textFile.count() 
res0: Long = 126
```

请使用你创建的textFile变量，计算文件的总行数。

## top ：前N个记录

使用`top`成员函数获得RDD中的前`N个记录`，可以指定一个`排序函数`进行排序比较。 如果不指定排序函数，那么使用默认的Ascii码序进行记录排序。

** 语法 **

``` scala
def top(num: Int)(implicit ord: Ordering[T]): Array[T]
```

** 参数 **

- num： Int , 要返回的记录数量
- ord: Ordering[T] , 排序函数

** 返回值 **

包含前N个记录的数组，记录类型为T。

** 示例 **

下面的示例返回使用默认排序后的头两个记录

``` scala
scala> textFile.top(2)
res28:Array[String] = Array(your original work... ,you agree to...)
```

请使用你创建的textFile变量，读取前两行，存入变量 top2 中。

## take：无序采样
使用`take`成员函数获得`指定数量`的记录，返回一个数组。与`top`不同，`take`在提取记录 前不进行排序，它仅仅逐分区地提取够指定数量的记录就返回结果。可以将take方法 视为对RDD对象的`无序`采样。

** 语法 **

``` scala
def take(num: Int): Array[T]
```

** 参数 **

- num: Int , 要获取的记录数量

** 返回值 **

包含指定数量记录的数组，记录类型为T。

** 示例 **

下面的示例返回文件中的两行（两个成员）：

``` scala
scala> textFile.take(2)
res1: Array[String] = Array(# Apache Spark,"")
```

请使用你创建的textFile变量，读取两行，存入变量 take2 中。

## first : 取第一个记录

使用`first`成员函数获得RDD中的`第一个`记录。

** 语法 **

``` scala
def first(): T
```

** 示例 **

下面的示例返回文件的第一行（RDD的第一个记录）：

``` scala
scala> textFile.first()
res1: String = # Apache Spark
```

请使用你创建的textFile变量，读取第一行，存入变量 Line1 中。

## max : 取值最大的记录

使用`max`成员函数获得`值最大`的记录，可以指定一个排序函数进行`排序比较`。默认使用 Ascii码序进行排序。

** 语法 **

``` scala
def max()(implicit ord: Ordering[T]): T
```

** 参数 **

- ord : Ordering[T] , 排序函数

** 示例 **

下面的示例按字符串比较顺序找出最大的行记录：

``` scala
scala> textFile.max()
res10: String = your original work and that you license the ...
```

请使用你创建的textFile变量，读取最大的行长度。

## min : 取值最小的记录

使用`min`成员函数获得`值最小`的记录，可以指定一个`排序函数`进行排序比较。默认使用 Ascii码序进行排序。

** 语法 **

``` scala
def min()(implicit ord: Ordering[T]): T
```

** 参数 **

- ord : Ordering[T] , 排序函数

** 示例 **

下面的示例按字符串比较顺序找出最小的行记录：

``` scala
scala> textFile.min()
res 20: String = ""
```

请使用你创建的textFile变量，读取最小的行长度。

## reduce : 规约RDD

使用`reduce`成员函数对RDD进行规约操作，`必须`指定一个函数指定`规约行为`。

** 语法 **

``` scala
def reduce(f: (T, T) => T): T
```

** 参数 **

- f : 规约函数 , 两个参数分别代表RDD中的两个记录，返回值被RDD用来进行递归计算。

** 示例 **

下面的示例使用`匿名函数`，将所有的记录连接起来构成一个字符串：

``` scala
scala> textFile.reduce((a,b)=>a+b)
res60:String = #Apache SparkSpake is a fast...
```

请使用你创建的textFile变量，返回最长的单词。

## collect : 收集全部记录

使用`collect`成员函数获得RDD中的`所有记录`，返回一个数组。collect方法 可以视为对RDD对象的一个`全采样`。

** 语法 **

``` scala
def collect(): Array[T]
```

** 示例 **

下面的示例返回RDD中的所有记录：

``` scala
scala> textFile.collect()
res10: Array[String] = Array(# Apache Spark, "", Spark is a fast ...)
```

请使用你创建的textFile变量，读取全部行，并存入变量 Lines 中。