# 第五章 数据读取与保存

## 5.1 动机

对于分布式数据集需要更多的API，三种常见的数据来源。
- 文件系统和文件格式
- Spark SQL中结构化数据
- 数据库与键值存储

## 5.2 文件格式

### 5.2.1 文本文件

```scala
val input = sc.textFile("file:///home/...") // 读取文件
val input = sc.wholeTextFiles("file:///home/...") // 读取文件夹, 生成文件名 文件内容对
result.saveAsTextFile("file:///home/...") // 将结果输出到文件夹
```

### 5.2.2 JSON

1. 读取
可以使用 `jackson` 库，先读取文本，然后进行解析每行处理为一个 json 遇到无法解析行则抛弃。
```python
import json
data = input.map(lambda x: json.loads(x))
```

```scala
import com.fasterxml.jackson.module.scala.DefaultcalaModule
import com.fasterxml.jackson.module.scala.experimental.ScalaObjectMapper
import com.fasterxml.jackson.module.scala.databind.ObjectMapper
import com.fasterxml.jackson.module.scala.databind.DeserializationFeature
case class Person(name: String, lovesPandas: Boolean)

val result = input.flatMap(record => {
    try{
        some(mapper.readValue(record, classOf[Person]))
    }catch
        case e: Exception => None
    }}) 
```


2. 保存
保存比读取简单,无需考虑解析失败。先将字符串RDD转为JSON格式，然后调用文本写入API 

```scala
rdd.map(mapper.writeValueAsString(_)).saveAsTextFile(outputFileName)
```


### 5.2.3 CSV
和json一样先读取文本然后解析
```scala
import Java.io.StringReader
import au.com.bytecode.opencsv.CSVReader

val input = sc.textFile(inputFile)
val result = input.map{line => val reader = new CSVReader(new StringReader(line));
    rader.readNext();
}
```

### 5.2.4 SequenceFile

SequenceFile: KV 结构的Hadoop格式。直接用`sc.sequenceFile`读取，用`saveAsSequenceFile`保存，Python无法使用。

读取
```scala
val data = sc.sequenceFile(inFile, classOf[Text], classOf[IntWritable]).map{case (x, y) => (x.toString, y.get())}

```
写入
```scala
val data = sc.paralleize(List(("panda", 3), ("kay", 6), ("snali", 2)))
data.saveAsSequenceFile(otputFile)
```

### 5.2.5 对象文件

和 SequenceFile 类似 用JAVA序列化写出来的对象文件，Python无法使用。

### 5.2.6 Hadoop输入输出格式
可以使用`hadoopFile`读，还有新API`newAPIHadoopFile`
```scala
val input = sc.newAPIHadoopFile(iputFile, classOf[LzoJsonInputFormat], classOf[LongWriterable], classOf[MapWritable], conf)
```
写入也有旧API`saveAsHadoopFile`和新API`saveAsNewAPIHadoopFile`。一些Hbase MongoDB之类都提供了直接读取HadoopFile的接口，这样可以不通过数据库查询语句而是直接读取数据。

### 5.2.7 文件压缩
使用文件压缩可以减少网络传输瓶颈，有助于提高性能。

## 5.3 文件系统

### 5.3.1 本地文件
Spark可以直接读取文件，不过要求文件在集群中所有节点的相同路径下都能找到。直接用`textFile("file:///path")`就可以找到。

### 5.3.2 Amazon S3
略，国内基本不会使用。

### 5.3.3 HDFS
路径指定`hdfs://master:port/path`就可以了。

## 5.4 SPARK SQL 中的结构化数据
Spark SQL 查询结果就是一个RDD。其中每条记录是一条row，每个字段是一个类似keyvalue的结构。

### 5.4.1 Apache HIVE
将Hive的配置文件，复制到`./conf/` 目录下然后用 Spark SQL 读取数据。返回结果和Spark SQL 一样。 
```java
import org.apache.spark.sql.hive.HiveContext

val hiveCtx = new org.apache.spark.sql.hive.HiveContext(sc)
val rows = hiveCtx.sql("select . from ... ")
```
### 5.4.2 JSON
json文本也可以用`Hive`类似方法读取。
```java
import org.apache.spark.sql.hive.HiveContext

val tweets = hiveCtx.jsonFIle("tweets.json")
tweets.registerTempTable("tweets")
val results = hiveCtx.sql("select . from ... ")
```

## 5.5 数据库

### 5.5.1 JAVA 数据库连接
使用JDBC连接

### 5.5.2 Cassandra
略
### 5.5.3 HBase
略
### 5.5.4 ElasticSearch
略
## 5.6 总结
