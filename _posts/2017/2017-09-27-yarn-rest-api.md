---
title: YARN 生产集群办公环境无法访问
date: 2017-08-30 19:40:00 +0800
categories: [Big data, Yarn]
tags: [security]
---

公司出于安全考虑，陆续在办公网络禁用对生产系统的直接访问，对生产系统所有操作必须先登录跳板机，再通过跳板机对生产系统访问。原来直接访问Yarn Web UI来跟踪任务执行情况的方式已经无用了。

幸好Yarn提供了基于HTTP Api的方式访问，下面开始搞起...

1. 根据官网文档要求编写json格式内容存储到json文件，这里名为 `yarn-rest-request.json`{: .filepath}
2. 将该文件保存到跳板机
```bash
scp yarn-rest-request.json admin@[server_host]:~/
```
3. 在跳板机上执行如下命令，这里的域名及端口修改为实际域名和端口
```bash
curl -L -H'Content-Type: application/json' \
  -XPOST \
  --data-binary @yarn-rest-request.json \
  http://big-elephant-3:8088/ws/v1/cluster/apps
```
4. 然后就可以看到服务器返回的信息了，如有必要可以将返回结果保存到文件便于后续使用。
5. 如果跳板机无法保存文件执行命令等操作，使用iTerm2软件配置好隧道穿透即可上传yarn-rest-request.json文件到生产服务器，后面按照第3步开始执行即可