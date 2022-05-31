---
categories: [Database, MySQL]
---
MySQL 数据库安装好以后，使用前需要对数据库初始化，比如创建库表、创建用户、授权等，下面写一个典型建库、建表、建用户的流程

## 创建新的数据库并授权

```sql
-- 创建数据库并指定字符集utf8mb4
create database if not exists ohemin default charset utf8mb4 collate utf8mb4_general_ci;

-- 创建数据库用户并指定密码
create user 'devuser'@'localhost' identified by password('123456');

-- 查询用户
select user,host from mysql.user;

-- 删除用户
drop user devuser@localhost;

drop user devuser@'%';

-- 更改密码方法1, 密码实时更新;
set password for test = password('123456');

-- 更改密码方法2, 需要刷新;
update mysql.user set password=password('123456') where user='test';

flush privileges;
```
> 建库字符集可以选择utf8或者utf8mb4，它们的区别在于utf8mb4可以在记录值中正确支持操作系统或者社交软件的表情符号，而utf8则会出现错误。好消息是高版本的MySQL已经将这两个字符集合并，无论选择utf8或者utf8mb4都可以正常使用表情符号.
{: .prompt-tip}

> collate utf8mb4_general_ci 是数据库校对规则，ci是case insensitive缩写，表示大小不敏感，在使用上建表语句或者查询中的大写表名或字段名均转为小写；如果想使用基于Oracle数据库区分大小写的规则可用 collate utf8mb4_general_cs，cs即case sensitive缩写，表示大小写敏感。
{: .prompt-tip}

## 用户分配权限
```sql
-- 授予用户devuser通过外网IP对数据库'os'的全部权限
grant all privileges on os.* to 'devuser'@'%' identified by 'devuser123';

-- 授予用户devuser通过任意P对于数据库os中表的创建\修改\删除权限 ,以及表数据的增删改查权限 
grant create,alter,drop,select,insert,update,delete on happyos.* to happyos@'%';

-- 刷新权限
flush privileges;
```