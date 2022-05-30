---
title: mac中安装MySQL数据库
date: 2015-05-26 19:22:00 +0800
categories: [Mac, MySQL]
tags: [develop, environment]
---

apache 已经配置好并启动了，下一步要安装Java开发必备数据库，MySQL

这次安装比较简单，先到官网下载安装包：

1. 浏览器打开 <https://dev.mysql.com/downloads/>
2. 在打开的页面中点击 `MySQL Community Server`，这个是社区版，开发者免费使用
3. 进入新页面后会有系统版本等选择，我们选择自己对应操作系统的版本，然后点击`Download`按钮开始下载
4. 下载完成后文件类似`mysql-5.7.23-xxx.dmg`，点击打开，在弹出的窗口中仁显示一个名称相似但文件后缀为pkg的文件，继续双击打开，会启动一个安装程序，一路next完成安装
5. 安装完成后，会在`系统偏好设置`中，新增加了一个名为`MySQL`的图标，点击进去后可以看到一个`Start MySQL Server`按钮，就是这里启动MySQL了，下方还有一个复选框，可以设置是否在系统启动时自动运行MySQL服务，可以视情况自己决定是否要勾选。

至此开发环境MySQL也已安装完成
