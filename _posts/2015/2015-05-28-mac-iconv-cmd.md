---
title: 从Windows复制到mac的文本文件无法打开
date: 2015-05-28 11:02:00 +0800
categories: [Mac, tip]
tags: [command]
---

从原来的Windows系统迁移过来一些文本文件，发现在mac上打不开了，但在Windows上能正常打开，后来查到原因是文件的编码问题，在简体中文环境下Windows记事本创建的文本文件默认为GBK编码，mac上默认打开编码则为UTF-8，由于编码不正确mac会直接提示无法打开。

解决办法有两种：

1. 在mac上不要直接双击文件打开，而是先在`dock`中或者`finder`的`应用程序`中打开`文本编辑器`，在应用弹出的文件窗口中选中要打开的文件，并且点击下方的`选项`按钮并在`纯文本编码:`一栏选择`中文(GB 18030)`
2. 使用mac系统自带命令`iconv`将文件转码为`UTF-8`编码，以后就可以直接打开该文件了

iconv 转换命令
```bash
iconv -f GBK -t UTF-8 gbk.txt > utf8.txt
```

-f参数后是原文件编码，-t参数后是转换后的编码，执行后得到的就是可以直接正常打开的文本文件了

想知道iconv命令支持哪些编码格式，可以执行下面命令
```bash
iconv -l
```

> 本文针对Windows 7及更早版本的Windows系统，Windows 10及其之后版本修改了默认存储编码为UTF-8，结束了一直以来的编码问题
{: .prompt-warning}
