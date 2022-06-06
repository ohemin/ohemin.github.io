---
categories: [Linux, CentOS]
---
本人CentOS开发服务器换了块硬盘，因此需要重新安装一下，为了下次加快安装设置速度，把遇到的问题记录一下

## CentOS8

初始安装介质是CentOS 8.2，从光盘或者U盘启动，进入安装程序：

1. 语言：English
2. 时区：Asia/Shanghai
3. 根据需要选择安装软件
4. 安装磁盘：sda1，如果空间不够选择清空磁盘
5. 网络开关打开以太网
6. 设置root密码（不设置无法远程登录）
7. 设置一个新用户 "admin" 
8. 安装完成，重启系统

以root用户登录，完成一些初始配置

### network

第一步设置网络，否则没有连接
```bash
# 下面命令中的enp2s0视不同主机的网卡名称会有所不同，按照实际替换即可
vi /etc/sysconfig/network-scripts/ifcfg-enp2s0
```

配置文件的初始值`ONBOOT=no`，意味着不会随系统启动而启动，所以开机后网络处于不可用状态，必须改掉

使用DHCP自动分配IP的配置如下

```conf
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp2s0
UUID=1184bd2c-04f9-4169-8a4d-f9191cf60542
DEVICE=enp2s0
ONBOOT=yes
```

使用static静态IP配置如下

```conf
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp2s0
UUID=1184bd2c-04f9-4169-8a4d-f9191cf60542
DEVICE=enp2s0
IPADDR=192.168.0.100
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
ONBOOT=yes
```

修改完成后重启网卡
```bash
# 重新加载配置文件
nmcli c reload
# 重启网卡
nmcli c up enp2s0
```

### 更换yum源为国内阿里源

由于Centos8的官方源已更换，我安装好的系统根本访问不了默认yum源，直接全部删除重配置

```bash
sudo rm -f /etc/yum.repos.d/*.repo
sudo curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

打开下载的CentOS-Base.repo添加如下内容

```conf
[epel]
name=epel
baseurl=https://mirrors.aliyun.com/epel/8/Everything/x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/epel/RPM-GPG-KEY-EPEL-8
```

更新yum源
```bash
yum -y update
```
