---
layout: post
title: Centos 7 配置 ip 地址
categories: [Linux]
description: Centos 7 配置 ip 地址
keywords: Linux
---

# Centos 7 配置 ip 地址

### 设置静态 ip

查看网卡信息

```shell
[root@controller ~]# ip addr
```

修改配置文件：

```shell
[root@controller ~]# cd /etc/sysconfig/network-scripts/
[root@controller network-scripts]# vim ifcfg-ens160

# 修改之后的结果
TYPE=Ethernet 
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static  # BOOTPROTO=dhcp 表示动态ip
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
UUID=2679f5aa-405a-4114-a57b-3cd657824cec
DEVICE=ens160  # 网卡名称
ONBOOT=yes  # 需要修改,开机启动,激活设备,默认是关闭的
IPADDR=192.168.8.211
NETMASK=255.255.255.0
GATEWAY=192.168.8.254
DNS1=114.114.114.114

```

重启网络服务：

```shell
[root@localhost ~]# systemctl restart network
```

验证配置:

```shell
[root@localhost ~]# ping 192.168.8.254
```



### 临时设置

查看网络接口：

```shell
[root@localhost ~]# ifconfig 	# 查看当前活动状态的网络接口
[root@localhost ~]# ifconfig eth0 	# 查看eth0网卡状态信息
[root@localhost ~]# ifconfig -a 	# 即 ifconfig -all 查看所有启动禁用的网络接口
[root@localhost ~]# ip a 	# 查看所有的网络接口信息
```

启用、禁用网卡：

```shell
# 启用网卡
[root@localhost ~]# ifconfig eth0 up
[root@localhost ~]# ifup eth0
# 禁用网卡 
[root@localhost ~]# ifconfig eth0 down
[root@localhost ~]# ifdown eth0
```

临时配置 ip：

```shell
[root@localhost ~]# ifconfig eth0 192.168.8.210  netmask  255.255.255.0 up # ip 网关
[root@localhost ~]# ifconfig eth0 192.168.8.210/24  # 设置 ip 
[root@localhost ~]# ifconfig eth0 192.168.8.210  # 设置 ip 
[root@localhost ~]# route add default gw 192.168.8.254  # 设置网关
[root@localhost ~]# 
```

设置网关：

```shell
[root@localhost ~]# echo "nameserver 8.8.8.8" >> /etc/resolv.conf
[root@localhost ~]# echo "nameserver 114.114.114.114" >> /etc/resolv.conf
```

重启网络服务：

```shell
[root@localhost ~]# systemctl restart network
```

