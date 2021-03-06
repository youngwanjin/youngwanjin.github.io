---
layout: post
title: OpenStack Neutron LBaaS 
categories: [OpenStack,Neutron ]
description: OpenStack 负载均衡学习
keywords: OpenStack, Loadbalancer

---

### Neutron LBaaS基本概念

**负载均衡**是将来访的网络流量在运行相同应用的多个服务器之间进行分发的一种核心网络服务。其功能有负载均衡器提供，负载均衡器可以是**硬件设备**也可以由**软件实现**。它充当反向代理，在多个服务器之间分发网络或者应用流量。常用来增加应用的访问容量（并发用户数）和可靠性，它也会通过降低服务器的负载来提高应用的总体性能。

#### LBaaS V1  

![](/images/posts/openstack/lbaas-v1.png)

+ **VIP**

  VIP（Virtual IP address）就是负载均衡对外提供服务的地址，VIP有自己的IP地址，而且一般都是可以通过公网访问的，在Neutron中VIP对应二层虚拟设备br-int上的一个port。当负载均衡pool里面至少有一个member时，才有存在的意义，因为此时至少有一个ops实例可以处理网络请求.

+ **Pool**

  Pool 是 LBaaS V1 中的 root resource，其他资源都是基于pool来创建的。Neutron LBaaS 默认以HAProxy为Driver实现。在默认情况下，一个pool对应着一个独立的HAProxy进程，一个独立的namespace。V1 版本一个pool只能有一个VIP，在V2 版本中，允许pool对应多个VIP。

+ **Member**

  Member 对应的是pool里面用来处理网络请求的ops实例，在OpenStack  Neutron 中，member 是一个逻辑关系，表示着实例与pool的对应关系，即：一个ops实例，可以对应不同的pool，在Neutron LBaaS 里面可以创建多个member，在创建member对应的实例时必须与pool属于同一个subnet，否则将不能工作。

+ **Health Monitor**

  Hm 只有在关联pool时才有意义，它是用来检测 pool 里面 member 的状态。他会以轮询的方式，去查询各个member，如果member未作出响应，它会更新member的状态为 INACTIVE ，这样在 VIP 分发网络请求的时候，就会不考虑状态为 INACTIVE 的 member，它也会更新 member 的状态至  ACTIVE ，这时， member 会重新出现在VIP的分发列表中。

  Hm 在 Neutron 的负载均衡服务中不是必须的，即：没有Hm，也可以组成一个负载均衡池，只是没有hm的检查pool会一直认为所有的member都是ACTIVE 状态，这样member会一直出现在VIP的分发列表中，这样会造成负载均衡的响应异常。

#### LBaaS V2

![](/images/posts/openstack/lbaas-v2.png)

+ **Load Balancer**:  负载均衡服务的 root source ，同时也是VIP关联的逻辑对象。一个LB可以拥有一个或多个VIP，VIP可以是Neutron Subnet 的一个Port，并从subnet 中分配 IP。
+ **Listener**:  用于监听客户端对 LB（VIP）的访问请求，监听项为 HTTP/HTTPS、TCP协议的元素，但是不监听IP地址。只有符合监听规则的访问请求才会被转发到与Listener关联的Pool中。一个LB可有多个Listener，一个 Listener 也可以关联多个 Pool。
+ **Pool**: 作为 Member 的容器，用于接收Listener分发过来的客户端的请求。通常，将业务类型相近的云主机规划到同一个Pool内。
+ **Member**：实际运行用户业务的云主机，包含在一个Pool中，具有权重属性。每个成员均由其用于服务的IP地址+端口号指定。
+ **Health Monitor**：Member 的 Health Check与 Pool 关联，通过一些算法对pool中的member定期做健康检查，并同步状态。这是一个可选项，如果没有则pool会一直认为member是ACTIVE状态，会造成负载均衡服务异常。
+ **Amphora**：提供负载均衡的服务的实体，在用户创建 Load Balancer 时自动创建。
+ **HAProxy**: 运行在Amphora中的提供负载均衡服务的进程。



#### 负载均衡器分类

+ **第4层负载均衡器**

  基于网络和传输层协议（IP、TCP、FTP、UDP）来实现负载均衡。

+ **第7层负载均衡**

  基于应用层协议（HTTP、SMTP、FTP、Telnet）来实现负载均衡。对于HTTP来说，第7层的负载均衡器能根据应用层的特定数据比如HTTP头，cookies或者应用消息中的数据来进一步做请求分发。

  

#### 负载分发算法

+ **轮询（ Round robin ）**： 轮流分发到各个（ACTIVE）服务器。
+ **加权轮询（ Weighted round robin ）**：每个服务器都是有一定的加权（weight），轮询时考虑加权值。
+ **最少连接（ Least connections ）**：转发到有最少连接数的服务器。
+ **最少响应时间（ Least response time ）**：转发到响应时间最短的服务器。



#### 可靠性和可用性

负载均衡器通过监控应用的健康状况来确保可靠性和可用性，并且只转发请求到能够及时做出响应的服务和应用（member）。



#### 会话保持（ Session Persistence ）

用户（浏览器）在和服务器交互时，通常会在本地保存一些信息，而整个过程称之为一个会话（Session）并用唯一的Session ID 进行标识。因为HTTP协议是无状态的，所以任何需要上下文逻辑的情形都必须使用会话机制，HTTP客户端也会额外缓存一些数据在本地，可以减少请求，从而提高性能。负载均衡可能将不同的请求转发到不同的后端，这样会影响性能，所以就需要将同一个会话的请求都转到相同的后端。

会话保持 表示在一个会话期间，转发一个用户的请求到同一个后端服务器，

+ 会话保持方法
  + Source IP：相同来源的请求转发到同一后端服务器。
  + HTTP Cookie：该模式下，loadbalancer为客户端的第一次连接生成cookie，后续携带该Cookie的请求会被同一个member处理。
  + APP Cookie：该模式下，依靠后端应用服务器生成的cookie决定被某个member处理。



### 负载均衡实现方法







----

学习记录，如有冒犯，及时指正。

+ [Ref1]( https://www.ibm.com/developerworks/cn/cloud/library/1506_xiaohh_openstacklb/index.html )
+ [Ref2]( https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html )
+ [Ref3](http://luckylau.tech/2017/03/07/Openstack负载均衡LoadBalancerv2/) 