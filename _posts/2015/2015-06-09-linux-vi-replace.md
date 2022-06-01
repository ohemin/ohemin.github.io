---
title: vi vim中的文本替换
date: 2015-06-09 13:22:00 +0800
categories: [Linux, vi/vim]
---

vi/vim 是Linux下文件编辑软件，其中文本替换功能经常用到，如果熟悉常用替换命令，工作效率大幅提升

```bash
# 替换当前行第一个 vivian 为 sky 
：s/vivian/sky/ 
	
# 替换当前行所有 vivian 为 sky 
：s/vivian/sky/g 
	
# 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky 
：n，$s/vivian/sky/ 
	
# 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky
# n 为数字，若 n 为 .，表示从当前行开始到最后一行 
：n，$s/vivian/sky/g 
	
# 替换每一行的第一个 vivian 为 sky
：%s/vivian/sky/（等同于 ：g/vivian/s//sky/） 
	
#（等同于 ：g/vivian/s//sky/g） 替换每一行中所有 vivian 为 sky
#	可以使用 `#` 作为分隔符，此时中间出现的 / 不会作为分隔符
：%s/vivian/sky/g

# 替换当前行第一个 vivian/ 为 sky/
：s#vivian/#sky/# 
	
#（使用+ 来 替换 / ）： /oradata/apras/替换成/user01/
：%s+/oradata/apras/+/user01/apras1+apras1/
```
> 本篇中的替换语法同样适用于sed命令工具
{: .prompt-tip}