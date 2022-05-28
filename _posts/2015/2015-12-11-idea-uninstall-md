---
title:  完全卸载Inteliij IDEA
date: 2015-06-09 13:22:00 +0800
categories: [Mac, software]
tags: [idea]
---

在mac系统中安装了idea之后，因为升级更换版本等原因需要卸载重装，但是如果没有卸载干净容易出现一些奇怪问题

mac系统标准方式卸载就是在dock中按着应用图标不动，3秒后图标左上会出现一个 "`X`" 号，点击它就可以删除软件，但有时候这个 "`X`" 不会出现，这时就要祭出 `terminal大法` 了

打开终端，输入如下命令
```bash
cd /Applications
sudo rm -rf Inteliij\ IDEA.app
```

当应用名称包含空格时，命令中需要在空格高加上 \ 进行转义处理

应用主体删除后，再删除应用运行时留下的配置、日志、插件等其它遗留目录，目录用途和路径说明如下：

* Config: ~/Library/Preferences/IdeaIC13
* System: ~/Library/Caches/IdeaIC13
* Plugins: ~/Library/Application Support/IdeaIC13
* Logs: ~/Library/Logs/IdeaIC13

一口气全部删除

```bash
rm -rf \
  ~/Library/Preferences/IdeaIC13\
  ~/Library/Caches/IdeaIC13\
  ~/Library/Application\ Support/IdeaIC13\
  ~/Library/Logs/IdeaIC13
```
