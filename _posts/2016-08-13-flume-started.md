---
title: Apache Flume
date: 2016-08-13 01:29:00 +0800
categories: [BigData, Flume]
---

Flume 是大数据 Hadoop 生态的日志集成组件，通过它可以将其它服务器上的日志文件集成到 Hadoop HDFS、HIVE、ES等，Flume 是pull架构主要分为三层，source、 channels、sink

## sources

sources 负责数据的采集部分，在配置文件中以 "source." 开头的即是相关配置，通常可以配置它的类型，采集后送往哪些 channels ，高阶点的可以配置 n 个拦截器，对采集到的数据进行过滤转换等，官方提供了常见的过滤器实现，直接配置相关参数即可，如无法满足需求可以自己编写过滤器实现更灵活功能。

## channels

channels 顾名思义是数据通道，采集到的数据由此发送到不同消费端，建议采用 memory 类型，其实类型均会不同程度降低吞量。

## sinks

sinks 就是消费端，消费采集到的数据，通常情况存到 HDFS，通过对目录及文件地址的设计可以直接存为HIVE目录下的HDFS文件，比起 type=hive 的 sink 更快，但要注意使用前对HIVE元数据表进行刷新加载这些文件数据。

## features

Flume集群收集的日志发送到hdfs上建立文件夹的时间依据是根据 `event` 时间，既日志传入Flume服务器的时间，源代码实现上是 `Clock.unixTime()`，所以如果想要根据日志自身生成的时间来建立文件夹的话，需要对 `com.cloudera.flume.core.EventImpl` 类的构造函数 

public EventImpl(byte[] s, long timestamp, Priority pri, long nanoTime, String host, Map<String, byte[]> fields)

重写，解析方法参数 `byte[] s` 的内容取出时间，赋值给 timestamp，这样在 Flume sink 配置文件中 timestamp 才可以作为一个字段被识别和处理，由此解决根据日志生成时间存储等相关问题。

> flume的框架会构造 `byte[] s 长度为 0` 的数组，用来发送类似简单验证的 event，所以在抽取日志生成时间时需要注意 s 长度为 0 的问题。
{: .prompt-tip }

## HDFS Sink 配置实例

```conf
############################################
#  producer config
############################################
#agent section
producer.sources = s
producer.channels = c c1 c2
producer.sinks = r h es

#source section
producer.sources.s.type =exec
producer.sources.s.command = tail -f /usr/local/nginx/logs/test1.log
#producer.sources.s.type = spooldir
#producer.sources.s.spoolDir = /usr/local/nginx/logs/
#producer.sources.s.fileHeader = true

producer.sources.s.channels = c c1 c2

producer.sources.s.interceptors = i
#不支持忽略大小写
producer.sources.s.interceptors.i.regex = .*\.(css|js|jpg|jpeg|png|gif|ico).*
producer.sources.s.interceptors.i.type = org.apache.flume.interceptor.RegexFilteringInterceptor$Builder
#不包含
producer.sources.s.interceptors.i.excludeEvents = true

############################################
#   hdfs config
############################################
producer.channels.c.type = memory
#Timeout in seconds for adding or removing an event
producer.channels.c.keep-alive= 30
producer.channels.c.capacity = 10000
producer.channels.c.transactionCapacity = 10000
producer.channels.c.byteCapacityBufferPercentage = 20
producer.channels.c.byteCapacity = 800000

producer.sinks.r.channel = c

producer.sinks.r.type = avro
producer.sinks.r.hostname  = 127.0.0.1
producer.sinks.r.port = 10101
############################################
#   hdfs config
############################################
producer.channels.c1.type = memory
#Timeout in seconds for adding or removing an event
producer.channels.c1.keep-alive= 30
producer.channels.c1.capacity = 10000
producer.channels.c1.transactionCapacity = 10000
producer.channels.c1.byteCapacityBufferPercentage = 20
producer.channels.c1.byteCapacity = 800000

producer.sinks.h.channel = c1

producer.sinks.h.type = hdfs
#目录位置
producer.sinks.h.hdfs.path = hdfs://127.0.0.1/tmp/flume/%Y/%m/%d
#文件前缀
producer.sinks.h.hdfs.filePrefix=nginx-%Y-%m-%d-%H
producer.sinks.h.hdfs.fileType = DataStream
#时间类型必加，不然会报错
producer.sinks.h.hdfs.useLocalTimeStamp = true
producer.sinks.h.hdfs.writeFormat = Text
#hdfs创建多长时间新建文件，0不基于时间
#Number of seconds to wait before rolling current file (0 = never roll based on time interval)
producer.sinks.h.hdfs.rollInterval=0
#hdfs多大时新建文件，0不基于文件大小
#File size to trigger roll, in bytes (0: never roll based on file size)
producer.sinks.h.hdfs.rollSize = 0
#hdfs有多少条消息时新建文件，0不基于消息个数
#Number of events written to file before it rolled (0 = never roll based on number of events)
producer.sinks.h.hdfs.rollCount = 0
#批量写入hdfs的个数
#number of events written to file before it is flushed to HDFS
producer.sinks.h.hdfs.batchSize=1000
#flume操作hdfs的线程数（包括新建，写入等）
#Number of threads per HDFS sink for HDFS IO ops (open, write, etc.)
producer.sinks.h.hdfs.threadsPoolSize=15
#操作hdfs超时时间
#Number of milliseconds allowed for HDFS operations, such as open, write, flush, close. This number should be increased if many HDFS timeout operations are occurring.
producer.sinks.h.hdfs.callTimeout=30000
```

## 乱入几个 Hadoop HDFS 相关配置

在 Flume 向 HDFS 系统持续写入数据时，HDFS的一些配置会影响实际的目录及文件生成，需要加入注意

```conf
hdfs.round=false
#Should the timestamp be rounded down (if true, affects all time based escape sequences except %t)

hdfs.roundValue=
#Rounded down to the highest multiple of this (in the unit configured using hdfs.roundUnit), less than current time.

hdfs.roundUnit='second'
#The unit of the round down value - second, minute or hour.
```

## ES Sink 配置实例

```conf
############################################
#   elasticsearch config
############################################
producer.channels.c2.type = memory
#Timeout in seconds for adding or removing an event
producer.channels.c2.keep-alive= 30
producer.channels.c2.capacity = 10000
producer.channels.c2.transactionCapacity = 10000
producer.channels.c2.byteCapacityBufferPercentage = 20
producer.channels.c2.byteCapacity = 800000

producer.sinks.es.channel = c2

producer.sinks.es.type = org.apache.flume.sink.elasticsearch.ElasticSearchSink
producer.sinks.es.hostNames = 127.0.0.1:9300
#Name of the ElasticSearch cluster to connect to
producer.sinks.es.clusterName = sunxucool
#Number of events to be written per txn.
producer.sinks.es.batchSize = 1000
#The name of the index which the date will be appended to. Example ‘flume’ -> ‘flume-yyyy-MM-dd’
producer.sinks.es.indexName = flume_es
#The type to index the document to, defaults to ‘log’
producer.sinks.es.indexType = test
producer.sinks.es.serializer = org.apache.flume.sink.elasticsearch.ElasticSearchLogStashEventSerializer
```