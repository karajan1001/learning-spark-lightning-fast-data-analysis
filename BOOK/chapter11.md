# 第十一章 基于MLlib的机器学习。

Mlib是Spark中提供机器学习函数库

## 11.1 概述

设计理念:<font color='red'>数据以RDD表示，然后再分布式数据集上调用算法</font>。比如一个文本分类任务：

1. 首先用字符串RDD来表示消息。
2. 运行MLlib中的特征提取算法转换成数值特征；返回一个向量RDD。
3. 调用分类算法了返回一个模型。
4. 使用评估函数在测试集上评估。

## 11.2 系统要求
MLlib需要`gfortran`库。

## 11.3 机器学习基础
（略）

## 11.4 数据类型

- Vector: `mllib.linalg.Vectors`数学向量。既有稠密向量，又支持稀疏向量。
- LabeledPoint: `mllib.regression`中，用来表示带标签的数据点。包含一个特征向量与一个标签。
- Rating: 对一个产品的评分，在`mllib.recoomendation`中用于推荐。
- 各种Model类: 算法结果，一般有一个predict()方法可以用来对数据点进行预测。

### 向量操作
```scala
import org.apached.spark.mllib.linalg.Vectors
val densVe1 = Vectors.dense(1.0, 2.0, 3.0)
val densVe2 = Vectors.dense(Array(1.0, 2.0, 3.0))

val sparaseVec1 = Vectors.sparse(4, Array(0,1), Array(1.0, 2.0)) // 4维，非零位置为0，1，值为1.0 2.0
```

## 11.5 算法

### 11.5.1 特征提取

- TF-IDF: 在mllib.feature中 `HashingTF`和`IDF`。
- 缩放: 使用`StandardScalar`
- 正规化: 使用`Normalizer`
- Word2Vec: 使用`Word2Vec`类进行

### 11.5.2 统计
基本统计在`mllib.stat.Statistics`类中。

### 11.5.3 分类和回归
支持**线性回归**,**逻辑回归**,**SVM**,**朴素贝叶斯**,**决策树和随机森林**。

### 11.5.3 聚类
支持 **K-means**

### 11.5.5 协同过滤和推荐
使用用户和产品概念。用法为交替最小二乘。

### 11.5.6 降维
支持**PCA** **SVD**

### 11.5.7 评估
函数在`mllib.evaluation`包中。

## 11.6一些提示与性能考量
### 11.6.1 准备特征
### 11.6.2 配置算法 
需要配置正则化，迭代次数等。
### 11.6.3 缓存RDD以重复使用
### 11.6.4 识别稀疏度
### 11.6.5 并行度

## 11.7 流水线API

## 11.8 总结




