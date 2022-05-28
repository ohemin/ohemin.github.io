---
title: YARN Web UI 不能查看
date: 2017-08-30 19:40:00 +0800
categories: [Big data, Yarn]
---

访问YARN Web UI地址时，页面出现如下提示：

`You (User dr.who) are not authorized to view application application_xxxx`

经多次翻日志，翻文档后，得知需要关注如下配置项，与实际环境是否相符。Hadoop运维过程中不经意间的配置修改就可能引发种种问题。

```xml
<property>  
  <name>hadoop.http.staticuser.user</name>  
  <value>hadoop</value>  
</property> 
```

