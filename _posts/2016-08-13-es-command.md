---
title: ElasticSearch 命令实战
date: 2016-08-13 01:29:00 +0800
categories: [BigData, ElasticSearch]
tags: [command]
---

记录一下 elasticsearch 高频命令


## 添加删除模版 

```bash
curl -XDELETE 'http://[server_host]:9200/goods-source-log'
curl -XPUT 'http://[server_host]:9200/_template/goods-source-log' -d '@goods-source-log-v1.json'
```
## 删除索引  
会将整个索引包括数据全部删除
```bash
curl -XDELETE 'http://[server_host]:9200/goods-source-log'
```

## 查找数据
```bash
curl -XPOST 'http://[server_host]:9200/goods-source-log/_search?q=*&pretty'
```