---
title: Apache Sqoop 命令实战
date: 2016-08-13 11:29:00 +0800
categories: [Big data, Sqoop]
tags: [command]
---

最近 sqoop 用得比较多，在此把用到的命令记录一下

官方文档地址 1.4.6：<http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html>

## 创建 Sqoop 任务
```bash
sqoop job --create append-import-goods_source_search -- \
  --options-file sqoop-conf/import-hdfs.conf \
  --append \
  --table GoodsSourceSearch \
  --target-dir /user/hive/warehouse/mirror.db/goods_source_search/ \
  --check-column inputDate \
  --incremental append \
  --password ******
```

## 创建 Sqoop 全量任务，执行时覆盖已有数据
```bash
sqoop job --create import-auth_user -- \
  --options-file sqoop-conf/import-hive.conf \
  --table AuthUser \
  --hive-table auth_user \
  --password ******
```

## Sqoop 增量导入

```bash
sqoop job --create append-import-call_record -- \
  --options-file sqoop-conf/import-hdfs.conf \
  --append \
  --table CallRecord \
  --target-dir /user/hive/warehouse/mirror.db/call_record/ \
  --check-column inputDate \
  --incremental append \
  --password ******
```

## 导入文件格式

sqoop的hive导入低层文件类型只支持textfile格式，hdfs导入则支持textfile、sequencefile、avrofile三种格式，因此如果希望导入数据到hive表最简单的是选择textfile格式，如果对hive表数据处理性能有要求需要至少两步操作，如果hive表选择sequencefile或avrofile格式，可以参数指定对应类型后使用hdfs导入，然后hive表刷新元数据使其加载新导入文件，如果使用rcfile/orcfile格式，可textfile格式导入表后用HIVE语句"insert into table testtable select * from textfile_table"导入 __或者开发Sqoop扩展完成对格式支持__

> 使用avrofile等格式时，其数据表结构信息是存储在该Hive元数据表字段中的，MySQL的varchar2类型字段有最大长度，当数据表的字段多了以后整个数据表结构描述信息的字符串长度将超出MySQL varchar2类型字段最大长度，而且由于MySQL中行记录数据长度也存在限制，实际数据表结构描述可用长度远远不到varchar2最大长度上限，字段多的数据表将在建表时因结构描述信息被截断无法正常解析，实践中此问题多发于Sqoop自动导入建表过程中。
{: .prompt-warning }

分析以上两种方案，其中sequencefile和avrofile格式io等资源消耗较少，只需要写一次hdfs然后加载数据即可，rcfile,orcfile相比较而言多一次格式转换和写文件操作，如果表需要多次查询使用则牺牲导入时间换取后续表数据使用时的高效


> 同数据量文件大小排序：sequencefile > textfile > avrofile > rcfile
> 
> 同数据量查询负载排序：textfile > sequencefile > avrofile > rcfile
{: .prompt-tip }


## 创建sqoopJob追加数据到HDFS,同时对已有数据做更新:

```bash
sqoop job --create increment-import-repport-goods_source  --meta-connect jdbc:hsqldb:hsql://big-bigworker-6.100.idc.tf56:16000/sqoop -- --options-file sqoop-conf/import-hdfs.conf --table GoodsSource --target-dir /user/hive/warehouse/increment.db/report_goods_source/ --merge-key goodsSourceId --check-column updateDate --incremental lastmodified --password ******
```

## 创建sqoopJob只是追加数据到HDFS,不对会已有数据做更新

```bash	
sqoop job --create append-import-party_log -- --options-file sqoop-conf/import-hdfs.conf --append --table PartyLog --target-dir /user/hive/warehouse/mirror.db/party_log/ --check-column inputDate --incremental append --password ******

sqoop job --list  --meta-connect jdbc:hsqldb:hsql://big-bigworker-6.100.idc.tf56:16000/sqoop
	
sqoop job --create increment-export-report-log_yunbao_lujing_app_dau -- --options-file sqoop-conf/export.conf --table QLujingAppDriverAndShipperDau --update-key inputDate --update-mode allowinsert --export-dir /hive/warehouse/increment.db/report_lujing_app_dau
	
sqoop job --create merge-import-repport-red_packet_detail  --meta-connect jdbc:hsqldb:hsql://big-bigworker-6.100.idc.tf56:16000/sqoop -- --options-file sqoop-conf/import-hdfs.conf --table DimParty --target-dir /user/hive/warehouse/report.db/red_packet_detail/ --merge-key redPacketDetailId --check-column updateDate --incremental lastmodified --password ******
```