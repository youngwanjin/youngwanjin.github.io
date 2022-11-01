---
layout: post
title: CentOS 安装 Docker
categories: [Docker]
description: none
keywords: Docker
---

# CentOS 安装 Docker

> Centos 系统版本:  CentOS 7.6 64bit  （Tencent Cloud）

```shell
1. 查看 linux 内核版本
# uname -r 
3.10.0-1160.71.1.el7.x86_64

2. 更新 yum 包(生产环境谨慎使用)
yum -y update
注意:
yum -y update: 升级所有包同时也升级软件和系统内核
yum -y upgrade: 只升级所有包，不升级软件和系统内核

3. 卸载旧版本的 docker (如果已经安装过)
yum remove -y docker  docker-common docker-selinux docker-engine
```

```shell
1. 安装所需的工具包
yum install -y yum-utils device-mapper-persistent-data lvm2

2. 设置 docker yum 源
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo（中央仓库）

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（阿里仓库）

3. 查看 docker 的可用版本(找到合适版本安装)
yum list docker-ce --showduplicates | sort -r

4. 选择版本安装
yum -y install docker-ce-20.10.18-3.el7

5. 启动 docker 并设置开机自启
systemctl start docker
systemctl enable docker
```

[Reference>>](https://yeasy.gitbook.io/docker_practice/install/centos) 
