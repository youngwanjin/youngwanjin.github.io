---
layout: post
title: OpenStack Neutron 基础资源使用
categories: [OpenStack, Neutron]
description: some word here
keywords: OpenStack, Neutron
---

### 1. network

network 是一个隔离的二层广播域，支持多种network

+ **local** 

  local网络与其他网络和节点隔离， local 网络中的 instance 只能与位于同一节点上同一网络的 instance 通信，local 网络主要用于单机测试。 

+ **flat**  

  flat 网络是无 vlan tagging 的网络。flat 网络中的 instance 能与位于同一网络的 instance 通信，并且可以跨多个节点 

+ **vlan**

   vlan 网络是具有vlan tagging 的网络。vlan 是一个二层的广播域，同一 vlan 中的 instance 可以通信，不同 vlan 只能通过 router 通信。vlan 网络可以跨节点，是应用最广泛的网络类型

+ **vxlan**

  vxlan 是基于隧道技术的 overlay 网络。vxlan 网络通过唯一的 segmentation ID（也叫 VNI）与其他 vxlan 网络区分。vxlan 中数据包会通过 VNI 封装成 UDP 包进行传输。因为二层的包通过封装在三层传输，能够克服 vlan 和物理网络基础设施的限制

+ **gre**

   gre 是与 vxlan 类似的一种 overlay 网络。主要区别在于使用 IP 包而非 UDP 进行封装 



**openstack network 命令：**

