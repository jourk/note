---
title: Spark-基础教程5-RDD变换Transformation
tags:
  - spark
categories:
  - Notes
date: 2015-06-08 11:24:06
---
## map : 映射

`映射`变换使用一个`映射函数`对RDD中的每个记录进行变换,每个记录变换后的新值集合构成一个新的RDD。

** 语法 **

``` scala
def map[U](f: (T) => U)(implicit arg0: ClassTag[U]): RDD[U]
```

** 参数 **

- f: 映射函数,输入参数为原RDD中的一个记录,返回值构成新RDD中的一个记录。

** 示例 **

下面的示例将textFile的每个记录（字符串）变换为其长度值,获得一个新的RDD,然后取回第一个记录查看：

``` scala
scala> textFile.map(line=>line.length).first()
res13:Int = 14
```
请使用你创建的textFile变量,计算全部的单词数量。

## filter : 过滤

`过滤`变换使用一个`筛选函数`对RDD中的每个记录进行筛选,只有筛选函数返回真值的记录,才 被选中用来构造新的RDD。

** 语法 **

``` scala
def filter(f: (T) => Boolean): RDD[T]
```

** 参数 **

- f: 筛选函数,输入参数为原RDD中的一个元素,返回值为True或False。

** 示例 **

下面的示例仅保留原RDD中字符数多于20个的记录（行）,获得一个新的RDD,然后取回第一个 记录查看：

``` scala
scala> textFile.filter(line=>line.length>20).first()
res20: String = Spark is a fast and generic ...
```

请使用你创建的textFile变量,返回所有长度不超过5的单词,保存到变量 Word5 中。

## sample : 采样

`采样`变换根据给定的`随机种子`,从RDD中`随机地`按`指定比例`选一部分记录,创建新的RDD。采样变换 在机器学习中可用于进行交叉验证。

** 语法 **

``` scala
def sample(withReplacement: Boolean, fraction: Double, seed: Long = Utils.random.nextLong): RDD[T]
```

** 参数 **

- withReplacement: Boolean , True表示进行替换采样,False表示进行非替换采样
- fraction: Double, 在0~1之间的一个浮点值,表示要采样的记录在全体记录中的比例
- seed：随机种子

** 示例 **

下面的示例从原RDD中随机选择20%的记录,构造一个新的RDD,然后返回新RDD的记录数：

``` scala
scala> textFile.sample(true,0.2).count()
res12: Long = 26
```

请使用你创建的textFile变量,随机抽取10%的单词,保存到变量 wordShuffle 中。

## union : 合并

`合并`变换将两个RDD合并为一个新的RDD,重复的记录`不会`被剔除。

** 语法 **

``` scala
def union(other: RDD[T]): RDD[T]
```

** 参数 **

- other: 第二个RDD

** 示例 **

下面的示例,首先对textFile这个RDD进行一个每行反转的映射变换,获得一个新的RDD,再 将这个新的RDD和原来的RDD：textFile进行合并,最后我们使用count查看一下总记录数：

``` scala
scala> textFile.map(line=>line.reverse).union(textFile).count()
res13: Long = 282
```

可以看到,合并后的总记录数是原来的2倍。

请使用你创建的textFile变量,与它自身合并,计算最终的记录总数,并保存到变量 countAgain 中。

## intersection : 相交

`相交`变换仅取两个RDD`共同`的记录,构造一个新的RDD。

** 语法 **

``` scala
def intersection(other: RDD[T]): RDD[T]
```

** 参数 **

- other: 第二个RDD

** 示例 **

下面的示例将每个记录进行逆转后的RDD与原RDD相交,获得一个新的RDD,我们使用collect回收全部 数据以便显示：

``` scala
scala> textFile.map(line=>line.reverse).intersection(textFile).collect()
res27: Array[String] =Array(" ","")
```

可以看到,只有空行被保留下来,因为空行的逆序保持不变。

请使用你创建的textFile变量,与它自身相交,计算最终的记录总数。

## distinct : 剔重

`剔重`变换剔除RDD中的`重复`记录,返回一个新的RDD。

** 语法 **

``` scala
def distinct(): RDD[T]
```

** 示例 **

下面的示例将RDD中重复的行剔除,并返回新RDD中的记录数：

``` scala
scala> textFile.distinct().count()
res20: Long =91
```

请使用你创建的textFile变量,计算所有不重复的单词总数。