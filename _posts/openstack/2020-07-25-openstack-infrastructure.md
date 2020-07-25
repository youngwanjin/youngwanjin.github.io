---
layout: post
title: OpenStack 基础架构
categories: [OpenStack]
description: OpenStack 基础架构学习
keywords: OpenStack
---

# OpenStack 基础架构

### 基础介绍

openstack 主要提供计算、存储、网络的能力，由控制节点、计算节点、网络节点、存储节点四部分组成（四个节点可以安装在一台机器上，单机部署）

+ **控制节点：**主要负责对其余节点进行控制，包含虚拟机的创建、迁移，网络分配，存储分配等
+ **计算节点**：主要负责虚拟机的运行
+ **网络节点**：主要负责对外网络及内部网络之间的通信
+ **存储节点**：主要负责对虚拟机额外存储的管理

openstack 详细架构图：

![](/images/posts/openstack/openstack_architecture.png)

openstack 网络拓扑图：

![](/images/posts/openstack/openstack_network_topology.png)

### 组件介绍

+ **Horizon**： Dashboard， 以web服务方式管理云平台，创建云主机，分配网络，配安全组，添加磁盘等操作
+ **Nova**：计算， 负责响应虚拟机创建、调度、销毁等请求
+ **Neutron**：网络， 实现SDN（软件定义网络），提供一整套API，用户可以基于该API实现自己定义专属网络，不同厂商可以基于此API提供自己的产品实现 
+ **Keystone**： 认证服务，为访问openstack各组件提供认证和授权功能，认证通过后，提供一个服务列表（存放你有权访问的服务），可以通过该列表访问各个组件，即：负责认证、鉴权以及endpoint管理
+ **Glance**： 镜像服务， 为云主机安装操作系统提供不同的镜像
+ **Ceilometer**：监控服务， 收集云平台资源使用数据，用来计费或者性能监控
+ **Swift**：对象存储
+ **Cinder**：块存储，提供持久化块存储，即为云主机提供附加云盘 
+ **Heat**：编排服务，自动化部署应用，自动化管理应用的整个生命周期.主要用于Paas  



openstack 组件关系图：

![](/images/posts/openstack/openstack_component_relationship.png)

openstack 主要组件逻辑架构图：



![](/images/posts/openstack/openstack_component_logical_structure.png)

[Reference1 >> ]( https://www.cnblogs.com/klb561/p/8660264.html )

[Reference2 >>]( https://www.cnblogs.com/xiugeng/p/9668102.html )

[Reference3 >>]( https://m.yisu.com/zixun/10539.html )

