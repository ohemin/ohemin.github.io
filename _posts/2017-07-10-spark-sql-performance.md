---
title: Spark 程序优化一例
date: 2017-07-10 09:40:00 +0800
categories: [BigData, Spark]
tags: [performance]
---

Spark SQL 很🔥，官方宣称开发效率大幅提高，并且程序逻辑更容易理解，然而实际使用起来却也有运行效率低下，优化手段匮乏等缺点。传统数据库采用SQL，提高查询速度很大程度上依赖于索引手段进行SQL优化，而Spark SQL中的表无法使用索引技术。

Spark 应用基于内存计算，对于聚合类型计算 shuffle 过程难以避免，此时数据传输速度下降几个数量级。应着重减少 shuffle 次数以及 shuffle 数据量。

支撑公司当前业务的 Spark 程序计算速度不达预期，因此着手进行优化。

## 优化目标

稳定性，能力，速度，资源占用

## 优化结果

每5分钟时间窗口数据计算时间由 >90s 下降至 <30s

## 实际优化策略

* 删除数据类，减少 shuffle 过程中序列化反序列化耗时，同时减少内存占用，删除BigScreenIndex case class
* transformation 阶段即过滤掉不参与计算的数据，即优先过滤数据。例：过滤 dmlType = delete 或 dmlType = remove 的数据
* 减少计算次数，重用计算中间结果。例：业务sql 与 topic对应关系只需要计算一次
* 尽量减少spark sql的使用，因为 spark sql 为动态编译加载技术，运行上需要走分析器、优化器、转换为原生API那套逻辑，最后实现还是基于RDD数据结构，做为7x24小时运行的应用来说，节省的那点开发时间微不足道，示例：由于SQL语法限制，原有程序逻辑使用了5条SQL才实现业务需求，改为 streaming 处理代码后，仅1次数据转换就得到所属数据结构

### 详细业务逻辑及优化过程

1. 业务数据计算
2. 增量数据 ```topic list: select distinct topic from event_table```
3. 增量数据合并
```sql
SELECT title, type, smalltype, sum(valuelong) as valuelong
        FROM (
          SELECT title, type, smalltype, valuelong FROM old_table
          UNION ALL
          SELECT title, type, smalltype, valuelong from increment_table
        ) t GROUP BY title, type, smalltype
```
4. 指标比率计算
```sql
SELECT t1.title, t1.type, t1.smalltype, t1.valuelong, round((t1.valuelong / t2.valuelong) * 100) as valuedouble
        from merged_table t1 LEFT JOIN (
          select t.type, nvl(t.smalltype,'') as smalltype, sum(valuelong) as valuelong from merged_table t
          where t.type is not null
          group by t.type, t.smalltype
          ) t2 on t1.type = t2.type and nvl(t1.smalltype,'') = t2.smalltype
```
5. 新增及变更记录区分
```sql
select o.bigscreendataid, n.title, n.type, n.smalltype, n.valuelong, n.valuedouble,
          if(o.bigscreendataid is null, 'append',
            if(n.valuelong!=o.valuelong, 'changed',
              if(nvl(n.valuedouble, 0) != nvl(o.valuedouble,0), 'changed', 'none')
            )
          ) as status
        from new_table n left join jdbc_table o
            on nvl(n.title,'') = nvl(o.title,'')
               and nvl(n.type, '') = nvl(o.type, '')
               and nvl(n.smalltype, '') = nvl(o.smalltype, '')
```
6. 优化后的代码
```scala
public class BigScreenData() {
  public BigScreenData() {}
}
```

## 心得

* 在一次map阶段即转换好所需要的字段数据，以便重用，而不是在用到的时候再计算，造成多次计算, 例如id, timestamp, 在转换出new RDD[Row]时就计算好了，在loadHDFS, 和loadDB中直接使用，而非在loadDB中再计算一次
* 实现效果相同前提下，选择更优算法
* 关注spark ui, 对运行参数调整, driver, executor, memory, cores, receiver.maxRate
