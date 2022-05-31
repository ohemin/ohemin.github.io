---
categories: [Database, MySQL]
---

个人开发机上的MySQL，因为长期不用root用户密码已经忘记了，现在需要创建一个新用户需要找回root用户密码进行操作

## macOS 系统下找回管理密码

以mac操作系统为例，演示操作步骤

1. 在'系统偏好设置'中关闭mysql服务
2. 打开终端依次输入如下命令

```bash
cd /usr/local/mysql/bin

# 需要root权限执行相关操作
sudo su
./mysqld_safe --skip-grant-tables &

./mysql
```

进入MySQL命令行后输入如下MySQL命令
```sql
flush privileges;
set password for 'root'@'localhost' = password('123456');
exit;
```

完成密码设置后测试一下, 用新密码登录MySQL，成功登入，接下来就可以正常模式启动mysql服务了
```bash
./mysql -u root -p123456
```

> 这里密码设置为123456是方便开发机测试，在正式项目中不要使用这类简单密码
{: .prompt-warning}