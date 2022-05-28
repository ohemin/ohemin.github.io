---
title: Kafka 命令实战2
date: 2018-06-22 01:29:00 +0800
categories: [Big data, Kafka]
tags: [command]
---

Kafka 迭代很快，新版本又有不少变化，同事告诉我升级好了新集群，马上来练练

## 测试环境

### zookeeper: 

服务器名称 ```mt-zookeeper-vip:2181```

``` bash
10.77.32.90 mt-zookeeper-4
10.77.32.91 mt-zookeeper-5
10.77.32.92 mt-zookeeper-6
10.77.32.2 mt-zookeeper-vip
```
{: file="/etc/hosts"}

### broker-server

10.7.31.15:2181,10.7.31.16:2181,10.7.31.17:2181

10.7.31.15:9092,10.7.31.16:9092,10.7.31.17:9092

## 生产环境

### zookeeper
服务器名称 `mt-zookeeper-vip:2181`

### broker-server

10.33.36.101:9092,10.33.36.113:9092,10.33.40.117:9092

PLAINTEXT://10.33.36.101:9092,PLAINTEXT://10.33.36.113:9092,PLAINTEXT://10.33.40.117:9092


## 常用命令

```bash
cd kafka/

# 显示 topic 列表
bin/kafka-topics.sh --list \
  --zookeeper mt-zookeeper-vip:2181

# 创建一个 topic
bin/kafka-topics.sh --create \
  --zookeeper mt-zookeeper-vip:2181 \
  --replication-factor 3 \
  --partitions 1 \
  --topic __connect-offsets

# 删除一个 topic
bin/kafka-topics.sh --delete \
  --zookeeper mt-zookeeper-vip:2181 \
  --topic __connect-offsets

# 消费 kafka 消息，如果加上 --from-beginning 参数则从最早消息开始消费，
# 否则由kafka记录上次位置后开始
bin/kafka-console-consumer.sh \
  --zookeeper mt-zookeeper-vip:2181 \
  --topic bigDatamarket 
  [--from-beginning]

# 以交互方式发送一条 kafka 消息
bin/kafka-console-producer.sh \
  --broker-list [broker server] \
  --topic [topic name]

# 查看某 topic 当前 offset 值
bin/kafka-run-class.sh kafka.tools.GetOffsetShell \
  --broker-list [broker server] \
  --topic lbs_test
```

## 设置offset


```properties
# ...
zookeeper.connect=mt-zookeeper-vip:2181
group.id=dmp_stream
# ...
```
{: file="consumer.properties"}

### 命令行执行如下
```bash
bin/kafka-run-class.sh \
  kafka.tools.UpdateOffsetsInZK \
  latest \
  consumer.properties \
  dmp_task_result
```

### apache canal 推送的mysql记录更新消息结构

```json
{
  "dmlType":"update",
  "pkMap":{
    "partyManageId":"20765"
  },
  "normalMap":{
    "goodsSourceCount":"1612",
    "stampDate":"2017-07-27 10:11:29",
    "updateDate":"2017-07-27 10:11:29",
    "goldDriverStatus":"已开通"
  }
}
```