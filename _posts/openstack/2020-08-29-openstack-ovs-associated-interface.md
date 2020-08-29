---
layout: post
title: ovs网桥关联网卡
categories: [OpenStack]
description: some word here
keywords: OpenStack
---

# ovs网桥关联网卡

### 不组bond

为 ovs 网桥添加网卡

```bash
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex ens160 && ovs-vsctl set interface ens160 type=patch
```

创建 br-ex 配置文件

```bash
[root@controller ~]# cd /etc/sysconfig/network-scripts
[root@controller network-scripts]# touch ifcfg-br-ex
```

修改网卡配置文件

```bash
# 备份
[root@controller network-scripts]# cp ifcfg-ens160 ifcfg-ens160.back
# 修改文件
[root@controller network-scripts]# vim ifcfg-ens160

NAME=ens160
DEVICE=ens160
ONBOOT=yes
NETBOOT=yes
BOOTPROTO=static
TYPE=OVSIntPort
OVS_BRIDGE=br-ex
DEVICETYPE=ovs

```

修改 br-ex 配置文件

```bash
[root@controller network-scripts]# vim ifcfg-br-ex

NAME=br-ex
DEVICE=br-ex
ONBOOT=yes
TYPE=OVSBridge
DEVICETYPE=ovs
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.8.212
NETMASK=255.255.255.0
GATEWAY=192.168.8.254

```



 