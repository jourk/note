---
title: Spark-基础教程3-交互分析
tags:
  - spark
categories:
  - Notes
date: 2015-06-06 11:24:06
---
## Spark Shell

Spark Shell提供一个简单的方式来学习Spark框架的API，同时也是一个可以用于交互数据分析的强大 工具。我们在本课程中使用Spark Shell的Scala版本。

在Spark目录中运行下面命令启动Spark Shell：

``` bash
cd /usr/local/spark
./bin/spark-shell
```

在一堆balabala的提示信息后，我们看到了Spark-Shell的提示符：

``` scala
scala>
```

Spark-Shell是一个`REPL`解释器，我们输入Scala表达式后，按`回车`就可以看到执行结果。

比如，要看Spark的版本，在提示符下输入：

``` scala
scala> sc.version
res2: String = 1.3.1
```

执行的结果反馈总是以 `变量名: 类型 = 值` 的形式显示。在上面的结果中，可以看到，执行的结果被放入一个 临时变量res2中，类型是String，值是1.3.1 。

看看你的Spark的版本是多少？

## 集群对象：SparkContext

当我们启动Spark-Shell后，就自动获得一个SparkContext对象实例，这个对象被存入变量`sc`。

在提示符下输入：`sc` ，可以看到sc的类型是`org.apache.spark.SparkContext`：

``` scala
scala> sc
res1: org.apache.spark.SparkContext = org.apache.spark.SparkContext@3c5a3436
```

SparkContext对象代表`整个`Spark集群，是Spark框架`功能的入口`，可以用来在集群中创建RDD、累加器变量和广播变量。 SparkContext对象创建时可以指明连接到哪个集群管理器上，在Spark-Shell启动时，`默认`连接到本地的集群管理器。

使用SparkContext对象（在Shell里，就是`sc`变量）的master方法，可以查看当前连接的集群管理器：

``` scala
scala> sc.master
res10: String = local[*] 
```

显示结果表明，我们确实连接到了本地的集群管理器上，*代表不明确指定在每个计算节点上使用的CPU核心数（`资源限额`）。

看看你的sc.master提示什么信息？

## 分布数据集：RDD

Spark的核心抽象是一个分布式数据集，被称为`弹性分布数据集（RDD）` ，代表一个`不可变的`、`可分区`、`可被并行处理`的成员集合。

RDD对象需要利用SparkContext对象的方法创建，Spark支持从多种来源创建RDD对象，比如：从本地文本文件创建、从Hadoop 的HDFS文件创建、或者通过对其他RDD进行变换获得新的RDD。

下面的示例使用本地Spark目录下的README.md文件创建一个新的RDD：

``` scala
scala> val textFile = sc.textFile("README.md")
textFile: spark.RDD[String] = spark.MappedRDD@2ee9b6e3 
```

我们看到，执行的结果是，返回了一个Spark.RDD类型的变量textFile，RDD是一个`模板类`，方括号里的String代表 这个RDD对象`成员的类型`。由于是一个对象，因此值用地址表示：spark.MappedRDD@2ee9b7e3 。

SparkContext对象的textFile方法创建的RDD中，一个`成员`对应原始文件的`一行`。我们看到在执行的结果中可以看到返回一个 RDD，成员类型为String，我们将这个对象保存在变量textFile中。

使用README.md文件，创建一个RDD，保存到变量 textFile中。

## RDD：变换与动作

RDD的内部实现了分布计算的功能，我们在RDD上执行的操作，是`透明地`在整个集群上执行的。也就是说，当RDD建立 后，这个RDD就不属于本地了，它在整个集群中有效。当在RDD上执行一个操作，RDD内部需要和`集群管理器`进行沟通协商。

对一个RDD可以进行两种操作：动作（`action`）和变换（`transformation`）。动作总是从集群中`取回数据`，变换总是获得一个`新的RDD`，这是两种操作的字面上的差异。

