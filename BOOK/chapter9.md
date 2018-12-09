# 第九章 Spark SQL

1. Spark SQL 可以从结构化数据源读取数据。
2. 可以从外部工具中通过JDBC连接Spark SQL进行查询。
3. 与内部RDD接口高度整合。

## 9.1 连接Spark SQL
如果要把SPARK SQL连接一个部署好的HIVE，必须把`hive-site.xml`复制到`SPARK_HOME/conf`。如果没有HIVE支持SPARK SQL会使用本地文件系统，否则使用HDFS

## 9.2 在应用中使用SPARK SQL

### 9.2.1 初始化Spark SQL

使用Spark SQL需要

```scala
// 有hive
import org.apache.spark.sql.hive.HiveContext
val hiveCtx = new HiveCOntext(...)
// 无hive
import org.apache.spark.sql.SQLContext
val sc = new SparkContext(...)
```

```python
from pyspark.sql import HiveContext, Row
from pyspark.sql import SQLContext, Row
hiveCtx = HivContext(sc)
```
之后就能进行SQL操作了。

### 9.2.2 基本查询

```scala
val iput = hiveCtx.jsonFile(inputFile)
input.registerTempTable("tweets")
val topTweets = hiveCtx.sql("SLECT ... FROM .. ORDER BY ...")
```

### 9.2.3 SchemaRDD
读取数据和查询返回的都是SchemaRDD。这是一个Row对象组成的RDD。在Spark 1.3 中以后，SchemaRDD可能改名为`Datarame`这个特殊RDD除了支持现有的RDD转化操作，还支持一些其他的操作。
1. ROW对象。
使用ROW对象可以
```scala
val topTweetText = topTweets.map(row=>row.getString(0)) // 将第一列作为String类型返回。
```

### 9.2.4 缓存
SparkSQL中缓存的方式和普通RDD不同，使用专门的`cacheTable`方法可以更高效。还可以通过SQL语句
```sql
CACHE TABLE tablename
UNCACHE TABLE tablename
```
来缓存或者删除缓存。这些缓存只有当驱动器程序不退出时才会存在于内存中。

## 9.3 读取和存储数据

### 9.3.1 Apache Hive

### 9.3.2 Parquet
这是一种流行的列式存储格式，可以高效存储嵌套字段的记录。
```python
rows = hiveCtx.parquetFile(parquetFile)
names = rows.map(lambda row:row.name) # 读取
tbl = rows.registerTempTable"people") # 注册
pandaFriends = hiveCtx.sql("Select name from people where ...") # SQL 命令
pandaFriends.saveAsParquetFile("hdfs://...") # 保存
```

### 9.3.3 JSON
```scala
val input = hiveCtx.jsonFile(inputFile) # 读取json文件
input.printSchema() # 打印表结构
```
### 9.3.4 基于RDD
```scala
case class HappyPerson(a:String, B:String)
val happyPeopleRDD = sc.parallelize(List(HappyPerson("holden", "coffee")) # 隐式转为SchemaRDD
happyPeopleRDD.registerTempTable("happy_people")
```

## 9.4 JDBC/ODBC 服务器
可以提供JDBC连接支持，需要Spark在打开Hive支持的选项下编译。服务器可以通过
```bash
sbin/start-thriftserver.sh
```
然后在`localhost:10000`上监听。

### 9.4.1 使用Beeline
之后就可以使用Beeline进行HiveSQL命令操作。
### 9.4.2 长生命周期的表与查询
JDBC Thrift服务器是一个单驱动器程序，使得共享成为可能。

## 9.5 用户自定义函数
又叫<font color='red'>UDF</font>可以让我们使用注册的自定义函数在SQL中调用。

### 9.5.1 Spark SQL UDF
```scala
registerFUnction("strLenScala", (_: String).length)
val tweetLength = hiveCtx.sql("SELECT strLenScala('tweet') FROM tweets")
```

### 9.5.2 Hive UDF
也可以支持Hive UDF。需要使用`HiveContext`,注册方式为：
```scala
hiveCtx.sql("CREATE TEMPORARY FUNCTION name AS classfunction")
```

## 9.6 Spark SQL 性能

| 选项 | 默认| 用途|
|:---:|:---:|:---:|
|spark.sql.codegen|false|每条语句运行时编译为JAVA二进制</br>小数据集变慢，大数据集变快|
|spark.sql.inMemoryColumnar</br>Storage.compressed|false|自动对内存中的列式存储压缩|
|spark.sql.inMemoryColumnar</br>Storage.batchSize|1000|列式缓存每个批处理大小</br>太大可能导致内存不够</br>太小会导致压缩比下降|
|spark.sql.parquet</br>.compresssion.codec|snappy|压缩方式|

## 9.7 总结
除此外，三到六章内容也适合SchemaRDD