> 1. 学会使用 `-h ` 和 `--help`        [>> Reference]( https://docs.openstack.org/python-openstackclient/train/cli/command-objects/network.html )
> 2. 创建 network 依赖资源，可使用 openstack network 相关命令查看

1. **openstack network list**  -- 查看network列表

```shell
openstack network list # 查看network列表
                    [--external | --internal] 
                    [--long]
                    [--name <name>] 
                    [--enable | --disable]
                    [--project <project>]
                    [--project-domain <project-domain>]
                    [--share | --no-share] 
                    [--status <status>]
                    [--provider-network-type <provider-network-type>]
                    [--provider-physical-network <provider-physical-network>]
                    [--provider-segment <provider-segment>]
                    [--agent <agent-id>] 
                    [--tags <tag>[,<tag>,...]]
                    [--any-tags <tag>[,<tag>,...]]
                    [--not-tags <tag>[,<tag>,...]]
                    [--not-any-tags <tag>[,<tag>,...]]
--long
 	List additional fields in output
 	
--project <project>
	List networks according to their project (name or ID)
	
--project-domain <project-domain>
	Domain the project belongs to (name or ID). This can be used in case collisions between project names exist
	
--provider-network-type <provider-network-type>
	List networks according to their physical mechanisms. The supported options are: flat, geneve, gre, local, vlan, vxlan
	
--provider-physical-network <provider-physical-network>
	List networks according to name of the physical network
	
--provider-segment <provider-segment>
	List networks according to VLAN ID for VLAN networks or Tunnel ID for GENEVE/GRE/VXLAN networks
	
--agent <agent-id>
	List networks hosted by agent (ID only)
```

2. **openstack network create**  -- 创建 network

```shell
openstack network create  # 创建 network 
					[--share | --no-share]
                    [--enable | --disable] 
                    [--project <project>]
                    [--description <description>]
                    [--project-domain <project-domain>]
                    [--availability-zone-hint <availability-zone>]
                    [--enable-port-security | --disable-port-security]
                    [--external | --internal]
                    [--default | --no-default]
                    [--qos-policy <qos-policy>]
                    [--transparent-vlan | --no-transparent-vlan]
                    [--provider-network-type <provider-network-type>]
                    [--provider-physical-network <provider-physical-network>]
                    [--provider-segment <provider-segment>]
                    [--tag <tag> | --no-tag]
                    <name>
                    
--availability-zone-hint <availability-zone>
	Availability Zone in which to create this network (Network Availability Zone extension required, repeat option to set multiple availability zones)
	
--enable-port-security
	Enable port security by default for ports created on this network (default)

--disable-port-security
	Disable port security by default for ports created on this network

--default
	Specify if this network should be used as the default external network

--no-default
	Do not use the network as the default external network (default)

--provider-network-type <provider-network-type>
	The physical mechanism by which the virtual network is implemented. The supported options are: flat, geneve, gre, local, vlan, vxlan.

--provider-physical-network <provider-physical-network>
	Name of the physical network over which the virtual network is implemented

--provider-segment <provider-segment>
	VLAN ID for VLAN networks or Tunnel ID for GENEVE/GRE/VXLAN networks

--qos-policy <qos-policy>
	QoS policy to attach to this network (name or ID)

--transparent-vlan
	Make the network VLAN transparent

--no-transparent-vlan
	Do not make the network VLAN transparent
```

3. **opensatck network set**  -- 更新network属性

```shell
opensatck network set <network> # 更新network属性
```

4. **openstack network unset**

```shell
openstack network unset 
```

5. **openstack network delete**  -- 删除指定network

```shell
openstack network delete <network> # 删除指定network
```



### 2. Subnet

subnet 是一个**IPv4**或者**IPv6**地址段，地址段可以在 subnet pool 中获取，subnet 中的 vm 的 ip 是从 subnet 地址段中取，每个subnet需要定义 ip 地址范围和掩码。

network 和 subnet 的关系是一对多的关系，一个subnet只能属于一个network，但是一个network可以有多个subnet，但是每个subnet的网段必须不同。



**openstack subnet 命令：**

> 1. 学会使用 `-h ` 和 `--help`    [>> Reference]( https://docs.openstack.org/python-openstackclient/train/cli/command-objects/subnet.html )

1. **openstack subnet create**  -- 创建subnet  

```shell
openstack subnet create # 创建subnet
					[--project <project>]
                    [--project-domain <project-domain>]
                    [--subnet-pool <subnet-pool> | --use-default-subnet-pool]
                    [--prefix-length <prefix-length>]
                    [--subnet-range <subnet-range>]
                    [--dhcp | --no-dhcp] 
                    [--gateway <gateway>]
                    [--ip-version {4,6}]
                    [--ipv6-ra-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}]
                    [--ipv6-address-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}]
                    [--network-segment <network-segment>] 
                    --network  <network> 
                    [--description <description>]
                    [--allocation-pool start=<ip-address>,end=<ip-address>]
                    [--dns-nameserver <dns-nameserver>]
                    [--host-route destination=<subnet>,gateway=<ip-address>]
                    [--service-type <service-type>]
                    [--tag <tag> | --no-tag]
                    name
                    
--dns-nameserver <dns-nameserver>
	DNS server for this subnet (repeat option to set multiple DNS servers)
	
--host-route destination=<subnet>,gateway=<ip-address>
	Additional route for this subnet e.g.: destination=10.10.0.0/16,gateway=192.168.71.254 destination: destination subnet (in CIDR notation) gateway: nexthop IP address (repeat option to add multiple routes)
```

2. **openstack subnet list**  -- 查询subnet列表

```shell
openstack subnet list # 查询subnet列表
					[--long]
                    [--ip-version <ip-version>] 
                    [--dhcp | --no-dhcp]
                    [--service-type <service-type>]
                    [--project <project>]
                    [--project-domain <project-domain>]
                    [--network <network>] 
                    [--gateway <gateway>]
                    [--name <name>] 
                    [--subnet-range <subnet-range>]
                    [--tags <tag>[,<tag>,...]]
                    [--any-tags <tag>[,<tag>,...]]
                    [--not-tags <tag>[,<tag>,...]]
                    [--not-any-tags <tag>[,<tag>,...]]
```

3. **openstack subnet show** -- 查询subnet详情

```shell
openstack subnet show <subnet>  # 查询subnet详情
```

4. **openstack subnet delete** -- 删除subnet

```shell
openstack subnet delete <subnet>  # 删除subnet详情
```

5. **openstack subnet set** -- 更新 subnet

```shell
openstack subnet set <subnet>  # 更新subnet
					[--name <name>] 
					[--dhcp | --no-dhcp]
                    [--gateway <gateway>]
                    [--description <description>] 
                    [--tag <tag>]
                    [--no-tag]
                    [--allocation-pool start=<ip-address>,end=<ip-address>]
                    [--no-allocation-pool]
                    [--dns-nameserver <dns-nameserver>]
                    [--no-dns-nameservers]
                    [--host-route destination=<subnet>,gateway=<ip-address>]
                    [--no-host-route] 
                    [--service-type <service-type>]
                    <subnet>
```

6. **openstack subnet unset** -- 更新 subnet

```shell
openstack subnet unset # 更新 subnet
					[--allocation-pool start=<ip-address>,end=<ip-address>]
                    [--dns-nameserver <dns-nameserver>]
                    [--host-route destination=<subnet>,gateway=<ip-address>]
                    [--service-type <service-type>]
                    [--tag <tag> | --all-tag]
                     <subnet>
```



### 3. Port

 port 相当于虚拟交换机上的一个端口，port上定义了 MAC 地址和 IP 地址，当 instance 的虚拟网卡 VIF（Virtual Interface） 绑定到 port 时，port 会将 MAC 和 IP 分配给 VIF，port 只有分配给具体设施才有意义

一个subnet可以有多个port，一个port唯一对应一个subnet，一个port对应一个VIF



**openstack port 命令：**

> 学会使用 `-h ` 和 `--help`   [Reference]( https://docs.openstack.org/python-openstackclient/train/cli/command-objects/port.html )

1. **openstack port create**  -- 创建port

```shell
openstack port create # 创建port
				--network <network> 
				[--description <description>]
                [--device <device-id>]
                [--mac-address <mac-address>]
                [--device-owner <device-owner>]
                [--vnic-type <vnic-type>] 
                [--host <host-id>]
                [--dns-name dns-name]
                [--fixed-ip subnet=<subnet>,ip-address=<ip-address>]
                [--binding-profile <binding-profile>]
                [--enable | --disable] 
                [--project <project>]
                [--project-domain <project-domain>]
                [--security-group <security-group> | --no-security-group]
                [--qos-policy <qos-policy>]
                [--enable-port-security | --disable-port-security]
                [--allowed-address ip-address=<ip-address>[,mac-address=<mac-address>]]
                [--tag <tag> | --no-tag]
                <name>
--fixed-ip subnet=<subnet>,ip-address=<ip-address>
	Desired IP and/or subnet for this port (name or ID): subnet=<subnet>,ip-address=<ip-address> (repeat option to set multiple fixed IP addresses)
	
--device <device-id>
	Port device ID

--device-owner <device-owner>
	Device owner of this port. This is the entity that uses the port (for example, network:dhcp).

--vnic-type <vnic-type>
	VNIC type for this port (direct | direct-physical | macvtap | normal | baremetal | virtio-forwarder, default: normal)

--binding-profile <binding-profile>
	Custom data to be passed as binding:profile. Data may be passed as <key>=<value> or JSON. (repeat option to set multiple binding:profile data)

--host <host-id>
	Allocate port on host <host-id> (ID only)
	
--qos-policy <qos-policy>
	Attach QoS policy to this port (name or ID)
	
--dns-name <dns-name>
	Set DNS name for this port (requires DNS integration extension)

--allowed-address ip-address=<ip-address>[,mac-address=<mac-address>]
	Add allowed-address pair associated with this port: ip-address=<ip-address>[,mac-address=<mac-address>] (repeat option to set multiple allowed-address pairs)
```

2. **openstack port list** -- 查询port列表

```shell
openstack port list # 查询port列表
				[--long]
                [--ip-version <ip-version>] 
                [--dhcp | --no-dhcp]
                [--service-type <service-type>]
                [--project <project>]
                [--project-domain <project-domain>]
                [--network <network>] 
                [--gateway <gateway>]
                [--name <name>] 
                [--subnet-range <subnet-range>]
                [--tags <tag>[,<tag>,...]]
                [--any-tags <tag>[,<tag>,...]]
                [--not-tags <tag>[,<tag>,...]]
                [--not-any-tags <tag>[,<tag>,...]]
 
--service-type <service-type>
                List only subnets of a given service type in output
                e.g.: network:floatingip_agent_gateway. Must be a
                valid device owner value for a network port (repeat
                option to list multiple service types)
                
--subnet-range <subnet-range>
                List only subnets of given subnet range (in CIDR
                notation) in output e.g.: --subnet-range 10.10.0.0/16
```

3. **openstack port delete** -- 删除指定port

```shell
openstack port delete <port>  # 删除指定port
```

4. **openstack port show** -- 查询port详情

``` shell
openstack port show <port>  # 查询port详情
```

5. **openstack port set**  -- 更新指定port

```shell
openstack port set <port>  # 更新指定port
```

6. **openstack port  unset**  -- 更新指定port

```shell
openstack port  unset <port> # 更新指定port
```



### 4.Router

router 是一个逻辑概念，用于內部不同子网（不在一个二层域）之间的数据包转发以及将数据发送到外部网络，neutron router 是由 L3 agent 实现，传统L3 agent 只运行在网路节点

**集中式路由：**只在网络节点起 L3 agent 以实现 neutron router，这样导致所有的流量（节点内部，出节点）都会通过网络节点

**分布式路由：**在计算节点和网络都起 L3 agent 以实现 neutron router，计算节点内部的流量在计算节点内部进行转发，出节点的流量走网络节点转发



**openstack router 命令**：

1. **openstack router create**  -- 创建router

```shell
openstack router create # 创建router
					[--enable | --disable]
                    [--distributed | --centralized]
                    [--ha | --no-ha] 
                    [--description <description>]
                    [--project <project>]
                    [--project-domain <project-domain>]
                    [--availability-zone-hint <availability-zone>]
                    [--tag <tag> | --no-tag]
                    <name>
                    
                    
                    
--distributed         
	Create a distributed router # 分布式
	
--centralized         
	Create a centralized router # 集中式
	
--ha
	Create a highly available router
	
--no-ha
	Create a legacy router
```

2.  **openstack router list** -- 查询 router 列表

```shell
openstack router list # 查询 router 列表
```

3. **openstack router show**  -- 查询 router 详情

```shell
openstack router show <router> # 查询 router 详情
```

4. **openstack router set / unset** -- 更新指定 router

```shell
openstack router set / unset <router> # 更新指定 router
```

5. **openstack router delete**  -- 删除指定 router

```shell
openstack router delete <router> # 删除指定 router
```

6.  **openstack router add port**  -- router 关联port

```shell
 openstack router add port <router> <port> 
```

7. **openstack router remove port**  -- router 移除 port

```shell
openstack router remove port <router> <port> 
```

8. **openstack router add subnet** -- router 关联 subnet

```shell
openstack router add subnet <router> <subnet>
```

9. **openstack router add subnet** -- router subnet 解除关联

```shell
openstack router remove subnet <router> <subnet>
```



### 5. firewall

firewall 是由 L3 Agent 实现，它的规则会在 router 所在的节点转换为 ip tables 规则，主要用于保护跨子网的流量保护



**openstack firewall 命令：**

1. **openstack firewall group rule create** / **neutron firewall-rule-create **  ---- 创建防火墙规则 

```shell
openstack firewall group rule create  # 创建rule
								[--name <name>]
                                [--description <description>]
                                [--protocol {tcp,udp,icmp,any}]
                                [--action {allow,deny,reject}]
                                [--ip-version <ip-version>]
                                [--source-ip-address <source-ip-address> | --no-source-ip-address]
                                [--destination-ip-address <destination-ip-address> | --no-destination-ip-address]
                                [--source-port <source-port> | --no-source-port]
                                [--destination-port <destination-port> | --no-destination-port]
                                [--public | --private | --share | --no-share]
                                [--enable-rule | --disable-rule]
                                [--project <project>]
                                [--project-domain <project-domain>]
```

2. **openstack firewall group policy create**  / **neutron firewall-policy-create** -- 创建防火墙策略 

```shell
openstack firewall group policy create # 创建防火墙策略
       								[--description DESCRIPTION]
                                    [--audited | --no-audited]
                                    [--share | --public | --private | --no-share]
                                    [--project <project>]
                                    [--project-domain <project-domain>]
                                    [--firewall-rule <firewall-rule> | --no-firewall-rule]
                                    <name>

```

3. **openstack firewall group create** / **neutron firewall-create** ---- 创建防火墙

```shell
openstack firewall group create # 创建防火墙
							[--name NAME]
                            [--description <description>]
                            [--ingress-firewall-policy <ingress-firewall-policy> | --no-ingress-firewall-policy]
                            [--egress-firewall-policy <egress-firewall-policy> | --no-egress-firewall-policy]
                            [--public | --private | --share | --no-share]
                            [--enable | --disable]
                            [--project <project>]
                            [--project-domain <project-domain>]
                            [--port <port> | --no-port]
```