事实上，当在RDD上执行一个`变换`时，RDD仅仅`记录`要做的变换，只有当RDD上需要执行一个`动作`时，RDD才 通过集群管理器`启动`实质分布计算。

这有点像拍电影，`变换`操作只是`剧本`，只有导演喊`Action`的时候，真正的电影才开始制作。

## 感受动作和变换的区别

我们用一个例子来直观感受下动作和变换的区别。

下面的例子首先做一个映射变换，然后返回新纪录的条数。`map`是一个`变换`，负责将原RDD的每个记录变换到新的RDD，`count`是一个`动作`，负责获取这个RDD的记录总数。

先执行`map`，你应该看到很迅速干净地返回：

``` scala
scala> val rdd2=textFile.map(line=>line.length)
rdd2: org.apache.spark.rdd.RDD[Int] = MappedRDD[52] ...
```

再执行`count`，这会有些不一样：

```scala
scala> rdd2.count()
......
res10: Long = 141
.....
```

当执行map时，我们看到结果很快返回了。但当执行count时，我们可以看到一堆的提示信息，大概的意思就是 和调度器进行了若干沟通才把数据拉回来。

看起来确实这样，`变换`操作就只是写写剧本，`Action`才真正开始执行计算任务。

使用你刚才创建的textFile变量，进行上面的map和reduce操作，感受下map和reduce的区别。

## RDD动作：获取数据的控制权

对一个RDD执行动作指示集群将指定数据返回本地，返回的数据可能是一个具体的值、一个数组或一个HASH表。

让我们先执行几个动作：

``` scala
scala> textFile.count() // 这个动作返回RDD中的记录数
res0: Long = 126

scala> textFile.first() // 这个动作返回RDD中的第一个记录
```

`count`是一个动作，负责获取这个RDD的记录总数。`first`也是一个动作，负责返回RDD中的第一条记录。

在使用Spark时，最好在脑海中明确地区隔出两个区域：`本地域`和`集群域`。RDD属于集群域，那是Spark管辖的地带； RDD的动作结果属于本地域，这是我们的地盘。只有当RDD的数据返回本地域，我们才能进行再加工，比如打印等等。

请使用你的textFile变量，将其第一条记录保存到变量 x0 中。

## RDD变换：数据的滤镜

RDD变换将`产生`一个新的RDD。下面的例子中，我们执行一个过滤（`Filter`）变换，将获得一个新的RDD，由原 RDD中符合过滤条件（即：包含单词`Spark`）的记录成员构成：

``` scala
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: spark.RDD[String] = spark.FilteredRDD@7dd4af09
```

变量`lineWithSpark`现在是一个RDD，由变量`textFile`这个RDD中所有包含”Spark”单词的行构成。

由于一个RDD变换`总是`返回一个新的RDD，因此我们可以将变换和动作使用`链式语法`串起来。下面的 例子使用了`链式语法`解决一个具体问题：在文件中有多少行包含单词“Spark”？

``` scala
scala> textFile.filter(line => line.contains("Spark")).count() 
res3: Long = 15
```

这等同于：

```scala
scala> val rdd1 = textFile.filter(line => line.contains("Spark"))
...

scala> rdd1.count() 
res12: Long = 15
```

用链式语法写起来更流畅一些，不过这只是一种口味的倾向而已。

请使用你的textFile变量，计算出包含”Apache”的行数。

## 组合的威力

《道德经》说的是简单的东西组合起来也不得了。与之类似（当然还达不到那个高度）， RDD的诸多动作和变换，经过组合也可以实现`复杂`的计算，满足相当多现实的数据计算需求。

假设我们需要找出文件中单词数量最多的行，做个`map/reduce`就可以了：

```scala
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15
```

上面语句首先使用map变换，将每一行（成员）`映射`为一个整数值（单词数量），这获得了一个新的RDD。然后在 这个新的RDD上执行reduce动作，找到（返回）了单词数量最多的行。

请使用你的textFile变量，计算所有单词的数量。