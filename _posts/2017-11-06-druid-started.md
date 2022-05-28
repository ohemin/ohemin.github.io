---
title: Apache DRUID 部署及使用
date: 2017-11-06 20:55:00 +0800
categories: [Big data, Druid]
toc: true
---
## 初始设置

### 修改时区

`druid/conf/druid/broker/jvm.config` 里的 `UTC` 修改为 `Asia/Shanghai`

### HDFS数据加载支持

修改 `druid/conf/druid/_common/common.runtime.properties` , 确保 `druid.extensions.loadList` 中有 `druid-hdfs-storage`，在 `druid/conf/druid/_common/` 目录中添加以下hadoop配置文件 的软链接 `core-site.xml, hdfs-site.xml, yarn-site.xml, mapred-site.xml`

## 启动分布式服务

### 测试环境示例

DRUID 部署在同一服务器 `jt-host-kvm-53`

```bash
# 启动 historical
nohup java `cat conf-quickstart/druid/historical/jvm.config | xargs` \
  -cp "conf-quickstart/druid/_common:conf-quickstart/druid/historical:lib/*" \
  io.druid.cli.Main server historical > log/historical.log 2>&1 &

# 启动 broker
nohup java `cat conf-quickstart/druid/broker/jvm.config | xargs` \
  -cp "conf-quickstart/druid/_common:conf-quickstart/druid/broker:lib/*" \
  io.druid.cli.Main server broker > log/broker.log 2>&1 &

# 启动 coordinator
nohup java `cat conf-quickstart/druid/coordinator/jvm.config | xargs` \
  -cp "conf-quickstart/druid/_common:conf-quickstart/druid/coordinator:lib/*" \
  io.druid.cli.Main server coordinator > log/coordinator.log 2>&1 &

# 启动 overlord
nohup java `cat conf-quickstart/druid/overlord/jvm.config | xargs` \
  -cp "conf-quickstart/druid/_common:conf-quickstart/druid/overlord:lib/*" \
  io.druid.cli.Main server overlord > log/overlord.log 2>&1 &

# 启动 middleManager
nohup java `cat conf-quickstart/druid/middleManager/jvm.config | xargs` \
  -cp "conf-quickstart/druid/_common:conf-quickstart/druid/middleManager:lib/*" \
  io.druid.cli.Main server middleManager > log/middleManager.log 2>&1 &
```

### 集群部署示例

参照测试环境命令，将进程部署到不同服务器，本例资源受限根据服务负载进行了部分隔离

|hosts|role|port|
---|---|---
|big-elephant-1|kafka|
|big-elephant-4|broker,overlord|
|big-elephant-5|historical,coordinator,middleManager|

## 启动实时数据消费进程

```bash
nohup bin/tranquility kafka -configFile conf/yugong-status.json &
```

### 查看消费进程 Log

```bash
tail -f http://big-elephant-8:8090/druid/indexer/v1/task/[taskid]/log
```

## 扩展工具

### Druid Dumbo

https://github.com/liquidm/druid-dumbo

* 在HDFS中验证实时生成的原始数据。
* 重建与HDFS中原始数据不一致的细分。
* 将现有的段折叠成更高的粒度段。


## 示例配置文件

这个示例从kafka消息系统中消费消息，将消息中的值以分钟为时间窗口单位，metric_code字段为维度，metric_value为度量字段进行值统计，统计包括最大、最小、合计、总记录数，相当于增强版本 WordCount 程序。

