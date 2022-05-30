---
title: Apache Flume 简单介绍 
date: 2016-06-24 18:39:00 +0800
categories: [Big data, Flume]
---

## 简介

Flume是Cloudera公司研发的高可用、可靠、分布式日志采集系统。Flume由1~n个Agent组成，每个Agent部署到一个主机中，Agent由三大部件构成

![flume structure](/content/flume-structure.jpg)
_Flume组件结构图_

Flume系统有如下特点

1. 由1 ~ n个Agent组成，每个Agent部署到一个主机中
2. 一个Agent就是一个JVM进程，包含Source、Sink、Channel三组件，Source负责日志读取，Channel负责数据暂存，Sink负责数据写入
3. Source、Channel、Sink提供多种类型，不同类型组件可以通过编写的配置文件自由组合，例如Channel类型分为Memory、Disk等，Sink分为HDFS，HBase等
4. 一个主机不需要部署多个Agent，一个Agent可以配置多个Source，对应不同的日志类型或者日志路径
5. Source收集到的日志数据被封装为Event，可能是日志记录，或者avro对象，由具体配置文件配置，后续Channel中暂存也是Events
6. 不同类型或者路径的日志，即可以走各自不同的Channel，也可以合并到同一个Channel中去
7. 一个Channel可以连接一个Sink，也可以连接多个不同Sink，甚至Sink可以连接到另外一个Source，当一个Sink连接到另外一个Source时，可以视为建立了一个多级日志流
8. 三大组件均可以开发自定义类型，由于已经预留扩展接口和采用Java语言，实现起来较为容易

![flume structure multi](/content/flume-structure-multi.jpg)
_Flume构成的多级日志流_

Channel组件是Flume架构设计特点，Fan-in、Fan-out得以实现，赋于整个系统灵活性。

同时这种灵活性并非没有代价：

1. Channel的设计在使用内存的情况下依据日志数据多少会消耗内存，换用磁盘类型Channel传输速度大幅下降
2. Event的设计为基于简单的扩展功能提供很好切入点，但同时也带来序列化反序列化的运算开销和内存占用增加

基于以上优缺点，可知Flume满足以下场景

1. 对日志传输实时性要求不高 (受暂存架构影响)
2. 在使用日志数据时，不需要进行过于复杂的处理（本质只对单记录进行处理）
3. 单主机日志量级不会过大（所有功能部件均在同一主机）
