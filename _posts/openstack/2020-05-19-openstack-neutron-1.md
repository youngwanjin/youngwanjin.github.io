---
layout: post
title: OpenStack Neutron 简单介绍
categories: [OpenStack, Neutron ]
description: OpenStack Neutron 简单介绍
keywords: OpenStack, Neutron 

---

### neutron 功能

 neutron为openstack提供网络支持，包括二层交换、三层路由、负载均衡、防火墙、vpn等。

### neutron 架构

#### 1. 概述

![6194d4c14cbd2f0925b48791ba7af88d.png](en-resource://database/951:1)

- **Neutron Server** ： 对外提供OpenStack网络的API，接收请求，并调用Plugin处理请求。
- **Plugin**：处理 Neutron Server 发来的请求，维护 OpenStack 逻辑网络的状态，并调用 Agent 处理请求。

+ **Agent**：处理 Plugin 的请求，负责在 network provider 上真正的实现各种网络功能。

+ **network provider**：提供网络服务的虚拟或物理的网络设备。如：Linux Bridge，Open vSwitch 或 其他物理交换机。

+ **Queue**： Neutron Server、Plugin、Agent 之间使用mq通信和调用。

+ **Database**： 用来存放OPenStack的网络状态信息，包括network、subnet、port、router

#### 1. Neutron Server

![618919de9092fbd3a7749002b6b78b26.png](en-resource://database/953:1)

+ **Core API** : 对外提供管理 network、subnet、port 的TESTful API
+ **Extension API** ： 对外提供router、load balancer、firewall 等资源的RESful API
+ **Common Service**：认证和校验API请求
+ **Neutron Core** ：Neutron server 的核心处理程序，通过调用相应的Plugin处理请求。
+ **Core Plugin API**：定义了Core Plugin 的抽象功能合集，Neutron Core 通过API调用相应的Core Plugin。
+ **Extension Plugin API**：定义了Service Plugin 的抽象功能合集，Neutron Core 通过API调用相应的Service Plugin。
+ **Core Plugin** ：实现了Core Plugin API 在数据库中维护network、subnet和port的状态，并负责调用相应的agent在network provider上执行相关操作
+ **Service Plugin** ： 实现了Extension Plugin API 在数据库中维护router、load balancer、security group 并负责调用相应的agent在network provider上执行相关操作

#### 2. 架构展开

![4f271ff4f7a534703800b04fbd34e25e.png](en-resource://database/957:0)

