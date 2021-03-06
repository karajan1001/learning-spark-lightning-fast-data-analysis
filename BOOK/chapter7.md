# 第七章 在集群上运行Spark

## 7.1 简介
可以在本地用小数据集快速开发验证，然后无需修改代码就可以在大规模数据集上运行。

## 7.2 Spark运行时架构
主从结构: 
- 主：驱动器（Driver）节点
- 从：执行器（executor)节点
- 集群管理器：

驱动器+执行器 + 集群管理器 = Spark 应用（Application）

### 7.2.1 驱动器节点
当使用shell模式时，启动hell创建驱动器程序，结束时销毁驱动器程序。驱动器节点的两个职责：
- 把用户程序转为任务： 创建出一个操作组成逻辑上的有向无环图。然后对逻辑执行计划优化。并将任务分发给集群。
- 为执行器节点调度任务：有了物理执行计划后。分配任务，尽量减少数据的网络传输。

### 7.2.2 执行器节点
是一种工作进程，负责在Spark作业中运行任务，任务间相互独立。主要负责：
1. 运行任务，返回结果。
2. 通过块管理器为用户程序中要求缓存的RDD提供内存式存储。

> 本地模式下执行器和驱动器在同一个JAVA进程。

### 7.2.3 集群管理器
用来管理执行器的启动，某些情况下也会管理驱动器的启动，有三种集群管理的方法：
- **自带**: 内部
- **YARN**: 外部
- **Mesos**：外部


### 7.2.4 启动一个程序
不管使用哪种集群管理器都可以使用Spark提供的统一脚本`spark-submit`将应用提交到集群管理器上。

### 7.2.5 小结
完整的spark运行流程为
1. 用户通过`spark-submit`提交应用。
1. 脚本启动驱动器程序，调用`main`方法。
1. 驱动器程序与集群管理器通信，申请资源启动执行器节点。
1. 集群管理器为驱动器程序启动执行器节点。
1. 驱动器进程执行用户应用中的操作。将工作以任务的形式发送到执行器进程。
1. 执行器程序中进行计算并保存结果。
1. 如果`main`方法退出或者调用`stop`，驱动器程序就会终止执行器进程，并释放资源。

## 7.3 使用spark-submit部署应用

spark为集群管理器提供了统一工具提交作业
本地运行：
```bash
bin/spark-submit my_script.py
```

集群运行：
```bash
bin/spark-submit --master spark://host:7077 --executor-memory 10g my_script.jar // 使用spark 独立集群
bin/spark-submit --master yarn --executor-memory 10g my_script.jar // 使用yarn管理
```

一些参数选择：
1. --master 连接的集群管理器
1. --depley-mode 使用本地`client`启动驱动器程序，还是在集群`cluster`上启动驱动器程序
1. --class 运行scala时应用的主类
1. --name 应用的显示名。会在界面中展示。
1. --jars 需要上传放到应用的classpath中的JAR包，只适合少量。
1. --files 需要放到应用工作目录中的文件列表
1. --py-files `--jars`的python对应版本。
1. --executor-memory 执行器使用的内存比如`512m`,`15g`
1. --driver-memory 驱动器使用的内存

## 7.4 打包代码与依赖
python代码可以在工作节点上的python环境安装。但是要防止和已经有的包发生冲突。对于java和scala如果依赖少可以通过`--jars`提交独立jar包依赖，如果多是使用构建工具，生成单个大JAR包。包含所有依赖。
### 7.4.1 maven构建java
### 7.4.2 sbt构建scala
### 7.4.3 依赖冲突
可以使用`shading`方式打包应用。
## 7.5 Spark应用内与应用间调度
有一系列的调度策略保证资源不会被过度使用，还可以设置优先级。内部的公平调节器也会让长期运行的应用定义调度任务的优先级队列。
## 7.6 集群管理器
只运行spark则用自带独立管理器即可，如果要和其他分布式应用共享集群，则可以使用开源集群管理器。

### 7.6.1 独立集群管理器
1. 启动独立集群管理器。
  - 将spark复制到同一个目录
  - 设置好主节点到其他机器的无密码登录。
  - 编辑主节点的conf/slaves文件并填上所有节点的主机名。
  - 在主节点上运行sbin/start-all.sh。就可以在`http://masternode:8080`上看到集群管理器的网页用户界面。
  - 在主节点上sbin/stop-all.sh 停止集群。
2. 提交应用
  - 需要把`spark://masternode:7077`作为参数船体给`submit`或者`shell`。
3. 配置资源用量
  - 执行内存: --executor-memory
  - 占用核心: --total-execotorcores。
4. 高度可以用性。
  - 工作节点故障自然会被解决。
  - 可以使用ZooKeeper维护多个备用主节点。

### 7.6.2 Hadoop YARN 

只需要设置指向Hadoop配置目录的环境变量。然后使用spark-submit向特殊的主节点url提交作业。

```bash
export HADOOP_CONF_DIR='...' # 包含了yarn-site.xml 的目录
spark-submit --master yarn app_name
```

### 7.6.3 Apache Mesos

```bash
spark-submit --master mesos://masternode:5050 app_name
```

1. Mesos调度
2. 客户端和集群模式
3. 配置资源用量

### 7.6.4 Amazon EC2
## 7.7 选择合适的集群管理器
- 从零开始，选择独立集群管理器。
- 如果需要调用其他应用中的资源，YARN或者Mesos。YARN更好。
- Mesos 细粒度共享。
- 最好运行在HDFS节点上方便访问存储。

## 7.8 总结
