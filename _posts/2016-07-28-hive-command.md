---
title: Hive 命令实战
date: 2016-07-28 11:29:00 +0800
categories: [BigData, Hive]
tags: [command]
---

## 建表语句
```sql
-- 创建表
create table test(id int, name string);

-- 创建外部表
create external table test(id int, name string) location `/user/yundao/test_table`
-- 指定行分隔
 row format delimited
-- 指定列分隔
 fields terminated by '\t'
-- 指定分区
 partitioned by (pt string)
-- 指定存储格式，可选格式有
-- textfile 文本格式, 默认值
-- sequencefile Hadoop API 提供的一种二进制文件，它将数据以<key,value>的形式序列化到文件中，相比textfile空间占用小
-- rcfile 一种列存的数据格式，在汇总计算时有性能加成，相比textfile空间占用小
-- orcfile rcfile的升级版本
 stored as textfile;
-- 指定内容格式
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
```

__完整版例子__
```sql
create external table logs_user_event(
  `type` int, 
  `time` timestamp, 
  code int, 
  address string, 
  phonemode string, 
  os string, 
  serialnumber string, 
  uuid string, 
  mac string, 
  version string, 
  sourcecode string, 
  ip string, 
  partyid bigint
) 
 row format serde "org.openx.data.jsonserde.JsonSerDe" 
 with serdeproperties (
  "ignore.malformed.json" = "true", 
  "dots.in.keys" = "true"
)
location "/logs/user_event"
```

## 修改表的SerDe属性
```sql
ALTER TABLE lujing_adapp_burry_point_delay SET SERDEPROPERTIES ( "ignore.malformed.json" = "true");
```

## 显示表结构

```sql
-- 显示数据库已有表
show share.tables;
	
-- 显示表描述
desc tb_test;
desc extended tb_test;
desc formatted tb_test;
		
-- 修改字段
alter table tb_test change id id2 int comment "注释";

-- 修改表注释
alter table tb_test set TBLPROPERTIES('comment'='每天各线路找货会员表');
```

## 加载数据
```sql
-- 加载本地文件数据, 覆盖模式，声明格式与实际格式不同的数据加载后无法正确读取，例如数据格式为text格式，
-- 表格式为sequencefile,rcfile,avrofile等，目前sqoop导入到hive只支持textfile格式，
-- 导入到hdfs支持textfile,sequencefile,avrofile三种格式*
load data local inpath '/home/yundao/test_data.txt' overwrite into table test;

-- 加载本地文件数据, 追加模式
load data local inpath '/home/yundao/test_data.txt'  into table test;

-- 加载HDFS数据, 覆盖模式
load data inpath '/user/yundao/test_data' overwrite into table test;

-- 加载HDFS数据, 追加模式
load data inpath '/user/yundao/test_data' into table test;

-- 从查询插入数据, 覆盖模式
insert overwrite table test_target select * from test_source;

-- 从查询插入数据, 追加模式
insert into table test_target select * from test_source;

-- 加载数据到分区，期中test_source表必须有与test_target分区同名列
insert into table test_target PARTITION (pt='[分区名]') select * from test_source;

-- 当test_source表没有名为pt的列时
insert into table test_target PARTITION (pt='[分区名]') select *, '2016-07-28' as pt from test_source;

-- 将查询结果插入多个表或HDFS目录
from test_insert1
insert overwrite local directory '/home/yundao/hive' select * 
  insert overwrite directory '/user/yundao/export_test' select value;
```

## 导出数据
```sql
-- 导出到本地目录
insert overwrite local directory 'party_route'                                
  row format delimited
  fields terminated by '\t'
  select * from party_route order by to_party_id, take_month desc, total_value desc;
```   
__导出到hdfs目录只需要去掉第一行中的'local'关键字__


