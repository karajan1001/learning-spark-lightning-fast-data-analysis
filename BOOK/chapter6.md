# 第六章 Spark编程进阶

## 6.1 简介
<font color='red'>共享变量</font>: 一种可以在Spark任务中使用的特殊类型变量。主要两种：
1. 累加器
2. 广播变量。

## 6.2 累加器
Spark运算时可以使用驱动器程序中定义的变量，但是这些变量都是在程序启动时一次性复制到集群的每个任务中的。更新这些值并不会影响驱动器,或者其他任务中的对应变量。而共享变量则可以突破这个限制。 使用方法是:

```scala
val blankLines = sc.accumulator(0) # 定义一个累加器
file.flatMap(line =>{ if (...){
    blankLines += 1 # 累加器支持的操作
}
...
})
println(blankLines.value) # 在驱动器中输出累加器的值
```

> 工作节点的任务不能方位累加器的值。累加器对其只能写不能读。

### 6.2.1 累加器与容错性

> 行动操作中每个任务的累加器修改只会应用一次。但是转化操作就没有这样的保证。转化操作应用累加器只能在调试中使用。

### 6.2.2
可以使用更复杂的累加器`AccumulatorParam`。不过所进行的操作必须满足交换率和结合(**阿贝尔群**）。

## 6.3 广播变量
让程序向所有节点广播一个较大的<font color='red'>只读值</font>。相比将值通过程序传递，有两个好处：
1. 只用传一次，不用每个任务都传。
2. 传递较大数据时效率更高。

格式为:
```scala
val signPrefixes = sc.broadcast(localCallSignTable())
val countryContactCounts = contactCounts.map{ case (sign, count) =? val country = lookupInArray(signm signPrefixes.values) (country, count)}.reduceByKey((x, y => x + y)
countryContactCOunts.saveAsTextFile(outputDir + "/countries.txt")
```

### 6.3.1 广播的优化

## 6.4 基于分区进行操作

基于分区对数据进行操作可以避免为每个元素进行重复的配置。比如**打开数据库链接**等等。分区**map**和分区**foreach**可以让部分代码只对每个分区运行一次。

| 函数 | 提供 | 返回 | 对于RDD 中的函数签名 |
|:----:|:----:|:----:|:----:|
|mapPartitions | 分区中元素迭代器 | 返回的元素的迭代器 | f:(Iterator[T]) -> Iterator[U]|
|mapPartitionsWithIndex | 分区序号以及</br>分区中元素迭代器 | 返回的元素的迭代器 | f:(Iterator[T]) -> Iterator[U]|
| foreachPartitions | 元素迭代器 | 无 | f:(Iterator[T]) -> Unit |

## 6.5 与外部程序的管道
可以通过<font color='green'>pipe()</font>方法使用任意语言实现Spark任务中部分逻辑。

## 6.6 数值RDD操作
对包含数值数据的RDD提供了一些描述性统计操作。有**count**,**sum**,**mean**,**max**,**min**,**variance**,**sampleVariance**,**stdev**,**sampleStdev**

## 6.7 总结

