---
title: Hive 任务调优
date: 2015-10-23 13:40:00 +0800
categories: [BigData, Hive]
tags: [performance]
---

最近在做 Hive 集群搭建配置，记录对任务执行性能影响较大的参数

```bash
#每个Map最大输入大小
set mapred.max.split.size=256000000;

#一个节点上split的至少的大小 
set mapred.min.split.size.per.node=100000000; 

#一个交换机下split的至少的大小
set mapred.min.split.size.per.rack=100000000; 

#执行Map前进行小文件合并
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;

#在Map-only的任务结束时合并小文件
set hive.merge.mapfiles = true; 

#在Map-Reduce的任务结束时合并小文件
set hive.merge.mapredfiles = true; 

#合并文件的大小
set hive.merge.size.per.task = 256000000; 

#当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
set hive.merge.smallfiles.avgsize=16000000; 
```