```json
{
  "dataSources": [{
    "spec": {
      "dataSchema": {
        "dataSource": "metric_append",
        "parser": {
          "type": "string",
          "parseSpec": {
            "timestampSpec": {
              "format": "yyyy-MM-dd HH:mm:ss.SSS",
              "column": "timestamp"
            },
            "dimensionsSpec": {
              "dimensions": ["metric_code"]
            },
            "columns": ["metric_code", "metric_value", "timestamp"]
          }
        },
        "granularitySpec": {
          "type": "uniform",
          "queryGranularity": "minute"
        },
        "metricsSpec": [{
          "type": "count",
          "name": "count"
        }, {
          "type": "doubleSum",
          "name": "metric_sum",
          "fieldName": "metric_value"
        }, {
          "type": "doubleMin",
          "name": "metric_min",
          "fieldName": "metric_value"
        }, {
          "type": "doubleMax",
          "name": "metric_max",
          "fieldName": "metric_value"
        }]
      },
      "tuningConfig": {
        "maxRowsInMemory": "10000",
        "type": "realtime",
        "windowPeriod": "PT30M",
        "intermediatePersistPeriod": "PT30M"
      }
    },
    "properties": {
      "topicPattern.priority": "1",
      "topicPattern": "YugongMetricAppend"
    }
  }, {
    "spec": {
      "dataSchema": {
        "dataSource": "metric_complete",
        "parser": {
          "type": "string",
          "parseSpec": {
            "timestampSpec": {
              "format": "yyyy-MM-dd HH:mm:ss.SSS",
              "column": "timestamp"
            },
            "dimensionsSpec": {
              "dimensions": ["metric_code"]
            },
            "columns": ["metric_code", "metric_value", "timestamp"]
          }
        },
        "granularitySpec": {
          "type": "uniform",
          "queryGranularity": "minute"
        },
        "metricsSpec": [{
          "type": "count",
          "name": "count"
        }, {
          "type": "doubleSum",
          "name": "metric_sum",
          "fieldName": "metric_value"
        }, {
          "type": "doubleMin",
          "name": "metric_min",
          "fieldName": "metric_value"
        }, {
          "type": "doubleMax",
          "name": "metric_max",
          "fieldName": "metric_value"
        }]
      },
      "tuningConfig": {
        "maxRowsInMemory": "10000",
        "type": "realtime",
        "windowPeriod": "PT30M",
        "intermediatePersistPeriod": "PT30M"
      }
    },
    "properties": {
      "topicPattern.priority": "1",
      "topicPattern": "YugongMetricComplete"
    }
  }, {
    "spec": {
      "dataSchema": {
        "dataSource": "yugong_writers",
        "parser": {
          "type": "string",
          "parseSpec": {
            "format": "json",
            "timestampSpec": {
              "format": "yyyy-MM-dd HH:mm:ss.SSS",
              "column": "createtime"
            },
            "dimensionsSpec": {
              "dimensions": ["metricCode", "metricDate"]
            }
          }
        },
        "granularitySpec": {
          "type": "uniform",
          "queryGranularity": "minute"
        },
        "metricsSpec": [{
          "type": "count",
          "name": "count"
        }]
      },
      "tuningConfig": {
        "maxRowsInMemory": "10000",
        "type": "realtime",
        "windowPeriod": "PT30M",
        "intermediatePersistPeriod": "PT30M"
      }
    },
    "properties": {
      "topicPattern.priority": "1",
      "topicPattern": "YugongWrite*"
    }
  }],
  "properties": {
    "zookeeper.connect": "10.33.35.192:2181,10.33.35.194:2181,10.33.35.196:2181",
    "zookeeper.timeout": "PT20S",
    "druid.selectors.indexing.serviceName": "druid/overlord",
    "druid.discovery.curator.path": "/druid/discovery",
    "kafka.zookeeper.connect": "10.33.35.192:2181,10.33.35.194:2181,10.33.35.196:2181",
    "kafka.group.id": "tranquility-k",
    "consumer.numThreads": "5",
    "commit.periodMillis": "15000",
    "reportDropsAsExceptions": "false"
  }
}
```

## 以SQL方式访问DURID数据

### 首先下载所需包进行验证测试

```shell
java -cp lib/calcite-core-1.2.0-incubating.jar:lib/avatica-1.8.0.jar:lib/calcite-linq4j-1.2.0-incubating.jar:lib/mysql-connector-java-5.1.25.jar:lib/sqlline-1.4.0-SNAPSHOT-jar-with-dependencies.jar sqlline.SqlLine
```

### 连接druid

```shell
java -cp lib/jackson-annotations-2.8.0.jar:lib/jackson-core-2.8.1.jar:lib/jackson-databind-2.8.1.jar:lib/guava-11.0.2.jar:lib/calcite-core-1.2.0-incubating.jar:lib/calcite-linq4j-1.2.0-incubating.jar:lib/calcite-avatica-1.2.0-incubating.jar:lib/calcite-druid-1.13.0.jar:lib/sqlline-1.4.0-SNAPSHOT-jar-with-dependencies.jar sqlline.SqlLine
```

或者
```shell
java -cp lib/avatica-1.10.0.jar:lib/calcite-core-1.13.0.jar:lib/calcite-druid-1.13.0.jar:lib/calcite-linq4j-1.13.0.jar:lib/commons-compiler-2.7.6.jar:lib/commons-lang3-3.2.jar:lib/guava-18.0.jar:lib/jackson-annotations-2.8.0.jar:lib/jackson-core-2.8.1.jar:lib/jackson-databind-2.8.1.jar:lib/janino-2.7.6.jar:lib/joda-time-2.8.1.jar:lib/sqlline-1.4.0-SNAPSHOT-jar-with-dependencies.jar sqlline.SqlLine
```

然后
```
!connect jdbc:calcite:schemaFactory=org.apache.calcite.adapter.druid.DruidSchemaFactory;schema.url=http://big-elephant-4:8082;schema.coordinatorUrl=http://big-elephant-5:8081;caseSensitive=mysql admin admin
```

### 执行SQL查询

```sql
select sum(count) as v from metric_append where __time > TIMESTAMP '2017-08-10 00:00:00' and metric_code='lj.publishcargo.any.amount' group by metric_code
```

```sql
select metric_sum from metric_complete where __time > TIMESTAMP '2017-08-10 00:00:00' and metric_code='lj.publishcargo.any.amount' order by __time desc limit 1
```
{: file="~/word_count_plus.json"}