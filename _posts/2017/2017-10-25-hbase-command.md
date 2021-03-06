---
title: HBase 命令实战
date: 2017-07-10 09:40:00 +0800
categories: [Big data, HBase]
tags: [command]
---

运维告诉我，HBase集群已经搭建好了，现在来实战一把

## namespace

* 名为 hbase 的 namespace：系统内建表，包括 `namespace` 和 `meta` 表
* 名为 default 的 namespace：用户建表时未指定 `namespace` 的表都创建在此

```sql
-- 创建namespace
create_namespace 'ai_ns'

-- 删除namespace
drop_namespace 'ai_ns'

-- 查看namespace
describe_namespace 'ai_ns'

-- 列出所有namespace
list_namespace

-- 在namespace下创建表
create 'ai_ns:testtable', 'fm1'

-- 查看namespace下的表
list_namespace_tables 'ai_ns'
```

## DDL

```sql
-- 查看有哪些表
list

-- 创建表
create 'YG_ODS:PARTY',{NAME => '0', VERSIONS => 1},{NAME => '1', VERSIONS => 2}

-- 删除表
disable 't1'
drop 't1'

-- 查看表结构
describe 't1'

-- 修改表结构（修改表结构必须先disable）
disable 'test1'
alter 'test1',{NAME=>'body',TTL=>'15552000'},{NAME=>'meta', TTL=>'15552000'}
enable 'test1'
```

## privileges
```sql
-- 语法 : grant <user> <permissions> <table> <column family> <column qualifier> 参数后面用逗号分隔
-- 权限用五个字母表示： "RWXCA".
-- READ('R'), WRITE('W'), EXEC('X'), CREATE('C'), ADMIN('A')
-- 例如，给用户‘test'分配对表t1有读写的权限，
grant 'test','RW','t1'

-- 查看权限
-- 语法：user_permission <table>
-- 例如，查看表t1的权限列表
user_permission 't1'

-- 收回权限
-- 与分配权限类似，语法：revoke <user> <table> <column family> <column qualifier>
-- 例如，收回test用户在表t1上的权限
revoke 'test','t1'
```

## DML

```sql
-- 添加数据
-- 语法：put <table>,<rowkey>,<family:column>,<value>,<timestamp>
-- 例如：给表t1的添加一行记录：rowkey是rowkey001，family name：f1，column name：col1，value：value01，timestamp：系统默认
put 't1','rowkey001','f1:col1','value01'

-- 查询某行记录
-- 语法：get <table>,<rowkey>,[<family:column>,....]
-- 例如：查询表t1，rowkey001中的f1下的col1的值
get 't1','rowkey001', 'f1:col1'
-- 或者：
get 't1','rowkey001', {COLUMN=>'f1:col1'}
-- 查询表t1，rowke002中的f1下的所有列值
get 't1','rowkey001'
```

## 扫描表
```sql
-- 语法：scan <table>, {COLUMNS => [ <family:column>,.... ], LIMIT => num}
-- 另外，还可以添加STARTROW、TIMERANGE和FITLER等高级功能
-- 例如：扫描表t1的前5条数据
scan 't1',{LIMIT=>5}

-- 查询表中的数据行数
-- 语法：count <table>, {INTERVAL => intervalNum, CACHE => cacheNum}
-- INTERVAL设置多少行显示一次及对应的rowkey，默认1000；CACHE每次去取的缓存区大小，默认是10，调整该参数可提高查询速度
-- 例如，查询表t1中的行数，每100条显示一次，缓存区为500
count 't1', {INTERVAL => 10000, CACHE => 50000}

-- 删除行中的某个列值
-- 语法：delete <table>, <rowkey>,  <family:column> , <timestamp>,必须指定列名
-- 例如：删除表t1，rowkey001中的f1:col1的数据
delete 't1','rowkey001','f1:col1'

-- 删除行
-- 语法：deleteall <table>, <rowkey>,  <family:column> , <timestamp>，可以不指定列名，删除整行数据
-- 例如：删除表t1，rowk001的数据
deleteall 't1','rowkey001'

-- 删除表中的所有数据
-- 语法： truncate <table>
-- 其具体过程是：disable table -> drop table -> create table
-- 例如：删除表t1的所有数据
truncate 't1'

-- 查询指定rowKey和指定列数据
get 'mofang:metric_today',
    "\x03\xBC\xD5\x9E03dd85fb223cb2cbc203e518059aa558",
    {
        COLUMNS => [
            'base',
            'minutes'
        ],
        FILTER => "ColumnRangeFilter('1111',true,'1115',true)",
        LIMIT=>20
    }

-- 过滤器使用
scan 'mofang:metric_today',
    {
        COLUMNS => [
            'base',
            'minutes'
        ],
        FILTER => "ColumnRangeFilter('1111',true,'1115',true)",
        LIMIT=>20
    }
```

## Sqoop导入数据到HBase

```bash
sqoop import \
    --connect 'jdbc:mysql://10.7.13.48:8066/trade?tinyInt1isBit=false&useCompression=true&tcpRcvBuf=1024000000&useCursorFetch=true&defaultFetchSize=1000' \
    --table Party \
    --hbase-create-table \
    --hbase-table ODS_PARTY:PARTY \
    --column-family 1 \
    --hbase-row-key partyId \
    --split-by stampDate \
    --username 'admin' \
    --password ****** \
    -m 5

sqoop import \
    --connect 'jdbc:mysql://10.33.60.38:3306/dataMarket?tinyInt1isBit=false&useCompression=true&tcpRcvBuf=1024000000&useCursorFetch=true&defaultFetchSize=1000' \
    --table Person \
    --hbase-create-table \
    --hbase-table ODS_PARTY:PERSON \
    --column-family 1 \
    --hbase-row-key partyId \
    --split-by stampDate \
    --username 'dataMarket' \
    --password ****** \
    -m 5
```

执行完成后查看表结构
```console
root
|-- partyId: integer (nullable = false)
|-- partyType: string (nullable = true)
|-- partyName: string (nullable = true)
|-- mobileNumber: string (nullable = true)
|-- mobileNumberIsActive: string (nullable = true)
|-- email: string (nullable = true)
|-- emailIsActive: string (nullable = true)
|-- tradeType: string (nullable = true)
|-- oldPartyId: integer (nullable = true)
|-- starLevel: string (nullable = true)
|-- inputDate: timestamp (nullable = true)
|-- createDate: timestamp (nullable = true)
|-- status: string (nullable = true)
|-- stampDate: timestamp (nullable = false)
```