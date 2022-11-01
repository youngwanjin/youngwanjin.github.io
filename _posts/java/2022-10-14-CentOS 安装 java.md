---
layout: post
title: CentOS 安装 java
categories: [JAVA]
description: none
keywords: JAVA
---

# CentOS 安装 java 

> Centos 系统版本:  CentOS 7.6 64bit  （Tencent Cloud）
>
> java 语言版本:  `openjdk 1.8` 
>
> 官方安装 ： https://openjdk.org/install/ 

## yum 安装

```shell
1. 查看 java 的版本
# java -version 

2. 查看已经安装的 jdk 
# yum list installed | grep java
# yum list installed | grep jdk

3. 卸载已经安装的 openjdk 
# yum remove java-1.8.0-openjdk* -y 

4. 查看 yum 源里面的 openjdk 
# yum search java|grep jdk  /  yum search openjdk  /  yum list | grep openjdk  /  yum list *openjdk*

5. 查看系统的版本
# cat /etc/system-release  /  cat /etc/centos-release  /  cat /etc/redhat-release 

6. 选择版本进行 yum 安装
# yum install -y java-1.8.0-openjdk*

7. 查看 java 的版本
# java -version  /  java  /  javac 
```

> 查看 java 路径：`find / -name java` 
>
> 默认安装路径：`/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.345.b01-1.el7_9.x86_64/jre/bin/java` 
>
> 参考文档：https://timberkito.com/?p=12 

## 从 Oracle 获取安装包手动安装

[Reference >>](https://timberkito.com/?p=12 ) 





