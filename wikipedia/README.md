编程作业1：Wikipedia
===================
- [编程作业1：Wikipedia](https://www.coursera.org/learn/scala-spark-big-data/programming/CfQX2/wikipedia)

# Wikipedia
首先，请先下载作业：[wikipedia.zip](http://alaska.epfl.ch/~dockermoocs/bigdata/wikipedia.zip)，并解压缩。还需要下载[数据（133 MB）](http://alaska.epfl.ch/~dockermoocs/bigdata/wikipedia.dat), 并放置 src/main/resources/wikipedia目录下。

在本作业中，您将使用 Spark 来探索全文维基百科文章。

对于公司来讲，是否采用新兴的编程语言，事先衡量一下编程语言的流行程度是非常重要的。于这个原因，行业分析公司RedMonk每两年使用各种数据源（通常来自GitHub和StackOverflow等网站）计算出编程语言流行度的排名。在本作业中，我们将使用维基百科的全文数据来生成编程语言流行程度的基本指标，以查看我们基于维基百科的排名是否与流行的RedMonk排名有任何关系。

## 设置Spark

为了简化逻辑，我们将以“ local ”模式运行Spark。这意味着完整的Spark应用程序将在您的笔记本电脑本地的一个节点上运行。

首先，我们需要一个SparkContext。一个SparkContext是集群的"连接句柄"。拥有SparkContext后，您可以使用它来创建和填充带有数据的RDD。要创建SparkContext，首先需要创建一个SparkConfig实例。一个SparkConfig代表你的Spark应用程序的配置。在这里，您必须指定您打算以“local”模式运行您的应用程序。此时还必须为Spark应用程序命名。如需帮助，请参阅[Spark API文档](https://spark.apache.org/docs/2.1.0/api/scala/index.html#org.apache.spark.package)。

## 读入维基百科数据

有几种方法可以将数据读入Spark。最简单方法是使用 SparkContext 的parallelize方法将内存中的现有集合转换为RDD 。 

我们已经在对象 WikipediaData 对象中实现了一个 parse 方法，该对象解析数据集的一行并将其转换为WikipediaArticle。

创建一个RDD（实现val wikiRdd），其中包含文章的WikipediaArticle对象。

## 计算编程语言的排名

我们将使用一个简单的度量标准来确定编程语言的受欢迎程度：至少提及一次该语言的维基百科文章的数量。

### 排名尝试＃1：rankLangs

#### 计算 occurrencesOfLang
通过实现一个辅助方法 occurrencesOfLang 开始， 这个方法计算 RDD[WikipediaArticles] 中至少提及一次该语言的维基百科文章的数量。为简单起见，我们检查文章文本中至少有一个单词（用空格分隔）等于给定的语言。

#### 计算 ranking, rankLangs
使用occurrencesOfLang，实现一个方法 rankLangs ，它返回一个 pair 列表，pair的 第一部分是语言名称，第二部分是提及该语言的文章数量。rankLangs 一个可能的返回值示例如下：

```scala
List(("Scala", 999999), ("JavaScript", 1278), ("LOLCODE", 982), ("Java", 42))
```

列表应按降序排序。也就是说，根据该排名，文章数量最多的语言应该排在第一位。
请注意运行此部件需要多长时间！（它应该需要几十秒。）

### 排名尝试＃2：rankLangsUsingIndex

#### 计算倒排索引
倒排索引是一种索引数据结构。 它存储了内容（例如单词或数字）至一组文档的映射关系。特别是，倒排索引的目的是允许快速全文搜索。在我们的用例中，倒排索引对于从编程语言的名称映射到至少提及一次名称的维基百科文章集合非常有用。

为了更有效和更方便地使用数据集，实现一种计算“倒排索引”的方法，该方法将编程语言名称映射到至少出现一次的维基百科文章。

实现方法makeIndex，它返回以下类型的RDD：RDD [（String，Iterable [WikipediaArticle]）]。对于给定的langs列表中的每种语言，最多有一对。此外，每对中的第二个组件（Iterable）包含至少提及一次该语言的WikipediaArticles。

提示： 对于此部分， RDD的方法flatMap和groupByKey可能派上用场。

#### 计算 ranking, rankLangsUsingIndex

使用上一部分中实现的makeIndex方法来实现更快的方法来计算语言排名。

与 尝试#1 类似， rankLangsUsingIndex 应该返回一个 pair 列表，pair 第一部分是语言名称，第二部分是提及该语言的文章数量。列表应按降序排序。也就是说，根据该排名，文章数量最多的语言应该排在第一位。

提示： 对于此部分， PairRDD 的 mapValues 方法可能派上用场。

你能否注意到尝试#2有更好的性能，为什么？

### 排名尝试＃3： rankLangsReduceByKey

如果上面的倒排索引仅用于计算排名而不用于其他任务（比方说全文搜索），则使用reduceByKey方法直接计算排名更有效，无需先计算倒排索引。请注意，reduceByKey方法仅定义在PairRDD 上（即 RDD 中每个元素都是 key-value 对）。

实现rankLangsReduceByKey方法。
与 尝试#1 与 尝试#2 类似 , rankLangsReduceByKey 也返回同样的 pair 列表结果。

相对于 尝试#2， 是否感知性能提升？ 如果性能提升了，为什么？


