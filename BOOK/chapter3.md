# 第三章 RDD编程

本章介绍Spark对数据的核心抽象 弹性分布式数据集（Resilient Distributed Dataset）。RDD 其实就是分布式的元素集合。

## 3.1 RDD 基础
Spark 中 RDD是一个不可变的分布式对象集合。每个RDD分为多个分区运行与不同节点上，RDD可以包含python java scala中的任意对象。用户可以
1. 读取外部数据集
2. 驱动器程序里分发驱动器程序中的对象集合(`set` 或者 `list`)。
来创造一个RDD。

RDD 支持两种类型的操作：
1. 转化操作(transformation)：生成一个新的RDD
2. 行动操作(action): 计算出一个结果，并把结果返回到驱动器程序中，或者外部存储系统。

Spark只有在行动操作时才会真正计算,这样可以节省计算资源。如果要在RDD计算完成后保存下来需要使用`RDD.persist()`

一次Spark程序的典型计算过程：
1. 创建RDD
2. 转化操作
3. 对需要的数据`persist()`操作
4. 行动操作触发计算。

## 3.2 创建RDD

两种创建RDD的方式：
1. 读取外部数据集。程序中更常用
2. 对驱动器中的集合并行化。只适合开发原型

```python
lines = sc.textFile("/path/toREADME.md") # 方式1
lines = sc.parallelize(["pandas", "i like pandas"]) # 方式2
```

```scala
val lines = sc.textFile("/path/toREADME.md") // 方法1 
val lines = sc.parallelize(List("pandas", "i like pandas")) // 方法2
```

## 3.3 RDD操作

### 3.3.1 转化操作

### 3.3.2 行动操作

### 3.3.3 惰性求值

## 3.4 向Spark传递函数


