---
title: 修改bash窗口前导提示 
date: 2022-05-29 11:01:00 +0800
categories: [Linux]
---

上次修改桌面布局后，terminal 窗口宽度变得较为固定，此时遇到一个问题是当我进入一个名称较长的目录后窗口的前导提示已经占据了一行中大半的空间，体验非常不好，于是决定动手修改一下。


# 先查看一下当前所使用的前导格式
```bash
echo $PS1
> \h:\W \u\$
```

这里贴一下格式说明：

|变量 | 描述 |
|--- |---   |
|\d |当前系统日期 |
|\t |系统时间 |
|\h |主机名      |
|\# |命令符号      |
|\u |用户名      |
|\W |当前路径名      |
|\w |当前完整路径      |

修改为只显示当前路径名的格式，因为要修改系统级配置，需要加上sudo使用root权限

```bash
sudo sed -i '' 's/\\h:\\W \\u/\\W/g' /etc/bashrc

# 检查修改是否正确
cat /etc/bash | grep PS1=
> PS1='\W\$'

# 使修改生效
source /etc/bashrc
```