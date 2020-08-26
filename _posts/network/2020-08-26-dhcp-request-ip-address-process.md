---
layout: post
title: DHCP请求IP地址流程
categories: [Network]
description: some word here
keywords: Network
---

# DHCP请求IP地址流程

DHCP 分为两部分：客户端和服务器

所有客户机的ip地址都是由DHCP服务器集中管理的，客户端向服务端发送请求ip的请求，服务端负责分配IP地址

DHCP基于UDP协议，端口为**67**和**68**，68端口一般为客户端采用，67端口一般服务端采用

### DHCP分配ip的方式

+ 自动分配，当 DHCP 客户端第一次成功的从服务端分配到一个IP地址之后，就永远使用这个IP地址
+ 动态分配，DHCP 客户端从服务端分配到IP之后非永久使用，每次使用完会释放这个IP地址

+ 手动分配，是由DHCP服务器管理员手动为客户机分配IP地址

### DHCP 服务工作流程

DHCP客户机在启动时，会搜寻网络中是否存在DHCP服务器，如果有，则给DHCP服务器发送一个请求，DHCP服务器收到请求后，会为客户机选择TCP/IP配置参数，并把这些参数发送给客户机。如果配置了冲突检测，DHCP服务器会在分配给客户机ip之前使用ping命令测试作用域中每个可用ip的连通性。这样可以确保提供给客户机的ip地址都没有被使用。

新客户机获取ip流程：

#### 1. 客户机寻找DHCP服务器

DHCP 客户机第一次登录到网络的时候，计算机发现本机上面没有任何ip地址的设定，会以广播的方式发送DHCP discover 发现信息寻找DHCP服务器，即向255.255.255.255发送特定的广播包，同一个广播域里面的所有主机都会收到，但是只有DHCP服务器会做出响应

源ip：0.0.0.0

源端口：68

源mac：客户机mac地址

目的ip：255.255.255.255

目的端口：67



#### 2. 服务器分配IP地址

接收到DHCP discover信息的DHCP服务器，会从未分配的ip地址池里面挑选一个ip分配给客户机，即向客户机发送一个DHCP offer 信息，offer信息包括：

DHCP客户机MAC信息，DHCP服务器提供的合法IP，子网掩码，默认网关(路由)，租约期限，DHCP服务器IP

源IP：服务器ip

源端口：67

目的ip：255.255.255.255

目的端口：68



#### 3. 客户机选择IP

DHCP 客户机从接收到的第一个 DHCP offer 消息中选择IP地址，发出IP地址的服务器将保留该地址(打上已经分配的标签)，防止提供给其他的客户机。当客户机选择到IP地址后，DHCP 客户机会广播 DHCP request 消息到所有的 DHCP 服务器，表明他接受提供的内容， DHCP request 消息包括为该客户机提供 DHCP 服务的服务器的标识符（IP地址）DHCP 服务器会查看服务标识字段来确认自己是否被选择为客户机指定的 DHCP 服务器，如果  DHCP offer 被拒绝，则 DHCP 服务器会取消提供的 IP 以便于给下一个租户使用

源IP：0.0.0.0

源端口：68

源mac：客户机mac

目的IP：255.255.255.255

目的端口：67



#### 4. 服务器确认租约

DHCP 服务器接收到 DHCP request 消息后，会以广播的方式回复 DHCPACK 做成功确认，该消息包含 IP 地址的有效租约和配置信息。 此时服务器确认了租约请求，但是客户机还没有接收到确认消息

源IP：服务器IP

源端口：67

目的IP：255.255.255.255

目的端口：68

当客户机收到 DHCP request 消息时，他就完成了IP地址的配置，完成了TCP/IP初始化

如果 DHCP request 不成功，DHCP 服务器将会广播一个 DHCPNACK 消息，客户机接收到 DHCPNCAK 消息时，会重新申请 IP

> 如果DHCP客户机无法找到 DHCP 服务器，会从 TCP/IP 的 B 类网段里面挑选一个IP作为自己的临时IP，并不断尝试与DHCP服务器联系，直到获得IP
>
> 如果客户机有多张网卡，DHCP 服务器会为每张网卡分配一个唯一且有效的IP



#### 5. 重新登录

当 DHCP 客户机重新登录到网络时，无需再次发送 DHCP discover 消息了，而是直接发送包含前一次所分配 IP 地址的 DHCP request 请求消息，DHCP 服务器收到这一消息后，会尝试让客户机继续使用原来的IP，并回答一个 DHCPACK 确认消息，如果此 IP 已无法再使用，则会回复一个DHCPNACK 消息，此时客户机需要重新发送 DHCP discover 消息来重新申请IP



#### 6. 更新租约

DHCP 服务器向 DHCP 客户机出租的IP地址一般都有一个租借期限，期满后 DHCP 服务器便会收回出租的IP地址，如果 DHCP 客户机要延长其 IP 租约，则必须更新其 IP 租约，DHCP 客户机启动时和 IP 租约期限到达租约的50%时，DHCP 客户机都会自动向 DHCP 服务器发送更新其IP租约的信息



### dhclient  使用

```bash
dhclient -r     # 释放IP
dhclient        # 获取IP
dhclient eth0   # 续租IP地址
```













[Reference1 >> ]( https://www.zyops.com/dhcp-working-procedure/ )

[Reference2 >> ]( https://blog.51cto.com/yuanbin/109574 )

