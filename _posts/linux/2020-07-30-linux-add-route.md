---
layout: post
title: Linux  添加路由
categories: [Linux]
description: Linux 添加路由规则
keywords: Linux, Network
---

## 静态路由

### route 命令

1. 添加路由

```shell
route add -net 192.168.0.0/24 gw 192.168.0.1

route add -host 192.168.1.1 dev 192.168.0.1
```

2. 删除路由

```shell
route del -net 192.168.0.0/24 gw 192.168.0.1
```

> 1. **add** ：添加路由
> 2. **del** ：删除路由
> 3. **-net** ：设置到某个网段的路由
> 4. **gw** ：出口网关 IP 地址
> 5. **-host** ：设置到某台主机的路由
> 6. **dev** ：出口网关，物理设备名称
> 7. **target** ：目的网络或主机
> 8. **netmask**： 目的地址的网络掩码 

3. 添加默认路由

```shell
route add default gw 192.168.0.1
```

> 1. **route -n ** 查看路由表
> 2. **route --help** 查看命令帮助



### ip route 命令

1. 添加路由

```shell
ip route add 192.168.0.0/24 via 192.168.0.1

ip route add 192.168.0.0/24 dev 192.168.0.1
```

2. 删除路由

```shell
ip route del 192.168.0.0/24 via 192.168.0.1
```

> 1. **add** 添加路由
> 2. **del** 删除路由
> 3. **via** 网关出口，ip 地址
> 4. **dev** 网关出口，物理设备名

3. 添加默认路由

```shell
ip route add default via 192.168.0.1 dev eth0
```

> **ip route** 查看路由表信息
>
> **ip route help** 查看帮助信息



## 永久路由

1.  在`/etc/rc.local`里添加

```shell
route add -net 192.168.0.0/24 dev eth0

route add -net 192.168.1.0/24 gw 192.168.1.254
```

2. 在 ` /etc/sysconfig/static-routes ` 离添加

```shell
# 没有static-routes,可以手动建立一个这样的文件
any net 192.168.0.0/24 gw 192.168.0.1

any net 10.250.228.128 netmask 255.255.255.192 gw 10.250.228.129
```

3. 开启 ip 转发

```shell
# 临时
echo "1" >/proc/sys/net/ipv4/ip_forward 
# 永久开启
vi /etc/sysctl.conf --> net.ipv4.ip_forward=1
```

