---
title: Hadoop HDFS Basis
categories: [Big data, Hadoop]
tags: [architecture]
---
Hadoop 包含三大组件分别是：HDFS（分布式文件系统）、MR（分布式计算引擎）、YARN（分布式资源管理）

## HDFS

HDFS（Hadoop Distributed File System），名为Hadoop分布式文件系统，__具有高容错、高吞吐量、容易扩展、高可靠性__ 的特征，是Hadoop生态重要的存储系统。

HDFS是主从结构的分布式系统，一个典型的HDFS系统包含一个Namenode和几个Datanode组成，用户通过Client和相应的配置同Namenode和Datanode交互访问文件系统.

HDFS以数据块为基本存储单位，一个数据块的默认大小ver1.x为64m，ver2.x为128m，ver3.x为256m，写入HDFS的文件在写满一个数据块后再写下一个数据块，

### Namenode

Namenode保存命名空间信息，包含了系统的文件目录树、文件/目录信息以及文件的数据块索引。同时还保存有数据块与数据节点的对应关系，这部分使用时常驻内存中，由启动时动态构建，为避免关机断电后的数据丢失在磁盘上肯定也有存储，这就是两个重要文件`fsimage`及`editlog`。

Namenode支持HA，此时共两个节点，一个是Active Namenode，一个是Standby Namenode。Active Namenode负责处理HDFS系统的所有客户端请求，读写响应、与Datanode通讯等。Standby Namenode会实时同步(通过JournalNode)Active Namenode的命名空间数据，保持与Standby Namenode一致，一但发现 Active Namenode出现故障，Standby Namenode会升级成Active Namenode保障集群功能正常运行。

### fsimage & editlog

fsimage保存截止某个时刻的全量命名空间的数据信息，而editlog保存此时刻之后一系列变更和修改，将两者合并就可以得到截止当前时刻的命名空间最终状态。fsimage相当于全量数据，保存的是最终状态，editlog相当于是流水数据保存了过程状态，可知editlog文件大小是随着系统使用时间不断增长的，为了避免文件过大必须在一个周期后将editlog里的记录与fsimage进行合并，新成一个新的fsimage文件。在非HA架构中fsimage和editlog文件的合并由Namenode完成，在HA架构中合并操作可由备份节点完成，备份节点在运行时不断同步editlog文件记录(JournalNode机制)，在合适的时间进行fsimage和editlog文件的合并，然后将合并后的fsimage同步给活动节点，活动节点检验后替换旧的fsimage文件并进行加载，活动节点的工作更轻松、系统运行更顺畅了。

### JournalNode

JournalNode是在集群中启动的逻辑节点，通常是在某些Datanode上启动的进程，JournalNode目的是为了让Standby Namenode与Active Namenode保持同步，Active Namenode对命名空间所做的修改会持久化到editlog文件当中，而文件必须要同步到N-(N-1)/2个JournalNode节点才能保证安全性，换言之集群有3个JournalNode最多充许1个挂掉，有5个JournalNode最多充许2个挂掉，一但Active Namenode出现故障，Standby Namenode可以从JournalNode中读取全部editlog，切换到Active，保障安全的故障转移。

### Datanode

Datanode主要功能是负责文件读写，会按照一定间隔向Namenode发送心跳、数据块汇报以及缓存汇报；Namenode则会响应心跳包，响应中有可能带有创建文件、删除文件或者复制文件的命令。

### 写文件简略流程

1. 调用Client的API创建文件准备开始写文件
2. Clinet向Namenode发送请求创建文件
3. Namenode首先对请求进行合法性验证
4. Namenode根据最新的Datanode状态信息（网络延时、负载、可用磁盘空间）等信息，给Client返回创建的文件ID及所在节点、数据块索引等信息
5. client通过返回信息建立与Datanode的通讯
6. 成功后Clinet与Datanode建立流式通道并开始写入文件数据
7. 写入完成后Client关闭与Datanode的网络链接
8. Datanode按照复制策略将刚刚写入的文件复制到另外两个节点（默认2副本）

### 读文件简略流程

1. 调用Client的APi准备开始读文件
2. Client向Namenode发送请求读取文件(HadoopRPC)
3. Namenode查询命令空间，如果没有找到文件则返回异常，如果找到文件则根据 `文件 > 数据块 > Datanode` 的索引将数据返回
4. Client 与 Datanode 进行网络通讯
5. 网络通讯成功后，Client 与 Datanode 建立流式通道读取文件数据
6. 完成后关闭 Client 与 Datanode 的流式通道
7. Datanode 将本次访问情况记录并上报Namenode

### 高容错性

Namenode支持HA降低单点故障风险，文件3副本机制保障不会由于个别Datanode节点故障导致文件丢失

### 高吞吐量

文件读写IO在Client与Datanode之间进行，由于Datanode通常有多个可以均摊负载，一个文件分为多个数据块也可以支持并行读取，文件数据读写基于流式接口执行有利于批量数据处理同时提高了吞吐量

### 容易扩展

Datanode支持横向扩展，由此增加存储能力。Namenode可以通过切换到Sandby的方式调整节点硬件配置。

### 高可靠性

Namenode 命名空间数据都写入磁盘文件fsimage，存储全量的命名空间数据，editlog保存命名空间修改日志，每间隔一段时间对两个文件进行合并，产生新的fsimage文件。JournalNode分布式多进程同步Active Namenode的editlog日志，在故障转移过程中由JournalNode保障editlog的完全同步，保障Namenode的故障转移。Datanode中存储的文件默认存储另外2个副本到其它Datanode，保障文件数据不会由个别Datanode故障而丢失。。

### 租约管理

HDFS文件是write-once-read-many，因此不支持客户端的并行写操作，那么这里就需要一种机制保证对HDFS文件的互斥操作。HDFS使用租约（lease）机制来实现这个功能，租约就是HDFS系统给Client一段时间内对某文件的独占锁定，客户端在完成写入操作关闭文件时即释放租约，如果在租约期限内未完成操作，则需要进行续约，否则会由HDFS收回租约从而写入失败。

> 租约有软限制（softLimit）和硬限制（hardLimit），软限制为默认60秒可以在集群配置中进行配置，硬限制为60分钟无法更改
{: .prompt-tip}
