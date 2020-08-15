---
layout: post
title: OpenStack 原生环境搭建(Pike)
categories: [OpenStack]
description: some word here
keywords: OpenStack
---

# OpenStack 环境搭建

> OpenStack 版本：Pike

 ### 环境准备

| 节点       | 系统     | ip            | 描述     |
| ---------- | -------- | ------------- | -------- |
| controller | CentOS 7 | 192.168.8.212 | 控制节点 |
| compute    | CnetOS 7 | 192.168.8.213 | 计算节点 |

#### 配置 ip

[Referance >> ]( https://youngwanjin.github.io/2020/08/11/centos-add-ip/ )



#### 修改主机名

controller 节点：

```shell
[root@localhost ~]# vim /etc/hostname   
controller
# 重启主机
[root@localhost ~]# reboot
```

compute 节点：

```shell
[root@localhost ~]# vim /etc/hostname
compute
# 重启主机
[root@localhost ~]# reboot
```



#### 修改 host 文件

compute 和 controller 节点做相同的配置，方便节点之间互相登录

```shell
[root@controller ~]# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.8.212 controller
192.168.8.213 compute
```



#### 关闭防火墙 + 关闭 selinux

controller 节点:

```shell
[root@controller ~]# systemctl stop firewalld.service
[root@controller ~]# systemctl disable firewalld.service
[root@controller ~]# vim /etc/sysconfig/selinux
SELINUX=disabled
```

compute 节点:

```shell
[root@compute ~]# systemctl stop firewalld.service
[root@compute ~]# systemctl disable firewalld.service
[root@compute ~]# vim /etc/sysconfig/selinux
SELINUX=disabled
```



#### 同步主机时间(NTP)

##### controller 节点

###### 安装软件包

```shell
[root@controller ~]# yum install chrony -y
```

###### 修改配置文件

`/etc/chrony.conf`

```shell
[root@controller ~]# vim /etc/chrony.conf
server NTP_SERVER iburst
```

> NTP_SERVER : 合适的时间服务器

```shell
allow 102.168.0.0/16
```

###### 重启服务

```shell
[root@controller ~]# systemctl enable chronyd.service
[root@controller ~]# systemctl start chronyd.service
```



##### compute 节点

 ###### 安装软件包

```shell
[root@compute ~]# yum install chrony -y
```

###### 修改配置文件

`/etc/chrony.conf`

```shell
[root@compute ~]# vim /etc/chrony.conf
server controller iburst 
```

###### 启动服务

```shell
[root@compute ~]# systemctl enable chronyd.service
[root@compute ~]# systemctl start chronyd.service
```

##### 验证

###### controller 节点

```shell
[root@controller ~]# chronyc sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^+ ntp6.flashdance.cx            2   6   367    60    -23ms[  -16ms] +/-  213ms
^* tock.ntp.infomaniak.ch        1   6   277    57   -703us[+6042us] +/-   93ms
^+ ntp1.flashdance.cx            2   6   377    58  +1557us[+8299us] +/-  204ms
^+ stratum2-1.ntp.led01.ru.>     2   6   261    58    +12ms[  +18ms] +/-   94ms
```

###### cpmoute 节点

```shell
[root@compute ~]# chronyc sources
210 Number of sources = 5
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? ntp.wdc1.us.leaseweb.net      2   6    11    52    -19ms[  -19ms] +/-  243ms
^+ stratum2-1.ntp.led01.ru.>     2   6    17    58  +9474us[  +11ms] +/-   97ms
^? ntp5.flashdance.cx            0   6     0     -     +0ns[   +0ns] +/-    0ns
^+ ntp1.flashdance.cx            2   6    17    58  +1383us[+2534us] +/-  218ms
^* controller                    2   6    17    58    -11ms[-9668us] +/-  104ms
```



### 安装 openstack 软件包

> 配置合适的 yum 源
>
> ```shell
> tee /etc/yum.repos.d/pike-75.repo << EOF
> [pike-75]
> name=pike-75
> baseurl=http://192.168.8.200/pike/rpms7.5
> enabled=1
> gpgcheck=0
> EOF
> ```

#### controller 节点

```shell
[root@controller ~]# yum install -y openstack-utils openstack-selinux python-openstackclient
[root@controller ~]# yum upgrade -y
```

#### compute 节点

```shell
[root@compute ~]# yum install -y openstack-utils openstack-selinux python-openstackclient
[root@compute ~]# yum upgrade -y
```

> 注： 此处使用的是制作的本地 yum 源，安装命令有所区别，详情参见官方文档
>
> [Pike 版本rpm包]( http://vault.centos.org/7.5.1804/cloud/x86_64/ )



### 安装 SQL 数据服务

#### controller 节点

##### 安装软件包

```shell
[root@controller ~]# yum install mariadb mariadb-server python2-PyMySQL -y
```

##### 修改配置文件

`/etc/my.cnf.d/openstack.cnf`

```shell
[root@controller ~]# vim /etc/my.cnf.d/openstack.cnf
[mysqld]
bind-address = 192.168.8.212  # 控制节点的管理 ip 

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

##### 启动 SQL 服务

```shell
[root@controller ~]# systemctl enable mariadb.service
[root@controller ~]# systemctl start mariadb.service
[root@controller ~]# mysql_secure_installation
```

> 以 root 启动mysql数据库，注意看提示设置数据库密码，其他选项皆为 yes；密码为：root 



### 安装 message queue 服务

#### controller 节点

##### 安装 rpm 包

```shell
[root@controller ~]# yum install rabbitmq-server -y
```

##### 启动 mq 服务

```shell
[root@controller ~]# systemctl enable rabbitmq-server.service
[root@controller ~]# systemctl start rabbitmq-server.service
```

##### 创建 openstack 用户

```shell
[root@controller ~]# rabbitmqctl add_user openstack RABBIT_PASS
```

> RABBIT_PASS :  openstack 用户密码 ： openstack

##### 配置访问权限

```shell
[root@controller ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/" ...
```



### 安装 Memcached 服务

服务的身份验证机制使用Memcached来缓存令牌，memcached服务通常在控制器节点上运行，对于生产部署，我们建议启用防火墙，身份验证和加密的组合以保护其安全

#### controller 节点

##### 安装 rpm 包

```shell
[root@controller ~]# yum install memcached python-memcached -y
```

##### 修改配置文件

`/etc/sysconfig/memcached`

```shell
[root@controller ~]# vim /etc/sysconfig/memcached

PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1,controller"
```

##### 启动服务

```shell
[root@controller ~]# systemctl enable memcached.service
[root@controller ~]# systemctl start memcached.service
```



### 安装 Etcd 服务

#### controller 节点

##### 安装 rpm 包

```shell
[root@controller ~]# yum install etcd -y
```

##### 修改配置文件

`/etc/etcd/etcd.conf`  `ETCD_INITIAL_CLUSTER`, `ETCD_INITIAL_ADVERTISE_PEER_URLS`, `ETCD_ADVERTISE_CLIENT_URLS`, `ETCD_LISTEN_CLIENT_URLS` 

```shell
[root@controller ~]# vim /etc/etcd/etcd.conf

#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.8.212:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.8.212:2379"
ETCD_NAME="controller"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.8.212:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.8.212:2379"
ETCD_INITIAL_CLUSTER="controller=http://192.168.8.212:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
```

##### 启动服务

```shell
[root@controller ~]# systemctl enable etcd
[root@controller ~]# systemctl start etcd
```



### 安装 openstack 服务

#### Keystone 服务(controller node)

##### 建立数据库

```shell
[root@controller ~]# mysql -u root -p
```

> 密码： mysql 数据库密码，`root`

```shell
MariaDB [(none)]> CREATE DATABASE keystone;
```

```shell
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    -> IDENTIFIED BY 'KEYSTONE_DBPASS';
    
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    -> IDENTIFIED BY 'KEYSTONE_DBPASS';
```

> KEYSTONE_DBPASS：keystone 数据库密码：`keystone`



##### 安装配置 keystone 服务

###### 安装 rpm 包

```shell
[root@controller ~]# yum install openstack-keystone httpd mod_wsgi -y
```

###### 修改配置文件

`/etc/keystone/keystone.conf`

```shell
[root@controller ~]# vim /etc/keystone/keystone.conf

[database]
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone

[token]
provider = fernet
```

> KEYSTONE_DBPASS : keystone 数据库密码：keystone

###### 执行脚本生成数据表

```shell
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone
```

###### 初始化密钥库

```shell
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

###### 启动 keystone 服务

```shell
[root@controller ~]# keystone-manage bootstrap --bootstrap-password ADMIN_PASS --bootstrap-admin-url http://controller:35357/v3/  --bootstrap-internal-url http://controller:5000/v3/  --bootstrap-public-url http://controller:5000/v3/   --bootstrap-region-id RegionOne
```

> ADMIN_PASS: keystone 服务密码 ： keystone



##### 安装配置 apache 服务

###### 修改配置文件

`/etc/httpd/conf/httpd.conf`

```shell
[root@controller ~]# vim /etc/httpd/conf/httpd.conf
ServerName controller 
```

###### 创建软链接

`/usr/share/keystone/wsgi-keystone.conf` `/etc/httpd/conf.d/`

```shell
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

##### 完成安装

###### 启动 http server

```shell
[root@controller ~]# systemctl enable httpd.service
[root@controller ~]# systemctl start httpd.service
```

###### 配置 admin 账户

```shell
[root@controller ~]# export OS_USERNAME=admin
[root@controller ~]# export OS_PASSWORD=ADMIN_PASS
[root@controller ~]# export OS_PROJECT_NAME=admin
[root@controller ~]# export OS_USER_DOMAIN_NAME=Default
[root@controller ~]# export OS_PROJECT_DOMAIN_NAME=Default
[root@controller ~]# export OS_AUTH_URL=http://controller:35357/v3
[root@controller ~]# export OS_IDENTITY_API_VERSION=3
```

> ADMIN_PASS : admin 用户密码 ： keystone



##### 创建 domain, projects, users, roles

 Create the `service` project:

```shell
[root@controller ~]# openstack project create --domain default --description "Service Project" service 
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 24194e8a86864711bbf8bbd32b1047a5 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
+-------------+----------------------------------+
```

 Create the `demo` project: 

```shell
[root@controller ~]# openstack project create --domain default --description "Demo Project" demo
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | b6671b7ce38a44cb901d1f0b12e4ca42 |
| is_domain   | False                            |
| name        | demo                             |
| parent_id   | default                          |
+-------------+----------------------------------+
```

 Create the `demo` user: 

```shell
[root@controller ~]# openstack user create --domain default \
>   --password-prompt demo
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 70a67421d62a4cb085a08deafb54b418 |
| name                | demo                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> User Password: keystone

 Create the `user` role: 

```shell
[root@controller ~]# openstack role create user
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | d2efda8be55349fab9ba5f50c1de6778 |
| name      | user                             |
+-----------+----------------------------------+
```

 Add the `user` role to the `demo` project and user: 

```shell
[root@controller ~]# openstack role add --project demo --user demo user
```



##### 验证 keystone 服务

###### 取消  OS_AUTH_URL  和  OS_PASSWORD  环境变量

```shell
[root@controller ~]# unset OS_AUTH_URL OS_PASSWORD
```

###### admin 用户获取 token

```shell
[root@controller ~]#  openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
Password: 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-14T07:23:03+0000                                                                                                                                                                |
| id         | gAAAAABfNi3HVs9cf-bWxrI7xkIPg08cCxj_VLj8bhr4JcrYkq1MCiEMzsQh91hW8kl8nDTDGj72Eb9K6-hpUWRreLlzSB4Pe-MsLz5q5kOvRACUeQPucnxDh5J_3WCGwKgnnXimhRM5GG1vvrtqG7fHcJA8qyVaQ7FFxy_UzthzKBkcXL6euXw |
| project_id | a10a48cffde949d9bd00f95955cd0c65                                                                                                                                                        |
| user_id    | c84cdffc89c74cd6819d3c02f1e44f36                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

> Password : admin 用户密码 keystone

###### demo 用户获取 token

```shell
[root@controller ~]# openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name demo --os-username demo token issue
Password: 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-14T07:24:38+0000                                                                                                                                                                |
| id         | gAAAAABfNi4mw1Ic22jRGqvvWKTfhJjFwwdvwAnT42eKza4P4lbHswMlWfGJ7DVk7HW03qYts4edsD0jEHyy3zG53EF1wsYxt5ykpnbjhtdQCSh3opf-O8gG51epBz2AAK0GpyvZxSCEyq92yralv9S-Pk0ZNqiTgdJHK8Rz5YWTSzHvCSKk890 |
| project_id | b6671b7ce38a44cb901d1f0b12e4ca42                                                                                                                                                        |
| user_id    | 70a67421d62a4cb085a08deafb54b418                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

> Password : demo 用户密码 ： keystone

 

##### 创建 openstack 客户端脚本

###### 创建编辑脚本

`admin-openrc.sh`

```shell
[root@controller ~]# vim admin-openrc.sh

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

> ADMIN_PASS : admin 用户密码 ： keystone

`demo-openrc.sh`

 ```shell
[root@controller ~]# vim demo-openrc.sh

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
 ```

> DEMO_PASS : demo 用户密码：keystone

###### 使用脚本

source 脚本文件

```shell
. admin-openrc
```

获取 token

```shell
[root@controller ~]# openstack token issue
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-14T07:33:12+0000                                                                                                                                                                |
| id         | gAAAAABfNjAoME8Bfd9JjVIqgN8YtSkXKr3C5fIHLru2mb5fOqa5xAvV6a1gRcGEtkgnvYOdlgxo0f0eyhFSFDhHhGym8g5e-v66sjR4o7Sr8dN3zkSrR2pB9H0n0-VxvD4IQHmV-fEVxAyHOLqikw7rp7BdvGBFPSuuQMyAMVa_BDvvi7iFqcs |
| project_id | a10a48cffde949d9bd00f95955cd0c65                                                                                                                                                        |
| user_id    | c84cdffc89c74cd6819d3c02f1e44f36                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



#### 安装 glance 服务(controller node)

##### 建立数据库

```shell
[root@controller ~]# mysql -u root -p
```

```shell
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
```

> GLANCE_DBPASS : glance 数据库密码 ： glance

##### source 环境变量

```shell
. admin-openrc
```

##### 创建 glance 用户

 Create the `glance` user: 

```shell
[root@controller ~]# openstack user create --domain default --password-prompt glance
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 9f4a69a718554924b1a58d4131a12222 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> User Password: glance 用户密码 ： glance

 Add the `admin` role to the `glance` user and `service` project: 

```shell
[root@controller ~]# openstack role add --project service --user glance admin
```

 Create the `glance` service entity: 

```shell
[root@controller ~]# openstack service create --name glance --description "OpenStack Image" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 2f05d178f0f9420ba67583de4d1900de |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

##### 创建 API endpoint

```shell
[root@controller ~]# openstack endpoint create --region RegionOne image public http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9f792c74583f4a9caac33a74c811ae2e |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f05d178f0f9420ba67583de4d1900de |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne image internal http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 74296a124d3242ed9495e95911d435b2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f05d178f0f9420ba67583de4d1900de |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2fff0737474441538016036fe5c9730c |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2f05d178f0f9420ba67583de4d1900de |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

##### 安装配置 keystone

###### 安装 rpm 包

```shell
[root@controller ~]# yum install openstack-glance -y
```

###### 修改配置文件

`/etc/glance/glance-api.conf`

```shell
[root@controller ~]# vim /etc/glance/glance-api.conf

[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

> GLANCE_DBPASS : glance 数据库密码：glance
>
> GLANCE_PASS： glance 用户密码 ： glance



`/etc/glance/glance-registry.conf`

```shell
[root@controller ~]# vim /etc/glance/glance-registry.conf

[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
flavor = keystone
```

> GLANCE_DBPASS : glance 数据库密码: glance
>
> GLANCE_PASS ： glance 用户密码 ： glance

###### 生成数据表

```shell
[root@controller ~]# su -s /bin/sh -c "glance-manage db_sync" glance
```



##### 启动 glance 服务

```shell
[root@controller ~]# systemctl enable openstack-glance-api.service openstack-glance-registry.service
[root@controller ~]# systemctl start openstack-glance-api.service openstack-glance-registry.service
```

```shell
[root@controller ~]# ps -ef | grep glance
glance   23907     1  7 15:07 ?        00:00:02 /usr/bin/python2 /usr/bin/glance-api
glance   23908     1  5 15:07 ?        00:00:01 /usr/bin/python2 /usr/bin/glance-registry
glance   23925 23908  0 15:07 ?        00:00:00 /usr/bin/python2 /usr/bin/glance-registry
glance   23926 23908  0 15:07 ?        00:00:00 /usr/bin/python2 /usr/bin/glance-registry
glance   23927 23907  0 15:07 ?        00:00:00 /usr/bin/python2 /usr/bin/glance-api
glance   23928 23907  0 15:07 ?        00:00:00 /usr/bin/python2 /usr/bin/glance-api
root     23938 15638  0 15:07 pts/0    00:00:00 grep --color=auto glance
```



#### 安装 nova 服务(controller node)

##### 建立数据库

```shell
[root@controller ~]# mysql -u root -p
```

```shell
MariaDB [(none)]> CREATE DATABASE nova_api;

MariaDB [(none)]> CREATE DATABASE nova;

MariaDB [(none)]> CREATE DATABASE nova_cell0;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]>  GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
    ->   IDENTIFIED BY 'NOVA_DBPASS';
```

> NOVA_DBPASS ： nova 数据库密码 ： nova

##### source 环境变量

```shell
[root@controller ~]# source admin-openrc.sh
```

##### 创建 nova 用户

 Create the `nova` user

```shell
[root@controller ~]# openstack user create --domain default --password-prompt nova
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | dea0585c1de643c999cfee5fd5636d82 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> User Password: nova 用户密码 ： nova

 Add the `admin` role to the `nova` user

```shell
[root@controller ~]# openstack role add --project service --user nova admin
```

 Create the `nova` service entity

```shell
[root@controller ~]# openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 4f8255382b63424d892ac222d6a8cad6 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```

##### 创建 nova API endpoint 

```shell
[root@controller ~]# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 5c91943fdccd4ee4855af0a11d9f7bb4 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 4f8255382b63424d892ac222d6a8cad6 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6d6c511c362e45659c69a5c9a5338a33 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 4f8255382b63424d892ac222d6a8cad6 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | bd62d2d58310453f8a30ef5083d42791 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 4f8255382b63424d892ac222d6a8cad6 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

##### 配置  placement 

 Create a Placement service user

```shell
[root@controller ~]# openstack user create --domain default --password-prompt placement
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 5d6fc9aa3c984ee9b97750ad7766902c |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> User Password:  placement 用户密码 ： placement

 Add the Placement user to the service project with the admin role

```shell
[root@controller ~]# openstack role add --project service --user placement admin
```

 Create the Placement API entry in the service catalog

```shell
[root@controller ~]# openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | 00da19c485ae41618da43fb66a261022 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```

 Create the Placement API service endpoints

```shell
[root@controller ~]# openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | c08af54f33b542a4acee001409f4267b |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 00da19c485ae41618da43fb66a261022 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | f08ccc694dbf48cf9b27e0b5654569f1 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 00da19c485ae41618da43fb66a261022 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b59884456dea41bd82d897a8e44ae223 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 00da19c485ae41618da43fb66a261022 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

##### 安装配置 nova 服务

###### 安装 rpm 包

```shell
[root@controller ~]# yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api -y
```

###### 修改配置文件

`/etc/nova/nova.conf`

```shell
[root@controller ~]# vim /etc/nova/nova.conf

DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:RABBIT_PASS @controller
my_ip = 192.168.8.212
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS @controller/nova_api

[database]
connection = mysql+pymysql://nova:NOVA_DBPASS @controller/nova

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS 

[vnc]
enabled = true
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = PLACEMENT_PASS 
```

> NOVA_DBPASS : nova 数据库密码 ： nova
>
> RABBIT_PASS ： mq 密码 ： openstack
>
> NOVA_PASS : nova 用户密码 ： nova
>
> PLACEMENT_PASS ： placement 用户密码 ： placement

`/etc/httpd/conf.d/00-nova-placement-api.conf`

```shell
[root@controller ~]# vim /etc/httpd/conf.d/00-nova-placement-api.conf

<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```

###### 重启 httpd 服务

```shell
[root@controller ~]# systemctl restart httpd
```

###### 创建数据表

```shell
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
c27dc68b-c21f-4f8f-b179-2d5ec0a7ca74
[root@controller ~]# su -s /bin/sh -c "nova-manage db sync" nova
```

```shell
[root@controller ~]# nova-manage cell_v2 list_cells
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
|  Name |                 UUID                 |           Transport URL            |               Database Connection               |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |               none:/               | mysql+pymysql://nova:****@controller/nova_cell0 |
| cell1 | c27dc68b-c21f-4f8f-b179-2d5ec0a7ca74 | rabbit://openstack:****@controller |    mysql+pymysql://nova:****@controller/nova    |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
```

###### 启动 nova 服务

```shell
[root@controller ~]# systemctl enable openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service

[root@controller ~]# systemctl start openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service

```



#### 安装 nova 服务(compute node)

##### 安装 rpm 包

```shell
[root@compute ~]# yum install openstack-nova-compute -y
```

##### 修改配置文件

`/etc/nova/nova.conf`

```shell
[root@compute ~]# vim /etc/nova/nova.conf

[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:RABBIT_PASS@controller
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver


[api]
auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS

[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = PLACEMENT_PASS
```

> RABBIT_PASS : mq 密码 ： openstack
>
> NOVA_PASS ： nova 用户密码 ： nova
>
> MANAGEMENT_INTERFACE_IP_ADDRESS : 计算节点管理ip 
>
> PLACEMENT_PASS ： placement 用户密码 ： placement 

##### 检验compute node

```shell
[root@compute ~]# egrep -c '(vmx|svm)' /proc/cpuinfo
4
```

> 返回值大于等于1，说明主机支持硬件加速，如果返回值等于 0，修改配置文件：
>
> ```shell
> [root@compute ~]# vim /etc/nova/nova.conf
> 
> [libvirt]
> virt_type = qemu
> ```

##### 启动 nova 服务

```shell
[root@compute ~]# systemctl enable libvirtd.service openstack-nova-compute.service
[root@compute ~]# systemctl start libvirtd.service openstack-nova-compute.service
```

##### 添加 compute node to cell database

> 在 **controller** 节点执行下列命令

confirm there are compute hosts in the database

```shell
[root@controller ~]# source admin-openrc.sh
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
|  7 | nova-compute | compute | nova | enabled | up    | 2020-08-14T08:21:22.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+
```

 Discover compute hosts

```shell
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': c27dc68b-c21f-4f8f-b179-2d5ec0a7ca74
Checking host mapping for compute host 'compute': afb1f08e-c8a6-4531-aa34-e8383917aa18
Creating host mapping for compute host 'compute': afb1f08e-c8a6-4531-aa34-e8383917aa18
Found 1 unmapped computes in cell: c27dc68b-c21f-4f8f-b179-2d5ec0a7ca74
```

> 添加新的计算节点时，必须在控制器节点上运行 `nova-manage cell_v2 discover_hosts`才能注册这些新的计算节点。另外，可以在`/etc/nova/nova.conf`中设置适当的间隔：
>
> ```shell
> [scheduler]
> discover_hosts_in_cells_interval = 300
> ```



#### 安装 neutron 服务(controller node)

##### 建立数据库

```shell
[root@controller ~]#  mysql -u root -p
```

```shell
MariaDB [(none)]> CREATE DATABASE neutron;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
    ->   IDENTIFIED BY 'NEUTRON_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
    ->   IDENTIFIED BY 'NEUTRON_DBPASS';
```

> NEUTRON_DBPASS : neutron 数据库密码 ： neutron

##### source 环境变量

```shell
[root@controller ~]# source admin-openrc.sh
```

##### 创建 neutron 用户

 Create the `neutron` user

```shell
[root@controller ~]# openstack user create --domain default --password-prompt neutron
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 80b6813af19f4dce9e1ab7eabe1ddf4f |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> User Password: neutron 用户密码：neutron

 Add the `admin` role to the `neutron` user

```shell
[root@controller ~]# openstack role add --project service --user neutron admin
```

 Create the `neutron` service entity

```shell
[root@controller ~]# openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 68831af9c30041968efbfe790695f7f7 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

##### 创建 neutron API endpoint

```shell
[root@controller ~]# openstack endpoint create --region RegionOne network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | ec4d80c7a60d4dd3bf641011a0984ae7 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 68831af9c30041968efbfe790695f7f7 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9ff2016f038a4b3390b0fc4dda0a43da |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 68831af9c30041968efbfe790695f7f7 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 87f127eb06554622ad1fad24bcc0d644 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 68831af9c30041968efbfe790695f7f7 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

##### 配置网络选项

> 两种方式选一种

###### 安装 rpm 包

```shell
[root@controller ~]# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch ebtables -y
```

> 直接使用 ovs agent，不适用 linux bridge agent
>
> 启动 ovs 服务：
>
> ```shell
> [root@controller ~]# systemctl enable openvswitch
> [root@controller ~]# systemctl start openvswitch
> [root@controller ~]# systemctl status openvswitch
> ● openvswitch.service - Open vSwitch
>    Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; enabled; vendor preset: disabled)
>    Active: active (exited) since Sat 2020-08-15 10:28:58 CST; 12s ago
>   Process: 19089 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
>  Main PID: 19089 (code=exited, status=0/SUCCESS)
> 
> Aug 15 10:28:58 controller systemd[1]: Starting Open vSwitch...
> Aug 15 10:28:58 controller systemd[1]: Started Open vSwitch.
> 
> [root@controller ~]# ovs-vsctl show
> 1f74c335-2a35-4b8b-a08b-e06c698fb0a3
>     ovs_version: "2.9.0"
> ```

###### 修改配置文件

`/etc/neutron/neutron.conf`

```shell
[root@controller ~]# vim /etc/neutron/neutron.conf

[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:RABBIT_PASS@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[database]
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS

[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

> NEUTRON_DBPASS : neutron 数据库密码：neutron
>
> RABBIT_PASS ： mq 密码 ： openstack
>
> NEUTRON_PASS : neutron 用户密码 ：neutron
>
> NOVA_PASS ： nova 用户密码：nova

`/etc/neutron/plugins/ml2/ml2_conf.ini`

```shell
[root@controller ~]# vim /etc/neutron/plugins/ml2/ml2_conf.ini

[DEFAULT]

[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan,vlan,flat
# mechanism_drivers = linuxbridge,l2population
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security

[ml2_type_flat]
# flat_networks = provider

[ml2_type_vlan]
network_vlan_ranges = external:2259:2260,service:2246:2258,management

[ml2_type_vxlan]
vni_ranges = 10001:65535

[securitygroup]
enable_ipset = true
```

创建 ovs 网桥

```shell
[root@controller ~]# ovs-vsctl add-br br-ex  # external 网络，走 br-ex 网桥
[root@controller ~]# ovs-vsctl add-br br-service # service 网络，走 br-service 网桥
[root@controller ~]# ovs-vsctl add-br br-mgnt # management 网络，走 br-mgnt 网桥
```

> ```
> ovs-vsctl add-port br-eth2 eth2 # 可以使用这个命令为port添加port
> ```

`/etc/neutron/plugins/ml2/openvswitch_agent.ini`

```shell
[root@controller ~]# vim /etc/neutron/plugins/ml2/openvswitch_agent.ini

[DEFAULT]

[ovs]
bridge_mappings = external:br-ex,service:br-service,management:br-mgnt
local_ip = 192.168.8.212
ovsdb_interface = native

[agent]
tunnel_types = vxlan
l2_population = True
tunnel_csum = True

[securitygroup]
enable_security_group = True
```

`/etc/neutron/l3_agent.ini`

```shell
[root@controller ~]# vim /etc/neutron/l3_agent.ini

[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
```

`/etc/neutron/dhcp_agent.ini`

```shell
[root@controller ~]# vim /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
```

`/etc/neutron/metadata_agent.ini`

```shell
[root@controller ~]# vim /etc/neutron/metadata_agent.ini
[DEFAULT]

nova_metadata_host = controller
metadata_proxy_shared_secret = metadata
```

> METADATA_SECRET : metadata 密码 ： metadata

##### 修改 nova 配置文件

`/etc/nova/nova.conf`

```shell
[root@controller ~]# vim /etc/nova/nova.conf
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
service_metadata_proxy = true
metadata_proxy_shared_secret = METADATA_SECRET
```

> NEUTRON_PASS : neutron 密码 ： neutron
>
> METADATA_SECRET ： metadata 密码 ： metadata

##### 创建软链接

```shell
[root@controller ~]# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

##### 创建数据表

```shell
[root@controller ~]# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

##### 重启 nova-api 

```shell
[root@controller ~]# systemctl restart openstack-nova-api.service
```

##### 启动 neutron 相关服务

```shell
[root@controller ~]# systemctl enable neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service

[root@controller ~]# systemctl start neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service

[root@controller ~]# systemctl enable neutron-l3-agent.service

[root@controller ~]# systemctl start neutron-l3-agent.service
```



#### 安装 neutron 服务(compute node)

##### 安装 rpm 包

```shell
[root@compute ~]# yum install openstack-neutron-openvswitch ebtables ipset -y 
```

##### 修改配置文件

`/etc/neutron/neutron.conf`

```shell
[root@compute ~]# vim /etc/neutron/neutron.conf

[DEFAULT]
transport_url = rabbit://openstack:RABBIT_PASS@controller
auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS 

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

> NEUTRON_PASS : neutron 密码：neutron
>
> RABBIT_PASS ： mq 密码 ： openstack

##### 配置网络选项

创建 ovs 网桥

```shell
[root@controller ~]# ovs-vsctl add-br br-ex  # external 网络，走 br-ex 网桥
[root@controller ~]# ovs-vsctl add-br br-service # service 网络，走 br-service 网桥
[root@controller ~]# ovs-vsctl add-br br-mgnt # management 网络，走 br-mgnt 网桥
```

> ```
> ovs-vsctl add-port br-eth2 eth2 # 可以使用这个命令为port添加port
> ```

`/etc/neutron/plugins/ml2/openvswitch_agent.ini`

```shell
[root@compute ~]# vim /etc/neutron/plugins/ml2/openvswitch_agent.ini

[DEFAULT]

[ovs]
bridge_mappings = external:br-ex,service:br-service,management:br-mgnt
local_ip = 192.168.8.213

[agent]
tunnel_types = vxlan
l2_population = True
tunnel_csum = True

[securitygroup]
enable_security_group = True
```

##### 修改 nova 配置文件

`/etc/nova/nova.conf`

```shell
[root@compute ~]# vim /etc/nova/nova.conf

[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
```

> NEUTRON_PASS : neutron 密码 ： neutron

##### 重启 nova-compute 服务

```shell
[root@compute ~]# systemctl restart openstack-nova-compute.service
```

##### 启动 ovs-agent

```shell
[root@compute ~]# systemctl enable neutron-openvswitch-agent.service
[root@compute ~]# systemctl start neutron-openvswitch-agent.service
```



[Reference1>>]( https://docs.openstack.org/install-guide/openstack-services.html#minimal-deployment-for-pike )

---------

