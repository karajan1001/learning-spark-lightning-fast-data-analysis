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

比如: 
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
转化操作返回一个新的RDD，比如`filter`可以根据条件表达式返回符合要求的行。转化对象之后还能继续操作，某些转化操作的输入是多个而非一个RDD比如`union`。spark使用谱系图来记录不同的RDD之间转化关系，以便在计算还有持久化时恢复丢失数据。

```scala
val action_0 = lines.filter(line=>line.contains("\"r\\\":0"))
val action_2000_2000 = action_2000.union(action_2000_)
```

### 3.3.2 行动操作
行动操作返回非RDD写入外部系统，触发实际计算。比如常用的`take`可以返回少量元素，`collect`可以返回全部数据。

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
可以传递内联函数，方法的引用，或者静态方法。如果是函数，则引用的数据必须是可以序列化的(实现了Java的`Serializable`接口)。不过我们依然可以将对象中的方法或者字段包装到局部变量中去。

```scala
// 这样会传递整个this
rdd.map(x => x.split(query))
// 这样则不会
val query_ = this.query
rdd.map(x => x.split(query_))
```

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
最常见的是map和filter,配合lambda 使用的方法类似 python自带的map和filter函数。 还有一种flatMap可以完成1个输入多个输出的操作。比如一个句子提出多个关键词，不过要注意得到数组会被展平。比如

map:
```scala
scala> val input = sc.parallelize(List(1,2,3,4))
scala> input.map(x => x*x).collect()
res10: Array[Int] = Array(1, 4, 9, 16)
```
flatMap:
```scala
scala> action_2000_.flatMap(line => line.split("\"")).take(10)
res12: Array[String] = Array({, body, :, 19-01-28 00:17:03 INFO  [LogUtil] [lambda$log$0] {\, a\, :\, \, ,\, alg\, :\)
```

2. 伪集合操作
- `distinct` : 生成唯一元素的新RDD，会混洗所以速度比较慢。
- `union` : 两个RDD中元素的并集，不会进行去重。
- `intersection` : 两个RDD中元素的交集，会进行去重，性能比`distinct`更差。
- `subtract` : 类似集合减法，也需要混洗。
- `cartesian` : 笛卡尔积，生成A X B
- `sample` : 采样，两个参数为是否替换和采样率。 数量不固定，似乎每个样本独立采样。

3. 行动操作
最常见是`reduce`和`fold`，他们返回的结果总是和RDD中的元素类型相同，`fold`可以加上初始值，初始值需要是不会单元元素，比如加就是0，乘就是1，不会随着lambda表达式多次运算而改变，不然结果可能不稳定。`aggregate`则可以返回和RDD中元素不一样的数据类型。

- `collect` : 返回所有元素
- `count` : 返回元素个数
- `countByValue` : 各个元素次数
- `take` : 返回 n 个元素
- `top` : 返回最前面的n 个元素
- `takeOrdered`: 按某种顺序返回最前面 n 个元素
- `takeSample`: 返回任意
- `reduce`: 并行整合
- `fold`: 和reduce类似需要初值
- `aggregate`: 聚合
- `foreach`: 对每个元素使用给定函数

### 3.5.2 在不同RDD类型间转换

JAVA 和 SCALA 中一些特定类型的RDD可以使用特定类型的操作。比如数值RDD可以求`mean`和`variance`，键值对RDD可以使用`join`

## 3.6 持久化
使用`persist(level)`可以持久化RDD减少重复计算次数。等级有：

- MEMORY_ONLY : 使用空间大，CPU少，内存中
- MEMORY_ONLY_SER :空间小，CPU高，内存中
- MEMORY_AND_DISK : 空间大，CPU中，部分内存，部分磁盘
- MEMORY_AND_DISK_SER : 空间少，CPU高，内存部分，磁盘部分
- DISK_ONLY : 空间少，CPU高，磁盘中

## 3.7 总结
