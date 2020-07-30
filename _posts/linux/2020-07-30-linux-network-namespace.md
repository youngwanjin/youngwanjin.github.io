---
layout: post
title: Linux Network Namespace
categories: [Linux]
description: Linux network namespace
keywords: Linux, Network
---

## 基础概念

**network namespace** 能够创建多个隔离的网络空间，彼此透明，拥有自己独立的网络栈信息

每一个 **network namespace** 会有自己独立的网卡、路由表、ARP 表、iptables 等网络相关资源

> **ip netns help** 使用该命令可以查看相关命令帮助
>
> 创建的 network namespace 在 `/var/run/netns` 下面



```shell
[root@controller-2 ~]# ip netns help
Usage: ip netns list  
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id 
```

1. `ip netns list` 查看所有 network namespace
2. `ip netns add NAME` 添加 network namespace 
3. `ip netns exec [name]` 在 namespace 中执行命令 ，使用 `bash` 可以进入 namespace 相当于进入容器 



> 每个 namespace 创建完成后都会自动创建一个 `lo` interface ,用于实现 loopback 通信
>
> ```shell
> # 启用 lo
> ip netns exec [name] ip link set lo up
> ```
>
> 默认情况下，namespace 是不能和主机网络或者其他 network namespace 通信



## network namespace 之间通信

虚拟网卡 **veth** ，**veth pair** 是成对出现的，可以理解为是一个双向的 pipe (管道)，数据从 veth 一端进去会从另一端出来，可以使用这种特殊的虚拟网卡实现了两个 namespace 之间的直接通信

创建 **veth：**

```shell
1. ip link add veth0 type veth peer name veth1 # 指定名称创建 veth pair
2. ip link add type veth  # 使用系统默认名称创建veth pair
3. ip link  # 查看 veth pair 列表
4. ip link set veth0 netns netns0  # 将 veth pair veth0 添加到 namespace netns0 中
5. ip netns exec netns0 ip addr add 10.0.0.1/24 dev veth0  # 为 eth0 配置 ip 地址
6. ip netns exec netns0 ip route	# 查看 netns0 的路由表
7. ip netns exec netns0 ip link set veth0 up	# 启用 veth0
8. ip netns exec netns0 ping ...  	# 测试连通性
```

> 1. **veth pair** 无法单独存在，删除其中一个，另一个也会自动删除 
>
> 2. 启用一端 veth pair 会提示未准备好，只有两端都启用才能使用

![](D:\Mygithub\学习记录\images\linux_namespace_veth.png)



## namespace 连接 bridge

使用 linux bridge 技术实现多个 namespace 之间的通信，可理解为 bridge 实现了交换机的功能

> linux brideg 相关操作可以使用 brctl 实现

创建 bridge ：

```shell
1. ip link add br0 type bridge 		# 创建一个 bridge 
2. ip link set dev br0 up 		# 启用一个 bridge
3. ip link set dev veth0 master br0 	# 将 veth0 连接到 br0
4. ip link set dev veth0 up 	# 启用 veth0
5. ip addr add 10.1.1.2/24 dev veth0 	# 为 veth0 添加 ip
5. bridge link 	# 查看 bridge 上面的 link 信息
6. ip netns exec net0 ping ... 	# 测试连通性
```

> 创建一对 veth pair 一端添加到 namespace ，一端添加到 bridge ，并配置 ip

![](D:\Mygithub\学习记录\images\linux_networkspace_bridge.png)





[Reference1 >> ]( https://cizixs.com/2017/02/10/network-virtualization-network-namespace/ )

[Reference2 >> ]( https://www.jianshu.com/p/369e50201bce )

[Reference3 >>]( https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/ )