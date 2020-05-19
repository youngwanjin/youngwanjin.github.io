---
layout: post
title: OpenStack Neutron LBaaS 
categories: [OpenStack,Neutron ]
description: OpenStack 负载均衡学习
keywords: OpenStack, Loadbalancer

---

### 1. Neutron LBaaS基本概念

**负载均衡**是将来访的网络流量在运行相同应用的多个服务器之间进行分发的一种核心网络服务。其功能有负载均衡器提供，负载均衡器可以是**硬件设备**也可以由**软件实现**。它充当反向代理，在多个服务器之间分发网络或者应用流量。常用来增加应用的访问容量（并发用户数）和可靠性，它也会通过降低服务器的负载来提高应用的总体性能。

#### 1. LBaaS V1  

![](https://github.com/youngwanjin/youngwanjin.github.io/blob/master/images/posts/openstack/lbaas-v1.png)

+ **VIP**

  VIP（Virtual IP address）就是负载均衡对外提供服务的地址，VIP有自己的IP地址，而且一般都是可以通过公网访问的，在Neutron中VIP对应二层虚拟设备br-int上的一个port。当负载均衡pool里面至少有一个member时，才有存在的意义，因为此时至少有一个ops实例可以处理网络请求.

+ **Pool**

  Pool 是 LBaaS V1 中的 root resource，其他资源都是基于pool来创建的。Neutron LBaaS 默认以HAProxy为Driver实现。在默认情况下，一个pool对应着一个独立的HAProxy进程，一个独立的namespace。V1 版本一个pool只能有一个VIP，在V2 版本中，允许pool对应多个VIP。

+ **Member**

  Member 对应的是pool里面用来处理网络请求的ops实例，在OpenStack  Neutron 中，member 是一个逻辑关系，表示着实例与pool的对应关系，即：一个ops实例，可以对应不同的pool，在Neutron LBaaS 里面可以创建多个member，在创建member对应的实例时必须与pool属于同一个subnet，否则将不能工作。

+ **Health Monitor**

  Hm 只有在关联pool时才有意义，它是用来检测 pool 里面 member 的状态。他会以轮询的方式，去查询各个member，如果member未作出响应，它会更新member的状态为 INACTIVE ，这样在 VIP 分发网络请求的时候，就会不考虑状态为 INACTIVE 的 member，它也会更新 member 的状态至  ACTIVE ，这时， member 会重新出现在VIP的分发列表中。

  Hm 在 Neutron 的负载均衡服务中不是必须的，即：没有Hm，也可以组成一个负载均衡池，只是没有hm的检查pool会一直认为所有的member都是ACTIVE 状态，这样member会一直出现在VIP的分发列表中，这样会造成负载均衡的响应异常。

#### 2. LBaaS V2

![](https://github.com/youngwanjin/youngwanjin.github.io/blob/master/images/posts/openstack/lbaas-v2.png)

+ **Load Balancer**:  负载均衡服务的 root source ，同时也是VIP关联的逻辑对象。一个LB可以拥有一个或多个VIP，VIP可以是Neutron Subnet 的一个Port，并从subnet 中分配 IP。
+ **Listener**:  用于监听客户端对 LB（VIP）的访问请求，监听项为 HTTP/HTTPS、TCP协议的元素，但是不监听IP地址。只有符合监听规则的访问请求才会被转发到与Listener关联的Pool中。一个LB可有多个Listener，一个 Listener 也可以关联多个 Pool。
+ **Pool**: 作为 Member 的容器，用于接收Listener分发过来的客户端的请求。通常，将业务类型相近的云主机规划到同一个Pool内。
+ **Member**：实际运行用户业务的云主机，包含在一个Pool中，具有权重属性。每个成员均由其用于服务的IP地址+端口号指定。
+ **Health Monitor**：Member 的 Health Check与 Pool 关联，通过一些算法对pool中的member定期做健康检查，并同步状态。这是一个可选项，如果没有则pool会一直认为member是ACTIVE状态，会造成负载均衡服务异常。
+ **Amphora**：提供负载均衡的服务的实体，在用户创建 Load Balancer 时自动创建。
+ **HAProxy**: 运行在Amphora中的提供负载均衡服务的进程。



----

+ [Ref1]( https://www.ibm.com/developerworks/cn/cloud/library/1506_xiaohh_openstacklb/index.html )
+ [Ref2]( https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html )