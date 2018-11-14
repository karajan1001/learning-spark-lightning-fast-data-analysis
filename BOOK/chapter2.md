# 第二章 Spark 下载与入门

## 2.1 下载 Spark
在[官网](http://spark.apache.org/downloads.html)下载，文件名为`spark-2.4.0-bin-hadoop2.7.tgz`。下载后对该文件解压缩。
```bash
tar -xf spark-2.4.0-bin-hadoop2.7.tgz
```

解压后的目录结构为:
```bash
/README.md # 说明文档
/bin/ # 交互的可执行文件，包括spark shell
/core/, /streaming/, /python/... # 各种源代码
/examples/ # 包含可以查看和运行的程序样例。
```

## 2.2 Spark 中 Python 和 Scala 的 shell
spark 的shell 主要用来进行数据探索。它会自动调动分布式集群的存储和计算单元，在秒量级处理完成TB级的数据。spark 提供 python 以及 增强版的

如果要用jupyter 版本的pyspark需要增加两个环境变量。
```bash
export PYSPARK_DRIVER_PYTHON=ipython3 # 启动pyspark的命令，可以替换为python3.6 python2.7
export PYSPARK_DRIVER_PYTHON_OPTS="notebook" # 启动pyspark时候的参数，可以加 --config
```

## 2.3 Spark核心概念简介

每个Spark应用都有一个驱动器程序来发起集群上的各种并行操作。在交互式模式中，这个驱动器程序就是spark shell本身。驱动器通过`sc(SparkContext)`来访问spark，这个对象代表对计算集群的一个连接。有了`SparkContext`就能用它来创建RDD。

## 2.4 独立应用
除了交互式，Spark也可以在独立应用中连接使用。与交互式的唯一区别是，需要自行初始化 `SparkContext`。 对于JAVA和SCALA需要体检spark-core相关的maven依赖。 而对于Python需要用spark自带的`bin/spark-submit`脚本运行。

### 2.4.1 初始化SparkContext
创建`SparkContext`只需要两个参数：
1. 集群URL，告诉Spark如何连接到集群。
2. 应用名，方便集群管理器中找到自己的应用。
```python
from pyspark import SparkConf, SparkContext
conf = SparkConf().setMaster("local").setAppName("My App")
sc = SparkContext(conf=conf)
```

```scala
import org.apache.spark.SparkConf
import org.apache.sparkparkContext
import org.apache.sparkparkContext._
val conf = new SparkConf().setMaster("local").setAppName("My App")
val sc = SparkContext(conf)
```
关闭Spark可以调用`sc`中的`stop`方法。更高级配置连接集群会在后面章节中提到。

### 2.4.2 构建独立应用
使用sbt或者maven构建文件构建应用。然后用`bin/spark-submit`执行这些应用。更详细案例参考[官方文档](http://spark.apache.org/docs/latest/quick-start.html)

## 2.5 总结
