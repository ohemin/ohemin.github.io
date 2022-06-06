---
title: CentOS下的docker安装与配置
---
版本 CentOS 8.2

## docker-ce

经测试yum里的docker包不能直接安装，直接安装的是podman-docker

执行`docker`命令会抛出如下错误

```
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
/usr/bin/podman: symbol lookup error: /usr/bin/podman: undefined symbol: seccomp_notify_fd
```

删除原有安装，重新安装阿里源的docker-ce

```bash
sudo yum remove docker
# 配置阿里docker-ce源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 更新yum包
sudo yum makecache fast
# 安装docker-ce
sudo yum -y install docker-ce

# 启动 docker 服务，否则docker相关命令无法执行
sudo systemctl start docker

## 设置 docker 服务自启动
sudo systemctl enable docker
```

此时运行docker命令会提示权限不够，使用sudo倒是可以了，但是可以通过简单的配置让普通账户也能执行docker命令，这里以admin账户为例，要求设置的账户有sudo权限

```bash
# 查看docker用户
sudo cat /etc/group | grep docker
# 如果显示如下信息表示存在docker用户，一般正常安装docker-ce都会创建这个用户
# docker:x:987
ll /var/run/docker.sock
# 显示如下
# srw-rw----, 1 root docker 0 8月    5 20:31 /var/run/docker.sock

# 将当前用户admin添加至docker组
sudo gpasswd -a admin docker
# 刷新组信息
newgrp docker
# 查看当前账户所属组信息
id
# uid=1000(admin) gid=987(docker) groups=987(docker),10(wheel),1000(admin) 
```

再用该用户输入docker相关命令就不再有权限提示了