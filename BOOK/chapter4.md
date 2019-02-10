# 第四章 键值对操作

KV对是RDD中最常见数据类型。

## 4.1 动机
Spark 为包含KV对类型的`pair RDD`提供了一些专有的操作。

## 4.2 创建Pair RDD
创建的方法各有不同，总结就是返回一个二元组。比如python和java为:

```python
pairs = lines.map(lambda x: (x.split(" ")[0], x))
```
```scala
val pairs = lines.map(x => (x.split(" ")(0), x))
```

## 4.3 Pair RDD 的转化
所有标准RDD上的转化操作都可以作用在Pair RDD上。除此外还有很多和KeyValue相关的操作。

- 单RDD操作
  - reduceByKey: 合并相同的键值。
  - groupByKey: 对相同的键值分组。
  - combineByKey: 使用不同的返回合并具有相同键的值。
  - mapValues: 对value使用函数。
  - flatMapValues: 对 pair RDD 中的每个值应用一个返回迭代器的函数，然后对返回的每个元素生成一个对应key的Pair RDD 。
  - keys: 返回只有key的RDD。
  - values: 返回只有value的RDD。
  - sortByKey: 返回根据key排序的Pair RDD的RDD。

- 双RDD操作
  - subtractByKey: 删掉RDD中key与第二个rdd中key相同的元素。
  - join: 内链接，两个RDDkey相同的元素的笛卡尔积。
  - rightOuterJoin: 对两个RDD进行连接操作(笛卡尔积)，确保第一个RDD的key必须存在
  - leftOuterJoin: 对两个RDD进行连接操作，确保第二个RDD的key必须存在
  - cogroup: 将两个RDD 中拥有相同key的数据分组到一起

如果要对pairRDD中的key或者value 进行筛选可以使用
```scala
cat.filter{case (key, value) => value < 5}.collect()
// res66: Array[(Int, Int)] = Array((1,2), (3,4))
```


### 4.3.1 聚合操作
`reduceByKey`,`foldByKey`和他们的标准RDD版本的区别在于，这是一个转化操作返回结果依然是一个Pair RDD，相当于对每个Key相同的元素组成的RDD 做了一个`map`或者`fold`操作。比如如下命令可以对每个key 求 value 均值。
```python
num1.mapValues(lambda x: (x,1)).reduceByKey(lambda x,y: (x[0] + y[0], x[1]+y[1])).mapValues(lambda x: x[0]/x[1])
```

```scala
rdd.mapValues(x=> (x, 1)).reduceByKey((x, y) => (x._1 + y._1, x._2 + y._2)).mapValues(x => (x._1.toFloat / x._2)).collect()
```

> `combineByKey`是更为通用的基于key进行合并的函数。类似于对每个key使用一次`aggregate`。所以这个操作可以返回和输入数据类型不同的返回值。

combineByKey 会遍历分区中所有的元素。每次遇到新元素都会创建一个 Combiner。这个过程发生每个分区而不是整个RDD，如果不同分区都遇到过相同的key此时需要`mergeValue`来合并不同分区的值。

> 如果在map端聚合无法得到收益可以禁用它。

上面平均值可以写为：
```python
sum_count = nums.combineByKey((lambda x: (x,1)),  # 创建combiner 当遇到新的key时候的初始化函数。
                              (lambda x, y:(x[0] + y, x[1] + 1)), # 分区内map端聚合
                              (lambda x, y: (x[0] + y[0], x[1] + y[1]))) # 分区与分区聚合
avg_ = sum_count.map(lambda key, xy: (key , xy[0]/xy[1]).collectAsMap()
```

```Scala
rdd.combineByKey(
    (v) => (v,1),  // 初始化
    (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1), // 分区内combine
    (acc1: (Int, Int), acc2:(Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2) // 分区间combine
).map{ case (key, value) => (key, value._1 / value._2.toFloat)}.collect()
```

#### 并行度调优
每个RDD都有固定树木的分区，分区数决定了并行度。本章大多数操作都能接受第二个参数，用来指定RDD的分区数。没有指定时Spark会根据集群大小推断一个默认值。

比如
```scala
rdd.mapValues(x=> (x, 1)).reduceByKey((x, y) => (x._1 + y._1, x._2 + y._2), 10) // 并行度为10
```

使用`repartition`会使分区混洗，这是个代价很大的操作。

### 4.3.2 数据分组

groupByKey可以根据Key对RDD进行分组，分组的结果是`[K, Iterable[V]]`。除此以外还有`cogroup`可以得到`[(K, (Iterable[V], Iterable[W]))]`

### 4.3.3 连接

连接操作，也是数据聚合常用操作之一。spark支持左连接，右连接，内连接和交叉连接。具体参考上面的双RDD操作。

### 4.3.4 数据排序
数据排序`sortByKey`可以接受ascending，和keyfunc作为参数决定顺序倒序还有比较方式。

## 4.4 Pair RDD 的行动操作
除了普通RDD上所有的行动操作。Pair RDD 还支持
| 函数 | 描述 | 
|:--:|:--:|
| countByKey  |返回每个key的元素个数|
| collectAsMap  |将结果用map表的方式返回|
| lookup(key)  | 返回给定key对应所有值 | 

## 4.5 数据分区
分区，可以控制数据分布，以获得在分布式系统中更少的网络传输，更多的性能。系统可以根据KEY的函数进行分区，保证同一组的key出现在同一个节点上。使用的方法为`partitionBy`，这样Pair RDD中的key就会按照某种规则到达同一个分区。这个操作是一个转化操作。

```scala
val userData = sc.sequenceFile[UserId, UserInfo]("hdfs://...")
.partitionBy(new HashPartitioner(100)) // 构造100个分区 应该大于集群中的总核心数
.persist()
```

之后的很多操作就会利用到利用到分区信息，其他一些操作也会利用已有分区信息，比如`sortByKey`和`groupByKey`还有`join`等操作。会将需要join的数据按照key发送到对应分区。不过那些会改变分区信息的操作比如`map`会导致RDD分区信息丢失。


### 4.5.1 获取分区的方式

```scala
val partitioned = rdd.partitionBy(new spark.HashPartitioner(2))
```
`rdd.partitioner`可以告诉我们每个key分别对应哪个分区。
```scala
partitioned.partitioner
//res88: Option[org.apache.spark.Partitioner] = Some(org.apache.spark.HashPartitioner@2)
```

### 4.5.2 从分区中获益的操作
所有根据Key的操作都能从分区中受益。比如`cogroup`, `groupWith`, `join`, `leftOuterJoin`, `RightOuterJoin`, `groupByKey`, `reduceByKey`, `combineByKey`, `lookup`。一些诸如`reduceByKey`的操作，可以让整个过程都发生在本地，避免低效的通信过程。

### 4.5.3 影响分区方式的操作
那些会影响分区的方式比如`map`可能会改变分区，所以操作后的RDD将不再带有分区信息。如果要避免这种情况可以使用`mapValues`。
### 4.5.4 PageRank示例

### 4.5.5 自定义分区方式
大多数应用使用`HashPartitioner`或者`RangePartitioner`就能解决大多数问题。但是某些情况，比如我们需要相似网站分到同一个区时可能，会需要提取网站再进行分区。 实现自定义分区器，需要`分区数，返回分区编号的方法，判断分区器相等性的方法`

## 4.6 总结

