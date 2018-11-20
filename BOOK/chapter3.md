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
转化操作返回一个新的RDD，转化对象之后还能继续操作，某些转化操作的输入是多个而非一个RDD比如`union`。日志分析过程中可以创建RDD生成的谱系图。
![]()

### 3.3.2 行动操作
行动操作返回非RDD写入外部系统，触发实际计算。

### 3.3.3 惰性求值
这与haskell等函数式语言类似。是为了减少计算步骤，相比mapreduce减少磁盘读写次数。

## 3.4 向Spark传递函数

### 3.4.1 Python
可以传递`lambda`表达式或者顶层函数和局部函数，不过要注意不要将难以序列化的类传递给spark比如局部函数中的`self`
```python
class SearchFunctions(object):
    def getMatches(self, rdd):
        # 在'self.ismatch'中引用整个self
        return rdd.filter(self.ismatch)
    def getMatches(self, rdd):
        # 避免了引用整个self
        query = self.query
        return rdd.filter(query)
```

### 3.4.2 Scala
可以传递内联函数，方法的引用，或者静态方法。如果是函数，则引用的数据必须是可以序列化的(实现了Java的`Serializable`接口)。

### 3.4.3 Java 
函数需要作为实现了 Spark 中 `org.apache.spark.api.java.function` 包中的任一函数接口的对象来传递。

| 函数名 |  实现方式 | 用途 |
|:--:|:--:|:---:|
| Function<T,R> |R call(T) | 接受一个输入返回一个输出，比如map和filter |
| Function2<T1, T2,R> |R call(T1, T2) | 接受两个输入返回一个输出，比如aggregate和fold |
| FlatMapFunction<T,R> | Iterable<R> call(T) | 接受一个输入返回任意输出，比如flatmap |

## 3.5 常见的转化操作和行动操作
### 3.5.1 基本RDD
1. 针对各个元素的转化操作
最常见的是map和filter

### 3.5.2 在不同RDD类型间转换
