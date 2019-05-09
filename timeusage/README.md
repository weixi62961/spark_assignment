# Coursera 作业3： Time Usage
- [练习编程作业: Time Usage](https://www.coursera.org/learn/scala-spark-big-data/programming/O0akp/time-usage)

## 准备
- 下载作业 [timeusage.zip](http://alaska.epfl.ch/~dockermoocs/bigdata/timeusage.zip) 及 [dataset 156M](http://alaska.epfl.ch/~dockermoocs/bigdata/atussum.csv), 将数据集放在 src/main/resources/timeusage/ 下

## 问题描述
数据集由kaggle提供，
CSV文件格式：第一行定义各列的字段名，第二行及以后是数据。描述了人们如何分配他们的时间（睡觉、吃饭、工作等）
我们的目标是识别3类活动：
1. 基本需求（睡觉和吃饭）
2. 工作
3. 其它（休闲）
然后发现人们如何在上述3种主要的活动之间分配时间，并且我们是否能够识别男性与女性， 就业与失业，年轻人（小于22岁）、壮年人（22-55岁）及老年人之间时间分配的差别。
在作业的最后，基于上述数据集，我们要回答下述问题：
1. 与其它活动的时间相比，我们花在基本需求上的时间是多少？
2. 女性和男性在工作上花了同样的时间吗？
3. 随着年龄的增长，人们花在基本需求的上时间是否有变化？
4. 与失业人员相比，就业人员花在在休闲上的时间是多少？

为实现这一目标，我们将首先使用Spark读取数据集，将其转换为中间数据集，该数据集将更易于用于我们的用例，并最终计算出信息来回答上述问题。

## 读取数据
最简单的创建 DataFrame 的方式包括从文件读取，并且让Spark SQL推断出数据的模式。但针对CSV文件，这个方法不奏效。因为推断的列类型通常是 String。
在我们的例子中，第一列是一个标识，这是一个String类型，但剩下的其它包含的都是数值类型。因为数据模式无法被Spark SQL正确推断出来，我们只能编程来进行定义。但是，列的数目非常多，为了不一个个手工定义，我们依赖CSV文件的第一行的列名。
1. 第一步是实现 dfSchema 方法。 把CSV文件的第一行转换为Spark SQL的StructType. 这也是 dfSchema 方法的目的。这个方法返回的StructType，描述了CSV的数据模式：第一列是StringType，其它列是DoubleType,所有列都不为空。
```scala
def dfSchema(columnNames: List[String]): StructType
```
2. 第二步是实现 row 方法。根据 dfSchema 方法返回的模式，把CSV文件中每一行数据转换为 Spark SQL 中的Row
```scala
def row(line: List[String]): Row
```

## 投影
你可能已经注意到，原始的数据集中，包含许多与我们想探知的问题无关的信息，同时也有一些过于细节的有用信息。例如，我们仅关于每个调查对象是“年轻人”、“壮年人”及“老年人”三者之一，而不关心他的确切年龄。
每个活动的时间花费过于具体了（数据中包含了超过50种活动）。我们也不需要这么细节，我们仅关注3种活动：基本需求、工作及其它
因此，基于原始的数据集，我们很难写出我们所关注的问题答案的查询。

作业的第二部分，转换原始的数据集至我们易于分析的格式。
1. 第一步：识别出哪些列涉及相同的活动。基于每列的相关性（[参考文档](https://www.bls.gov/tus/lexiconnoex0315.pdf)），我们推出下面的规则：
- “基本需求“活动（睡觉、吃饭等），记录在以 “t01”, “t03”, “t11”, “t1801” 及 “t1803” 开头的列中。
- “工作“活动，记录在以 “t05” 及 “t1805” 开头的列中。
- “其它“活动（休闲），记录在以“t02”, “t04”, “t06”, “t07”, “t08”, “t09”, “t10”, “t12”, “t13”, “t14”, “t15”, “t16” 及 “t18”  开头的列中（除去上述两种活动以外的）。
接下来我们的任务在于实现 classifiedColumns 方法，这个方法把所有列分为至3种类别（基本需求、工作、其它）中。这个方法应该返回一个包含3个List的元祖，这3个List分别是基本需求、工作及其它。
```scala
def classifiedColumns(columnNames: List[String]): (List[Column], List[Column], List[Column])
```
2. 第二步实现 timeUsageSummary 方法。 这个方法把细节数据集转换为汇总数据集。汇总数据仅包含6列：调查对象的工作状态（工作/非工作）、性别、年龄、基本需求活动的时间、工作活动的时间及其它活动的时间。
```scala
def timeUsageSummary(
  primaryNeedsColumns: List[Column],
  workColumns: List[Column],
  otherColumns: List[Column],
  df: DataFrame
): DataFrame
```
每个活动列包含原始数据集中相关活动列的值的加和。注意原始数据集中使用分钟计量，而在结果数据集中我们希望是小时。
描述工作状态、年龄及性别的列，相比原始数据集包含了简化后的信息。
最后，失业人员在结果数据集中被过滤掉了。
timeUsageSummary 方法的注释会提供你更多信息。

## 聚合
最后，我们希望针对工作状态、性别、年龄的组合，将每种活动的平均值进行对比。
我们需要实现 timeUsageGrouped 方法，它计算了每种活动所花费时间的平均小时数，以工作状态（就业/失业）、性别和年龄（年轻人、壮年人、老年人）分组及排序。值最后取整至十分之一。
```scala
def timeUsageGrouped(summed: DataFrame): DataFrame
```
现在你可以运行工程，检验最终的 DataFrame 中包含的数据了。
通过对比老年人男性和老年人女性时间分配，你能发现什么？老年人人分配在休闲上的时间比壮年人人多多少？就业人士花在工作上的时间是多少？

## 尝试另外的方法来操作数据
我们还可以使用纯 SQL 查询语句代替 DataFrame API，来实现 timeUsageGrouped 方法。请注意，有时使用编程API比纯SQL查询容易得多。如果您没有使用SQL的经验，您可能会发现[这些示例](https://en.wikipedia.org/wiki/SQL_syntax#Queries)很有用。
```scala
def timeUsageGroupedSqlQuery(viewName: String): String
```
您能想到上述查询，若用纯SQL编写将会是一场噩梦吗？
最后，在此作业的最后部分，我们将探索另一种实现查询的替代方法：使用类型化的Dataset而不是无类型的DataFrame。
```scala
def timeUsageSummaryTyped(timeUsageSummaryDf: DataFrame): Dataset[TimeUsageRow]
```
实现timeUsageSummaryTyped方法，将timeUsageSummary返回的DataFrame转换为DataSet [TimeUsageRow]。 TimeUsageRow是一种数据类型，用于对汇总数据集的行的内容进行建模。 要实现转换，您可能需要使用Row的getAs方法。 此方法检索行的命名列，并尝试将其值转换为给定类型。
```scala
def timeUsageGroupedTyped(summed: Dataset[TimeUsageRow]): Dataset[TimeUsageRow]
```
然后，实现timeUsageGroupedTyped方法，该方法与timeUsageGrouped方法执行相同的查询，但尽可能使用类型化API。 请注意，并非所有操作都具有类型化的等效项。 round是没有类型等效的操作示例：它将返回一个Column，您必须通过调用.as[Double]将其转换为TypedColumn。 另一个例子是orderBy，它没有类型化的等价物。 确保您的数据集具有模式，因为此操作需要一个（使用类型转换时，列名通常会丢失）。
