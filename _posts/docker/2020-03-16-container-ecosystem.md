---
layout: post
title: 容器生态系统
categories: [Docker]
description: 每天5分钟玩转Docker容器技术 - 容器生态系统
keywords: Docker
---

### 容器生态系统包含：

* 容器核心技术

* 容器平台技术

* 容器支持技术

  

### 1. 容器核心技术

容器核心技术是指能够让 Container 在 host 上运行起来的技术。

* 容器规范

* 容器 runtime

* 容器管理工具

* 容器定义工具

* Registries

* 容器OS

  

#### 1.1  容器规范

由 Docker、CoreOS、Google 共同成立了一个叫 Open Container Initiative (OCI) 的组织，用于制定容器规范。

* runtime spec 
* image format spec 



#### 1.2 容器 runtime

runtime 是容器真正运行的地方，runtime 需要跟操作系统的 kernel 紧密协作，为容器提供运行时环境(类似于 JVM)。

* lxc
* runc
* rkt

>1. lcx 是 Linux 上的老牌 runtime，Docker 最初也是使用 lxc 做为 runtime。
>
>2. runc 是 Docker 自己开发的 runtime，符合 oci 规范，也是现在 Docker 的默认 runtime。
>
>3. rkt 是 CoreOS 开发的容器 runtime ，符合 oci 规范，也可以作为 Dokcer 的 runtime。



#### 1.3 容器管理工具

只有 runtime 是不够的，用户还需要有工具来管理容器。容器管理工具对内与 runtime 进行交互，对外为用户提供 interface ，比如 CLI。

* lxd 

* docker engine

  * deamon

  * cli

* rkt cli

> 1. lxd 是 lxc 对应的管理工具。
>
> 2. runc 的管理工具是 docker engine。docker engine 包含后台 deamon 和 cli 两部分。
>
> 3. rkt 对应的管理工具是 rkt cli。



#### 1.4 容器定义工具

容器定义工具允许用户定义容器的内容和属性，这样容器就能够被保存、共享和重建。

* docker image
* dockerfile
* ACI （App Container Image）

> 1. dokcer image 是 Docker 容器的模板，runtime 依据 docker image 创建容器。
>
> 2. dockerfile 是包含若干命令的文本文件，可以通过这些命令创建出 docker image。
>
> 3. ACI 与 docker image 类似，只是它是由 CoreOS 开发的 rkt 容器的 image 格式。 



#### 1.5 Registry

容器是通过 image 创建的，需要有一个仓库来统一存放 image，这个仓库叫做 Registry。

* Docker Registry
* Docker Hub
* Quay.io

> 1. 企业可以使用 Docker Registry 构建私有的 Registry。
>
> 2. Docker Hub（https://hub.docker.com）是 Docker 为用户提供的托管 Registry ，上面有很多可用的 image。
>
> 3. Quay.io (https://quay.io) 是另一个托管 registery。



#### 1.6 容器 OS

专门运行容器的操作系统。

* coreos
* atomic
* ubuntu core



### 2. 容器平台技术

容器核心技术使得容器可以在单个的 host 上运行，而容器平台技术就是让容器能够作为集群在分布式环境中运行。

* 容器编排引擎
* 容器管理平台
* 基于容器的 PaaS



#### 2.1 容器编排引擎

基于容器的应用，一般都会采用微服务架构。这种架构之下，应用被划分成不同的组件，并以服务的形式运行在各自的容器中，通过 API 对外提供服务。为了保证服务的可用性，每个组件都可能是以集群的方式运行，集群中的容器会根据业务的需要动态的创建、迁移、销毁。

所谓编排，通常包括容器管理、调度、集群的定义和服务发现等，通过编排引擎，容器被有机的组合成微服务，实现业务需求。

* docker swarm
* kubernetes 
* mesos + marathon

> 1. docker swarm 是 Docker 开发的容器编排引擎。
>
> 2. kubernetes 是 Google 开发的开源容器编排引擎，支持 Docker 和 CoreOS 。
>
> 3. mesos 是一个通用的集群资源调度平台，与 marathon 一起提供容器编排引擎的功能。



#### 2.2 容器管理平台

容器管理平台是架构在容器编排引擎之上的一个更为通用的平台。通常容器管理平台能够支持多种编排引擎，抽象了编排引擎的底层实现细节，为用户提供更方便的功能，例如：application catalog 和 一键应用部署。

* Rancher
* ContainerShip



#### 2.3 基于容器的PaaS

基于容器的 PaaS 为微服务应用开发人员和公司提供了开发、部署和管理应用的平台，使用户不必关心底层基础设施而专注于应用开发。

* Deis
* Flynn
* Dokku



### 3. 容器支持技术

* 容器网络
* 服务发现
* 监控
* 数据管理
* 日志管理
* 安全性



#### 3.1 容器网络

容器的出现使得网络的拓扑结构更加的动态和复杂。用户需要专门的解决方案来管理容器与容器、容器与其他实体之间的连通性和隔离性。

* docker network 
* flannel
* weave
* calico



#### 3.2 服务发现

动态伸缩是微服务应用的特点之一，负载增大，集群就会自动创建新的容器；负载减小，集群就会自动销毁对于的容器。容器也会根据 host 的资源使用情况在不同的 host 之间迁移，容器的 IP 和端口也会随之变化。这种情况我们就需要一种机制让 client 能够知道如何访问容器提供的服务。这就需要服务发现机制。

* etcd
* consul
* zookeeper



#### 3.3 监控

监控对于基础架构非常重要，而容器的动态特征对监控提出了更多的挑战。

* docker ps/top/stats (docker 原生)
* docker stats API (docker 原生)
* sysdig
* cAdvisor / Heapster
* Weave Scope



#### 3.4 数据管理

容器经常在不同的的 host 之间迁移，如何保证持久化数据也能够动态迁移。

*  Rex-Ray



#### 3.5 日志管理

* docker logs
* logspout

> 1. docker logs 是 Docker 原生的日志工具
> 2.  logspout 对日志提供了路由功能，可以收集不同容器的日志并转发给其他工具进行处理



#### 3.6 安全性

对于容器的安全性，一直深受争议。

* OpenSCAP

> OpenSCAP 能够对容器镜像进行扫描，发现潜在漏洞

