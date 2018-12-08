# 第八章 Spark调优与调试

## 8.1 使用sparkconf 配置
1.  可以通过通过这个变量为spark任务进行配置。
```scala
val conf = new SparkConf()
conf.set("spark.app.name", "My Spark APP") // app name
conf.set("spark.master", "local[4]") 
conf.set("spark.ui.port", "36000") // 重载默认端口

val sc = new SparkContext(conf)
```
2. 可以通过启动标记进行配置

```bash
$ bin/spark-submit \
    --class com.example.MyApp \
    --master local[4] \
    --name "My Spark App" \
    --conf "spark.ui.port=36000 \
    Myapp.jar
```

3. 还可以通过从配置文件`conf/spark-default.conf`读取，文件所在位置也能通过启动标记修改。
```bash
## Contents of my-config.conf ##
spark.master local[4]
spark.app.name "My Spark App"
spark.ui.port 36000

$ bin/spark-submit \
    --class com.example.MyApp \
    --properties-file my-config.conf \
    Myapp.jar
```

上面几种优先顺序为<font color='red'>1 > 2 > 3</font>

常用配置
| 选项 | 默认 | 描述 |
|:----:|:----:|:----:|
|spark.executor.memory</br>(--executor-memory)| 512m | 每个执行器进程分配的内存|
|spark.executor.cores</br>(--executor-cores)| 1 | 限制应用使用的核心个数 |
|spark.cores.max</br>(--total-executor-cores)|  | 所有执行器使用核心总数上限 |
|spark.speculation | false | 任务预测执行机制，</br>遇到比较慢的任务会另起节点 |
|spark.storage.</br>blockManagerTimeoutIntervalMs | 45000 | 判断执行器存活的心跳间隔</br>`新版会被统一超时参数取代` |
|spark.executor.extraJavaOptions |  | 启动执行器额外的参数|
|spark.executor.extraClassPath |  | 启动执行器额外类库|
|spark.executor.extraLibraryPath |  | 启动执行器额外的程序库|
|spark.serializer | org.apache.spark.</br>serializer.JavaSerializer | 指定用来序列化的类库，默认较慢</br>追求速度可以用`Kyro`|
|spark.[x].port | 任意 | 运行应用时用到的端口|
|spark.eventLog.enabled | false | 是否开启实践日志机制|
|spark.eventLog.dir | `file:///tmp/spark-events` | 日志机制路径 |

> `SPARK_LOCAL_DIRS`环境变量在`conf/spark-env.sh`中，可以指定Spark用来混系数据时的本地存储路径。

## 8.2 Spark执行的组成部分： 作业,任务和步骤
> 对于 spark 的 rdd 可以通过`rdd.toDebugString`查看其谱系。

- 作业: 特定行动操作生成的步骤集合（计算图）
- 步骤：一个步骤对应计计算图中的一个或者多个RDD启动很多个任务。
- 任务： 在不同的数据分区上同样的操作

## 8.3 查找信息
### 8.3.1 Spark 网页用户界面
在驱动器程序机器的`4040`端口上。
### 8.3.2 驱动器进程和执行器进程的日志

## 8.4 关键性能考量
### 8.4.1 并行度
两种方法控制并行度，

1. 在数据混洗时使用参数为RDD指定并行度。
2. 对已有RDD使用`repartition`或者`coalesce`重新分区。

### 8.4.2 序列化格式

使用`kyro`序列化会更快。但是未注册类会报错。

### 8.4.3 内存管理

内存使用3部分：

- RDD存储: 默认60%

- 数据混洗聚合缓存: 默认20%

- 用户代码: 默认20%

> 使用`MEMORY_ONLY_SER`等方法可以在存储时用序列化方法存储有效降低垃圾回收次数。但是会增加额外的序列化时间。

## 8.4.4 硬件供给
`spark.local.dir`可以映射多个磁盘获得更好的IO速度。