## 表分区(PARTITION)
```sql
-- 创建分区表
create table test1(dummy int) partitioned by(pt string);
	
-- 创建多分区表
create table test1(dummy int) partitioned by(ptm string, ptd string);
	
-- 创建静态分区
alter table test1 add partition (pt="201507");
alter table test1 add partition (ptm="201507", ptd="01");
	
-- 创建动态分区
--	从已有的数据加载并自动创建分区
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
	
insert overwrite table test1 partition(pt) select *,substr(create_date,1,6) as pt from source_table;
	
-- 列出表分区
show partitions test1;
	
-- 删除表分区
alter table test1 drop partition(pt="201507");


-- 函数&UDF/UDAF/UDTF
-- 临时函数
add jar hive-udf.jar;
create temporary function tf as 'com.tf56.yundao.udf.PartySpeedUDF';

-- 删除临时函数
drop temporary function tf;
delete jar hive-udf.jar;
	
-- 列出已添加的jar
list jar;
	
-- 永久函数(适用数据库范围内使用)
use share;
create function share.row_num as 'com.tf56.yundao.udf.RowNumberUDF' using jar 'hdfs://ns1/user/yundao/udf.jar';
	
-- 显示函数列表
show functions;
	
-- 显示函数描述
desc function to_date;
desc function extended to_date;
```

## 查询
### 排序

三个关键字说明如下

* distribute by：控制着在map阶段如何分区，按照什么字段进行分区，重点考虑均衡负载避免倾斜
* sort by：每个reduce按照sort by 字段进行排序，reduce的数量按照默认的数量来进行，当然可以指定。最终可以进行归并排序得出结果。适用于数据量比较大的排序场景。
* order by：reduce只有一个，在一个reduce中完成排序，使用于数据量小的场景或者对最后结果排序。
	
### mapjoin

应用场景：1.关联操作中有一张表非常小 2.不等值的链接操作
```sql
select /*+ mapjoin(A)*/ f.a,f.b from A t join B f  on ( f.a=t.a and f.ftime=20110802)
```

## 特殊数据结构

Array、Map、Struct

### 复合数据结构的常用函数
```sql
-- array_contains - 数组中是否包含指定值
select * from rc_coupon where array_contains([1,2,3,4], type);

-- explode - 展开数组为行
select explode(split('hello world', ' '));
	
-- posexplode - 展开数组为行, 并添加序号列
select posexplode(split('hello world', ' '));
```

## view

...

## 常用内置函数

### 字符串操作
```sql
-- lpad - 左填充补齐长度
select lpad('hi', 3, '?') from dual;
	
-- rpad - 右填充补齐长度
select rpad('hi', 3 , '.') from dual;
	
-- sentences - 分词,将字符串分词为数组列表
select sentences("hello world") from dual;
select sentences("hello,world") from dual;
	
-- str_to_map - 字符串转map，delimiter1 - map元素分隔符，delimiter2 - map键值分隔符
str_to_map(text, delimiter1, delimiter2) - Creates a map by parsing text
select str_to_map('key1:val1,key2:val2', ',', ':') from dual;
	
-- translate - 转换字符
translate('abcdef', 'adc', '19') 
  returns '1b9ef' replacing 'a' with '1', 'd' with '9' and removing 'c' from the input string
select translate('abcdef', 'adc', '19') from dual;
```

`ucase, upper` - 小字转大写

`lcase` - 大写转小写

`bin` - 2进制表示

`hex` - 16进制表示

`xpath, xpath_boolean, xpath_double, xpath_float, xpath_int` - 用于xml, html搜索计算的利器



### 统计函数

```sql
-- percentile - 百分值计算
select percentile(price, array(0.3,0.5,0.7)) from trade;

-- percentile_approx - 百分值计算(近似)
select percentile_approx(price, array(0.3,0.5,0.7), 100000000) from trade;

-- histogram_numeric - 直方图计算
select  histogram_numeric(price, 10) from trade;
	
-- lag - 后一条记录值
select p1.p_mfgr, p1.p_name, p1.p_size,                                                         	
  p1.p_size - lag(p1.p_size,1,p1.p_size) over( distribute by p1.p_mfgr sort by p1.p_name) as delt 
	from part p1 join part p2 on p1.p_partkey = p2.p_partkey
	
-- lead - 前一条记录值
select lead(id), id from dual;
	
-- ngrams - 参见n-gram算法
ngrams(array<>, int N, int K, int pf)

-- 计算twitter中某两个词同时出现的频率的 top 100 --
SELECT ngrams(sentences(lower(tweet)), 2, 100 [, 1000]) FROM twitter;
	
-- stack* - ??
select stack(1, id) from dual;
	
-- std - 计算标准差
select std(column1) from dual;
select stddev(column1) from dual;
select stddev_pop(column1) from dual;
	
-- var_pop, variance - 计算方差
```
	
