编程作业2：stackoverflow
===================
- [编程作业2：stackoverflow](https://www.coursera.org/learn/scala-spark-big-data/programming/FWGnz/stackoverflow-2-week-long-assignment)

# StackOverflow
首先，请先下载作业：[stackoverflow.zip](http://alaska.epfl.ch/~dockermoocs/bigdata/stackoverflow.zip)，并解压缩。还需要下载[数据（170 MB）](http://alaska.epfl.ch/~dockermoocs/bigdata/stackoverflow.csv), 并放置 src/main/resources/stackoverflow目录下。

作业的总体目标是实现分布式k-means算法，该算法根据帖子在流行的问答平台StackOverflow上的得分来进行聚类。此外，可以针对不同的编程语言并行执行此聚类，并且结果可以进行对比。

动机如下：StackOverflow是文档的重要来源。然而，基于答案可感知的价值，不同的用户提供的答案可能具有非常不同的评级（基于用户投票）。因此，我们想看一下问题的分布及其答案。例如，StackOverflow用户发布了多少高评价的答案，以及他们的分数有多高？评级较高的答案与评分较低的答案之间是否存在很大差异？

最后，我们有兴趣对比不同的编程语言社区的这些分布。这些分布的差异可能反映出文件可用性的差异。例如，针对某个库的 API 文档，StackOverflow上的文档可能更好。然而，为了避免得出错误结论，我们将重点关注定义明确的问题。

*注意：我们假设您能够回想起K-means算法，在并行编程部分曾经介绍过。您可以参考[K-means作业](http://alaska.epfl.ch/~dockermoocs/bigdata/kmeans/kmeans.html)以了解算法概述！*

## 数据说明
您将获得一个CSV文件，其中包含有关StackOverflow帖子的信息。提供的文本文件中的每一行都具有以下格式：

```bash
<postTypeId>,<id>,[<acceptedAnswer>],[<parentId>],<score>,[<tag>]


1. <postTypeId>: 帖子的类型，Type 1 = 问题， Type 2 = 答案
2. <id>: 帖子的唯一 ID（无论帖子类型）
3. <acceptedAnswer>：被接受的答案的帖子 ID，可选，如无则为空字符串 
4. <parentId>： 父帖子的 ID。若本帖为答案，则 parentId 指向 问题帖子的 ID。 若本帖为问题，则 parentId 为空。
5. <score>： StackOverflow 分数（基于用户投票）
6. <tag>： 若本帖为问题， 则 tag 指示帖子所涉及的编程语言。 若本帖为答案， 则 tag 为空。
```

您将在 main 方法中看到以下代码：
```scala
    val lines   = sc.textFile("src/main/resources/stackoverflow/stackoverflow.csv")
    val raw     = rawPostings(lines)
    val grouped = groupedPostings(raw)
    val scored  = scoredPostings(grouped)
    val vectors = vectorPostings(scored)
```

对应于以下步骤：

1. lines: csv 文件每行的字符串
2. raw: 每行的原始帖子条目信息
3. grouped: 问题与答案组合在一起
4. scored: 问题与分数
5. vectors: 针对每个问题的（语言， 分数）对。

在工程中，前两个方法已经实现，你需要实现剩下的方法。

## 数据处理

下面我们介绍一下，在应用kmeans算法之前如何处理数据。

### 分组问题与答案

第一步，你要实现的方法是 groupedPostings
```scala
val grouped = groupedPostings(raw)
```
在变量 raw 中，保存的是简单的帖子，可能是问题或者答案，为了使用这些数据，我们需要把它们分组在一起。 问题以 postTypeId == 1 来标识。问题的答案以 postTypeId == 2 及 parentId == QID 来标识。

理想情况下，我们希望得到 RDD[(Question, Iterable[Answer])]。 但是，直接在问题上进行分组是昂贵的（你能想到为什么吗？）。因此，一个较好的替代方案是以 QID 为 Key，最终产生 RDD[(QID, Iterable[(Question, Answer)])].

在 groupedPostings 方法中， 首先分别过滤出问题及答案，然后以元组的第一个元素 QID 进行 join 操作。然后，使用其中一个 join 操作（哪一个 join 方法？）来得到 RDD[(QID, (Question, Answer))]， 最后一步得到 RDD[(QID, Iterable[(Question, Answer)])]。 你将使用哪个方法针对 pairRDD 的 key 来进行分组？

在上述描述中，我们使用了 QID, Question 及 Answer 类型。我们已经将它们定义为 Posting 和 Int 的类型别名。其定义在 package.scala 文件中：
```scala
package object stackoverflow {
  type Question = Posting
  type Answer = Posting
  type QID = Int
  type HighScore = Int
  type LangIndex = Int
}
```

以上信息，可以支持你实现 groupedPostings 方法了。 方法签名如下：
```scala
def groupedPostings(postings: RDD[Posting]): RDD[(QID, Iterable[(Question, Answer)])]
```

### 计算分数

第二步，实现方法 scoredPostings 。它的返回值为 RDD[(Question, Highscore)]，(注意：Highscore是答案中最高的得分，而非标记为 acceptAnswer 的答案），scoreRDD 类型如下：

```scala
val scored: RDD[(Question, Highscore)] = ??? 
```

例如，上述RDD应该包含下述元祖：
```scala
((1, 6,   None, None, 140, Some(CSS)),  67)
((1, 42,  None, None, 155, Some(PHP)),  89)
((1, 72,  None, None, 16,  Some(Ruby)), 3)
((1, 126, None, None, 33,  Some(Java)), 30)
((1, 174, None, None, 38,  Some(C#)),   20)
```
*注：请利用 scoredPostings 中提供的 answerHighScore 方法*

### 为聚类创建向量
接下来，我们准备聚类算法的输入。为此，我们将 scoreRDD 转换为要聚类的向量的 vectorRDD。在我们的例子中，向量应该是一个 pair，包含下述的两个组件：

1. LangIndex：编程语言在langs列表中的索引，乘以langSpread因子
2. HighScore：计算所得的最高分

其中，langSpread因子已提供。基本上，它使用 euclideanDist 函数计算的距离，确保关于不同编程语言的帖子至少具有距离50000 。稍后您将了解此距离的含义以及将其设置为此值的原因。vectorRDD 类型如下：
```scala
val vectors: RDD[(LangIndex, HighScore)] = ???
```
例如，上述RDD应该包含下述元祖：
```scala
(350000, 67)
(100000, 89)
(300000, 3)
(50000,  30)
(200000, 20)
```

通过使用给定的 firstLangInTag 辅助方法实现 vectorPostings 方法。
*（用于测试： score RDD 应有 2121822 个条目）*

### Kmeans 聚类
```scala
val means = kmeans(sampleVectors(vectors), vectors)
```
基于这些初始均值和提供的变量收敛方法 converged ，迭代实现K-means算法：

1. 将每个向量与最接近的均值（其聚类）的索引配对;
2. 通过平均每个聚类的值来计算新的均值。

为了实现这些迭代步骤，使用所提供的方法 findClosest， averageVectors， 及 euclideanDistance

#### 备注1：
在我们的测试中，在44次迭代（对于langSpread = 50000）和104次迭代（对于langSpread = 1）之后达到收敛，并且对于第一次迭代，距离保持增长。虽然看起来有些不对劲，但这是预期的行为。有许多远程点迫使核心移动相当多，每次移动时效果都会波动到其他核心，这些核心也会四处移动，依此类推。请耐心等待，在44次迭代中，距离将从超过100000降至13，满足收敛条件。

如果您想更快地获得结果，请对数据进行下采样（每次迭代更快，但仍然需要大约40步才能收敛）：
```scala
val scored = scoredPostings(grouped).sample(true, 0.1, 0)
```
但是请记住，我们将在完整数据集上验证您的作业。意味着您可以对实验进行下采样，但在提交评分时确保您的算法适用于完整数据集。

#### 备注2：
变量langSpread对应于从聚类算法的角度来看语言的距离。 当 langSpred 设置为 50000 时， 语言太远而无法聚集在一起，从而导致聚类仅考虑每种语言的分数（类似于跨语言划分数据然后基于分数进行聚类）。当langSpread设置为1 时，会发生更有趣（但不太科学）的聚类（我们无法将其设置为0，因为它完全丢失了语言信息），我们根据得分进行聚类。现在查看哪些语言支配最重要的问题？

### 聚类计算细节

在调用kmeans之后， main 方法中有以下代码：
```scala
val results = clusterResults(means, vectors)
printResults(results)
```

实现clusterResults方法， 针对每个集群，计算如下：

1. 聚类中占主导地位的编程语言
2. 主导编程语言中，答案所占的百分比
3. 每个聚类的大小（其中包含的问题数量）
4. 答案得分最高的中位数

一旦得到上述返回值， 通过 printResults 方法打印出来。

## 问题

1. 您认为对数据进行分区会有所帮助吗？
2. 您考虑过保存（persist）中间计算结果吗？您能想到为什么将数据保存在内存中可能对此算法有帮助吗？
3. 在非空聚类中，有多少聚类以“Java”作为标签（基于大多数问题，见上文）？为什么？
4. 只考虑“Java聚类”，哪些聚类脱颖而出，为什么？
5. 与“Java聚类”相比，“C＃聚类”有何不同？

提示：如果您打破了评分者系统时间或内存限制，请考虑分区或持久性如何（如果有的话）可以帮助您获得一些性能。
请注意，我们的评分者只对您的方法进行单元测试。它不会运行main方法，因此请确保将任何缓存或分区代码放在main之外。