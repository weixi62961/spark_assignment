练习编程作业：Example
===================
- [练习编程作业：Example](https://www.coursera.org/learn/scala-spark-big-data/programming/I6L8m/example)

# 入门

此任务的目标是熟悉此课程所需的基础结构和工具。即使此作业的成绩将被排除在课程的最终成绩之外，仔细完成此作业也很重要。

## 安装所需的工具

在此之前，最重要的是确保正确安装所有工具。仔细查看“工具设置”页面，确认所有列出的工具都适用于您的计算机。

- Oracle JDK 1.8 及以上
- Sbt 0.13 及以上
- Eclipse的Scala IDE，Intellij IDEA

# 作业

## 第一部分：获取项目文件

下载[example.zip](http://alaska.epfl.ch/~dockermoocs/bigdata/example.zip)讲义存档文件并将其解压缩至 exmaple 目录。

## 第二部分：使用Scala REPL

在本课程中，我们将将通过sbt启动Scala REPL（交互式Scala控制台）。这样您就不需要在计算机上安装Scala发行版，因为sbt就足够了。
通过 [Sbt Tutorial](https://www.coursera.org/learn/scala-spark-big-data/supplement/817lQ/sbt-tutorial)进行学习，确保 sbt 正常工作。

## 第三部分：使用 IDE 打开项目

例如在 IntelliJ IDEA 中， "Import Project" 选择 example 目录，"Import mode" 选择 sbt，点击 Next，然后 Finish。
在文件夹src / main / scala中，双击文件Lists.scala。此文件中有两种方法需要实现（sum和max）。

## 第四部分：运行代码

编码完成后，可以使用两种方式运行代码。

### 使用Scala REPL

```bash
cd example
sbt console
scala> import example.Lists._
scala> max(List(1, 3, 2))
res0: Int = 3
```

### 使用 Main 对象

在 IntelliJ 工程中，创建 Main.scala

```scala
object Main extends App {
  println(Lists.sum(List(1, 3, 2)))
  println(Lists.max(List(1, 3, 2)))
}
```

通过菜单 Run - Run ’Main‘ 运行

## 第五部分：编写单元测试

在本课程的作业中，我们将要求您为代码编写单元测试。单元测试是测试代码的首选方法，因为与REPL命令不同，单元测试会保存，并且可以根据需要重新执行。这是一种很好的方法，可以确保在以后更改某些您之前编写的代码时不会出现任何问题。

我们将使用ScalaTest测试框架来编写单元测试。导航到文件夹src / test / scala并在包示例中打开文件ListsSuite.scala。此文件包含分步教程，以了解如何编写和执行ScalaTest单元测试。

## 第六部分：提交作业

一旦您实施了所有必需的方法并彻底测试了代码，您就可以将其提交给Coursera。提交解决方案的唯一方法是通过sbt。

```bash
sbt
submit your.email@domain.com submissionPassword
```
