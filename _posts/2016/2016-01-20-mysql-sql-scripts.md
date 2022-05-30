---
title: 抛弃IDE从命令行执行MySQL脚本
date: 2016-01-20 12:16:00 +0800
categories: [Database, MySQL]
tags: [develop]
---


在命令行让MySQL执行已经编写好的SQL脚本，比如批量数据插入，或者更新等

下面是基本信息

* MySQL IP: 192.168.0.100
* MySQL Port: 3306
* 数据库名: test
* 用户名：dev
* 脚本名：create-tables.sql


```bash
mysql -h192.168.0.100 -udev -p -Dtest<create-tables.sql
```
> 上面命令要求执行机上已经安装好MySQL客户端
{: .prompt-info}

如果不希望安装MySQL客户端则需要登录mysql服务器

```bash
scp create-tables.sql admin@192.168.0.100:~/
ssh admin@192.168.0.100
mysql -udev -p -Dtest<create-tables.sql
```
> 需要有MySQL服务器登录权限并且知道用户名和密码，一般正规公司环境有点难拿到权限
{: .prompt-info}