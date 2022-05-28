---
title: 使用mac自带apache服务 
date: 2015-05-25 09:22:00 +0800
categories: [Mac, software]
tags: [develop, environment]
---

mac 系统通常默认自带有Apache http服务，我们要做的就是把它启动

## 管理apache服务

### 启动服务
```bash
sudo apachectl start
```

### 停止服务
```bash
sudo apachectl stop
```

### 重启服务
```bash
sudo apachectl restart
```

## 管理apache自启动

### 希望每次系统开始后，apache服务就启动好了
```bash
launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

### 验证下是不是设置上了
```bash
launchctl list | grep httpd
```

### 又后悔了，不想让服务自动启动
```bash
launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

## 编辑apache配置
```bash
vi /etc/apache2/httpd.conf
```

### 建将DocumentRoot目录改掉
```bash
# 默认网页文件在 /Library/WebServer/Documents 管理起来不方便，改为更方便管理的目录
# Mac版命令，第1个参数为空字符串，意思是原地替换，如果不为空则替换前产生一处备份文件，将参数中字符添加到备份文件扩展名后
sudo sed -i '' 's/\/Library\/WebServer\/Documents/\/Users\/[your name]\/www/g' '/etc/apache2/httpd.conf'

# Linux版命令
sudo sed -i 's/\/Library\/WebServer\/Documents/\/Users\/[your name]\/www/g' '/etc/apache2/httpd.conf'
```
