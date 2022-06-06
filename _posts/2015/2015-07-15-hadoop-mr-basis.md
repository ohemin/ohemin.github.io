---
title: Hadoop MR Basis
categories: [Big data, Hadoop]
---
Hadoop 包含三大组件分别是：HDFS（分布式文件系统）、MR（分布式计算引擎）、YARN（分布式资源管理）

## MR (MapReduce)

MR是Hadoop中的计算模型，主要由三个阶段组成：Map > shuffle > Reduce。

Map阶段是对数据进行拆分(split)，将数据转化为键值对；

Reduce是合并，将具有相同的Key的Value进行聚合，最终输出全部聚合之后的结果。

shuffle阶段包含Map shuffle和Reduce shuffle。Map shuffle即是在Map端的shuffle，Reduce shffle即是在Reduce端的shuffle，我们在编写MR程序时，只需要编写Map逻辑代码和Reduce逻辑代码，shuffle是由系统自动实现。

Map shuffle 对 Map拆分好的键值对进行分区，将同一分区的数据进行聚合、排序，得到的结果按照一个分区一个文件写入磁盘。

Reduce shuffle 读取 Map shuffle写好的分区文件，将多个分区文件按照相同 key 值进行聚合，形成一个个键值对，所不同的是这次的值其实是多个值组成的数组。这些结果将做为Reduce阶段的输入参数进行处理。