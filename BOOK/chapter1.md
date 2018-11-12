# 第一章 spark数据分析导论

## 1.1 spark是什么

spark 是一个用来实现快速而通用的集群计算的平台。

- 速度快，内存计算，比MapReduce更快。
- 适用于不同场景，批处理，迭代，交互式查询，流处理。 
- 接口丰富，Python,Java,Scala，可以调用hadoop资源。

## 1.2 一个大一统的软件栈
整合多个组件，互相配合，协助。

### 1.2.1 Spark Core
基本功能，任务调度，内存管理，错误恢复，存储，交互。

### 1.2.2 Spark SQL
通过Spark Sql可以用SQL和HIVE SQL查询数据。

### 1.2.3 Spark Streaming
Spark Streaming是对实时数据进行流式计算的组件。

### 1.2.4 Mlib
Spark提供的常见的机器学习功能的程序库。还包括了一些更底层的比如梯度下降API。

### 1.2.5 GraphX
GraphX是用来操作图的程序库，可以进行并行的图计算。

### 1.2.6 集群管理器
为了能适应不同数量集群的要求，和兼顾最大的灵活性。支持在各种集群管理器上运行比如Hadoop Yarn, Apache MEsos以及自带的建议调度器。

## 1.3 Spark的用户和用途
### 1.3.1 数据科学任务
主要负责分析数据并建模，提供Python，Scala的接口可以方便的进行交互式数据分析，也提供独立SQL shell，和标准的Spark程序或者Spark Shell。

### 1.3.2 数据处理任务
使用工程技术设计和搭建软件系统，实现业务用例。

## 1.4 Spark简史
伯克利分校RAD实验室（AMPLab）,因为MapReduce在迭代计算和交互计算的任务上表现得效率低下。现在Spark有[社区 Meetups](http://www.meetup.com/spark-users/)，有[峰会](http://spark-summit.org/)

## 1.5 Spark的版本和发布
保持着常规的发布新版本的节奏。

## 1.6 Spark的存储层次
Spark不仅可以将任何HDFS上的文件读取未分布式数据集，也可以支持其他支持Hadoop接口的系统，比如本地文件系统，Hive，Hbase。