### 用于JSON的函数

```sql
-- get_json_object - 字符串转json对象
select get_json_object(line, '$.type') from ext_passport limit 10;
	
-- json_tuple - json字符串转tuple
select min(t.date), max(t.date) from ext_passport lateral view json_tuple(line, 'type', 'date') t as type, date;
```

### 窗口函数
```sql
-- sum, avg, min, max，关键是理解ROWS BETWEEN含义,也叫做WINDOW子句

-- sum - 合计
SELECT cookieid,
  createtime,
  pv,
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime) AS pv1, -- 默认为从起点到当前行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS pv2, --从起点到当前行，结果同pv1
  SUM(pv) OVER(PARTITION BY cookieid) AS pv3,        --分组内所有行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv4, --当前行+往前3行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND 1 FOLLOWING) AS pv5, --当前行+往前3行+往后1行
  SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS pv6 ---当前行+往后所有行
  FROM lxw1234;
```

```bash
cookieid createtime pv pv1 pv2 pv3 pv4 pv5 pv6
-----------------------------------------------------------------------------
cookie1 2015-04-10 1 1 1 26 1 6 26
cookie1 2015-04-11 5 6 6 26 6 13 25
cookie1 2015-04-12 7 13 13 26 13 16 20
cookie1 2015-04-13 3 16 16 26 16 18 13
cookie1 2015-04-14 2 18 18 26 17 21 10
cookie1 2015-04-15 4 22 22 26 16 20 8
cookie1 2015-04-16 4 26 26 26 13 13 4
-----------------------------------------------------------------------------
```

各列计算逻辑说明：

* pv1: 分组内从起点到当前行的pv累积，如，11号的pv1=10号的pv+11号的pv, 12号=10号+11号+12号
* pv2: 同pv1
* pv3: 分组内(cookie1)所有的pv累加
* pv4: 分组内当前行+往前3行，如，11号=10号+11号， 12号=10号+11号+12号， 13号=10号+11号+12号+13号， 14号=11号+12号+13号+14号
* pv5: 分组内当前行+往前3行+往后1行，如，14号=11号+12号+13号+14号+15号=5+7+3+2+4=21
* pv6: 分组内当前行+往后所有行，如，13号=13号+14号+15号+16号=3+2+4+4=13，14号=14号+15号+16号=2+4+4=10

如果不指定ROWS BETWEEN,默认为从起点到当前行;

如果不指定ORDER BY，则将分组内所有值累加;

关键是理解ROWS BETWEEN含义,也叫做WINDOW子句：

PRECEDING：往前

FOLLOWING：往后

CURRENT ROW：当前行

UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点

其他AVG，MIN，MAX，和SUM用法一样。

```sql
-- avg - 均值，商品日均价变化曲线
select goods_id, avg(price) 
  over(partition by goods_id,to_date(create_time) 
  order by create_time) as price_avg
from goods

-- min - 最小值，商品日最低价变化曲线
select goods_id, min(price) 
  over(partition by goods_id,to_date(create_time) 
  order by create_time) as price_min
from goods

-- max - 最大值
	...
```

### JDBC操作函数
```sql
CREATE TEMPORARY FUNCTION dboutput 
  AS 'org.apache.hadoop.hive.contrib.genericudf.example.GenericUDFDBOutput';

select dboutput(
  'jdbc:mysql://10.33.64.15:3306/report',
  'admin',
  '******',
  'INSERT INTO QLujingAppDriverAndShipperDau(
    inputDate, 
    driverAppDau, 
    adAppDau
  ) VALUES (?,?,?)',
  input_date, driverapp_dau, adapp_dau) 
from increment.report_lujing_app_dau;
```

> GenericUDFDBOutput是hive自带函数,能在shell里直接使用，但是hue里不会加载这个jar,需要添加到AUX_JARS_PATH=[jar]或者在HIVE_HOME目录下新增一个auxlib目录，将该jar放到这个目录下,重启hive服务后，才能正常使用; 网上的在hue里配置修改辅助jar路径，可能导致让shell中Hive相关命令不能正常工作。
{: .prompt-info }