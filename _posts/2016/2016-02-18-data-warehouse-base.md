--- 
categories: [Data Warehouse]
tags: [updating]
---

* OLTP：联机事务处理
* OLAP：联机分析处理

## Datawarehouse

### ODS：Operation Data Store

数据贴源层，通常直接保存从外部导入的数据，不做任何处理

### DWD：Data Warehouse Detail

数据明细层，对ODS层的数据进行清洗，过滤掉脏数据、如缺失关键值，或数据值明显错误等，形成可用的干净数据

另一方面可以将ODS层的不同来源的数据按照规范统一后，使其具有一致的数据格式和规范，保存到同一处仓库也便于统一计算

### DM/DWM/DWB：Data Warehouse Basis

数据基础层，也有称DM DWM（Data Warehouse Middle）保存轻度汇总数据，属于DWD向DWS过渡层次，提高数据复用度

### DWS：Data Warehouse Service

以主题域建立的宽表，都是已经汇总好的数据，按照业务划分，生成字段比较多的宽表，提供后续业务查询，如OLAP分析和数据分发

### ADS：Application Data Service

应用数据服务层，可直接用于应用的数据，通常放到Redis、ES等能满足快速响应的数据存储中，重点满足查询性能要求

### Data Model

数据模型，将数据按照一定理论思想进行拆分、组合、使其形成一系列数据结构，可以更容易、高效满足最终工作目标并具备有很好的扩展性，这个设计过程可称之为数据建模。例如为满足业务事务的业务表建模，有三范式理论。为满足分析的数据仓库建模，有维度建模理论。

> 未完待续...
{: .prompt-info}