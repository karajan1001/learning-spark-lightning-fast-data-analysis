# 第十章 Spark Streaming

Streaming 使用 Dstream作为抽象表示，这是一种随时间推移而受到的数据的序列。内部每个时间区间受到的Dstream都是一个RDD。DSteaming是RDD组成的序列。Dstreaming的来源多种多样有`Flume`,`Kafka`或者`HDFS`。

（似乎流式计算用`Storm`更为普遍，所以略去本节)
