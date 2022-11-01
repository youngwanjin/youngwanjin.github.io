---
layout: post
title: CentOS 安装 mvn 环境
categories: [JAVA]
description: none
keywords: JAVA
---

# CentOS 安装 mvn 环境

> Centos 系统版本:  CentOS 7.6 64bit  （Tencent Cloud）
>
> mvn 版本:  3.8.6
>

```shell
1. 进入安装目录
cd /usr/local/

2. 下载 tar.gz 包 
官网: https://maven.apache.org/download.cgi
下载连接: https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz

3. 上传包到 /usr/local/ 并解压
tar -xvzf apache-maven-3.8.6-bin.tar.gz

4. 删除 tar.gz 包
rm -rf apache-maven-3.8.6-bin.tar.gz

5. 添加环境变量
vim /etc/profile

# mvn
MAVEN_HOME=/usr/local/apache-maven-3.8.6
export PATH=${MAVEN_HOME}/bin:${PATH}

6. 是环境变量生效
source /etc/profile

7. 验证是否安装成功
mvn -v
```

配置 `mvn` 源: 

`vim /usr/local/apache-maven-3.8.6/conf/settings.xml`  添加如下内容：

```xml
<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
 </mirror> 
```

配置 `JDK` 版本:

```xml
    <profile>    
     <id>jdk-1.8</id>    
     <activation>    
       <activeByDefault>true</activeByDefault>    
       <jdk>1.8</jdk>    
     </activation>    
       <properties>    
         <maven.compiler.source>1.8</maven.compiler.source>    
         <maven.compiler.target>1.8</maven.compiler.target>    
         <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>    
       </properties>    
    </profile>
```

创建本地 `repo` 目录：

```shell
mkdir  /data/maven_repo 
```

```xml
<localRepository>/data/maven_repo</localRepository>
```





