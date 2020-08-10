---
layout: post
title: 原生 OpenStack 环境搭建
categories: [OpenStack]
description: opensatck 环境搭建
keywords: OpenStack
---

# 原生 OpenStack 环境搭建

> openstack 版本： train



### 环境准备

| 节点       | 系统     | ip            | 描述     |
| ---------- | -------- | ------------- | -------- |
| compute    | CentOS 7 | 192.168.8.210 | 计算节点 |
| controller | CnetOS 7 | 192.168.8.211 | 控制节点 |



**1. 关闭防火墙 + 关闭 selinux:**

```shell
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```



**2. 配置节点网络信息：**

> centos ： 
>
> ```shell
> # cd /etc/sysconfig/network-scripts/
> # vim ifcfg-ens160
> 
> BOOTPROTO="static" # dhcp改为static 
> ONBOOT="yes" # 开机启用本配置
> IPADDR=192.168.8.210 # 静态IP
> GATEWAY=192.168.8.254  # 默认网关
> NETMASK=255.255.255.0 # 子网掩码
> DNS1=114.114.114.114  #DNS 配置
> ```
>
> 重启网络服务
>
> ```shell
> # service network restart
> ```



**3. 修改节点主机名称解析：**

```shell
# hostnamectl set-hostname controller  	 # controller节点
# hostnamectl set-hostname compute       # compute 节点
```

​    修改 `/etc/hosts` 文件，添加 (修改完重启)

```python
192.168.8.210 compute
192.168.8.211 controller
```



**4. NTP 同步系统时间:**

```shell
# 1. 虚拟机可以上网
# yum install chrony  # chrony 是 ntp 的具体实现,默认和公共服务器同步时间
# systemctl enable chronyd.service 
# systemctl start chronyd.service
# chronyc sources # 验证
```



### OpenStack 软件包

#### 启用OpenStack存储库

> centos 配置可用的 yum 源：（需要确定源是否有相应版本）

下载安装源码包：

```shell
yum install centos-release-openstack-train # 所有节点都需要安装
```

升级软件包：

```shell
yum upgrade
```

安装 openstack 客户端：

```shell
yum install python-openstackclient -y
```

RHEL和CentOS 默认情况下启用SELinux,安装 `openstack-selinux`软件包以自动管理OpenStack服务的安全策略： 

```shell
yum install openstack-selinux
```



### SQL 数据库

#### 安装和配置

安装软件包：

```shell
 yum install mariadb mariadb-server python2-PyMySQL -y # 只在控制节点安装
```

配置 SQL 信息：

```shell
# 创建编辑 /etc/my.cnf.d/openstack.cnf

[mysqld]
bind-address = 192.168.8.211  # 控制节点的管理 ip 

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

启动服务：

```shell
# systemctl enable mariadb.service
# systemctl start mariadb.service
# mysql_secure_installation  # 设置密码一定要仔细看提示 否则密码设置错了 修改很麻烦
```

> 数据库密码 ： root



### Message Queue

rabbitMQ , Controller node

安装软件包：

```shell
# yum install rabbitmq-server -y
```

启动服务：

```shell
# systemctl enable rabbitmq-server.service
# systemctl start rabbitmq-server.service
```

添加openstack用户:

```shell
# rabbitmqctl add_user openstack openstack123 # openstack123 is password
```

> password is : openstack123

允许openstack用户进行配置，写入和读取访问：

```shell
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```



### Memcached 

Controller node

服务的身份服务身份验证机制使用Memcached来缓存令牌， memcached服务通常在控制器节点上运行

安装软件包：

```shell
# yum install memcached python-memcached -y
```

配置   `/etc/sysconfig/memcached` 配置服务以使用控制器节点的管理IP地址，这是为了允许其他节点通过管理网络进行访问：

```shell
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1,controller"
```

启动服务：

```shell
# systemctl enable memcached.service
# systemctl start memcached.service
```



### Etcd

Optional , Controller node

OpenStack服务可以使用Etcd（一种分布式可靠的键值存储）来进行分布式键锁定，存储配置，跟踪服务活动性和其他情况

安装软件包：

```shell
# yum install etcd -y 
```

修改配置文件 `/etc/etcd/etcd.conf` 将`ETCD_INITIAL_CLUSTER`，`ETCD_INITIAL_ADVERTISE_PEER_URLS`，`ETCD_ADVERTISE_CLIENT_URLS`，`ETCD_LISTEN_CLIENT_URLS` 设置为控制器节点的管理IP地址，以允许其他节点通过管理网络进行访问

```shell
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
ETCD_LISTEN_PEER_URLS="http://192.168.8.211:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.8.211:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="controller"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.8.211:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.8.211:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
ETCD_INITIAL_CLUSTER="controller=http://192.168.8.211:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
```

启动服务：

```shell
# systemctl enable etcd
# systemctl start etcd
```



### 安装 Keystone 服务

keystone 只需要在 controller 节点安装

**1. 创建 keystone 数据库**

```shell
# mysql -u root -p
# 创建数据库
MariaDB [(none)]> CREATE DATABASE keystone;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    -> IDENTIFIED BY 'keystone123'; 

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    -> IDENTIFIED BY 'keystone123'; # 授予 keystone 适当的权限
```

**2. 安装 rpm 包：**

```shell
# yum install openstack-keystone httpd mod_wsgi -y
```

**3. 修改配置：**

 ` /etc/keystone/keystone.conf ` 

database :

```shell
[database]

connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
```

>  KEYSTONE_DBPASS ： keystone 数据库密码 ： keystone123

token:

```shell
[token]
provider = fernet
```

**4. 加载 keystone 数据库 schema** 

```shell
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

**5. 初始化密钥存储库：**

```shell
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

**6. 启动 keystone 服务：**

```shell
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

> ADMIN_PASS 换成具体的密码：keystone123

**7. 配置 Apache http 服务：**

配置 ` /etc/httpd/conf/httpd.conf` 

```shell
ServerName controller
```

> ServerName : 控制节点 name ，如果没有配置需要添加 

**8. 创建 `/usr/share/keystone/wsgi-keystone.conf` 的软连接**

```shell
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

**9. 启动 http 服务：**

```shell
systemctl enable httpd.service
systemctl start httpd.service
```

**10 设置环境变量来管理配置账户：**

```shell
$ export OS_USERNAME=admin
$ export OS_PASSWORD=ADMIN_PASS
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:5000/v3
$ export OS_IDENTITY_API_VERSION=3
```

> ADMIN_PASS : 是第 6 步设置的密码： keystone123

**11. 创建 domain, projects, users, and roles**

domain:

```shell
[root@controller ~]# openstack domain create --description "An Example Domain" example
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | An Example Domain                |
| enabled     | True                             |
| id          | 386718dc96c84980aca258e95129b4ee |
| name        | example                          |
| options     | {}                               |
| tags        | []                               |
+-------------+----------------------------------+

```

 service：

```shell
[root@controller ~]# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 12e8d05346a6487b9384054cc31fad0e |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

```

project:

```shell
[root@controller ~]# openstack project create --domain default --description "Demo Project" myproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 10482ae4df55480493d1c323ea749124 |
| is_domain   | False                            |
| name        | myproject                        |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

user:

```shell
[root@controller ~]# openstack user create --domain default --password-prompt myuser
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 48c7f68de8e6444f880bd21d3459956b |
| name                | myuser                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> Password: default

role:

```shell
[root@controller ~]# openstack role create myrole
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | d9538368e84341b2acd5b22f85eae6e1 |
| name        | myrole                           |
| options     | {}                               |
+-------------+----------------------------------+
```

 add the `myrole` role to the `myproject` project and `myuser` user: 

```shell
[root@controller ~]# openstack role add --project myproject --user myuser myrole
```

**12验证 keystone 服务**

取消设置临时 `OS_AUTH_URL` 和 `OS_PASSWORD` 环境变量:

```shell
[root@controller ~]# unset OS_AUTH_URL OS_PASSWORD
```

获取 token ：

以管理员身份获取 token：

```shell
[root@controller ~]# openstack --os-auth-url http://controller:5000/v3 \
>   --os-project-domain-name Default --os-user-domain-name Default \
>   --os-project-name admin --os-username admin token issue
Password: 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-06T12:48:25+0000                                                                                                                                                                |
| id         | gAAAAABfK-4JaMZV_UoD7LrRcis_h7oK2RVlpnu_xKODZOsjosC8zrFdQu9sRZgV2fsJJXFUa-VRP6Hed47KBQMGt0Px4lzzSDWXp12D8wEoLA3kIy1InEltyZuUEzmIsv8axbetQJin0-RBqnRpJG0-ppkoMlYRIIAYsZKXzt6OJ0z1uDMO0Vg |
| project_id | e487f0b6dd5f49e4abf69649c9130560                                                                                                                                                        |
| user_id    | d6aa5398f9984fe6ab59fb6bcfae4b86                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

> Password : keystone123

以普通用户身份获取 token：

```shell
[root@controller ~]# openstack --os-auth-url http://controller:5000/v3   --os-project-domain-name Default --os-user-domain-name Default   --os-project-name myproject --os-username myuser token issue
Password: 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-06T12:51:54+0000                                                                                                                                                                |
| id         | gAAAAABfK-7aP_crOxnsuOkYc_LJJ6f-TBi-9KQL7zpPIo7avW4M1Li8UNUREWuMI9r53NWqFGxhVz1HTGMsu2VTPFFLKbPDPjueB9y1iZ6Yv47XQujUvOMAwdgB7AbYsTn_GtBEPFolUZKhc_kgQnYK_nk5su3iGOnOy2FHosnELi6tODBzglk |
| project_id | 10482ae4df55480493d1c323ea749124                                                                                                                                                        |
| user_id    | 48c7f68de8e6444f880bd21d3459956b                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

> Password : 用户密码 default

**13. 设置 source 文件**

 admin-openrc.sh

```shell
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

> ADMIN_PASS : keystone123

demo-openrc.sh

```shell
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=myproject
export OS_USERNAME=myuser
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

> DEMO_PASS : 用户密码 default

**14. 使用 scripts ，获取 token**

```shell
[root@controller ~]# source admin-openrc.sh
[root@controller ~]# openstack token issue
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2020-08-06T12:58:59+0000                                                                                                                                                                |
| id         | gAAAAABfK_CDbctjt2_iGUaMbilS2Qv-tBk_O24f1CFK_I4UAUZrPsUCp1TN1n1mxevbXB-B_0dfzTnj8HE8ipxedtEgMcA8ZCFwhD-abJaeZhC90mMpst06yAvK6x-hMtIEQACH0UqwqJlfKVPFZjUSyQTI5q7szixoETpXbWh0zQGCwgwicXA |
| project_id | e487f0b6dd5f49e4abf69649c9130560                                                                                                                                                        |
| user_id    | d6aa5398f9984fe6ab59fb6bcfae4b86                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



### 安装 placement

**1. 配置数据库：**

```shell
[root@controller ~]# mysql -u root -p

MariaDB [(none)]> CREATE DATABASE placement;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \
  IDENTIFIED BY 'PLACEMENT_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \
  IDENTIFIED BY 'PLACEMENT_DBPASS';
```

> PLACEMENT_DBPASS :  placement 密码 ： placement123

**2. 执行配置文件脚本：**

```shell
[root@controller ~]# source admin-openrc.sh
```

**3. 创建 user 并配置 keystone endpoint：**

 Create a Placement service user using your chosen `PLACEMENT_PASS`: 

```shell
[root@controller ~]# openstack user create --domain default --password-prompt placement
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 5252b71de09443278663de6963947448 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

```

> Password :  placement123

 Add the Placement user to the service project with the admin role: 

```shell
[root@controller ~]# openstack role add --project service --user placement admin
```

 Create the Placement API entry in the service catalog: 

```shell
[root@controller ~]# openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | b5e1f340c6f045f891fd6d6fa99af02c |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+

```

 Create the Placement API service endpoints: 

```shell
[root@controller ~]# openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0bcb727a09864f1ead73eae0d12a37a3 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b5e1f340c6f045f891fd6d6fa99af02c |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | eeda64ba59204580b14c82dde58e0d95 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b5e1f340c6f045f891fd6d6fa99af02c |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8ccbe93e9485463d912897069cb51d8a |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b5e1f340c6f045f891fd6d6fa99af02c |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

```

**4. 安装 rpm 包：**

```shell
[root@controller ~]# yum install openstack-placement-api -y
```

**5. 修改配置文件**

` /etc/placement/placement.conf ` :

 In the `[placement_database]` section, configure database access: 

```shell
[placement_database]
# ...
connection = mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement
```

> PLACEMENT_DBPASS : placement 数据库 密码

 In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access: 

```shell
[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = PLACEMENT_PASS
```

> PLACEMENT_PASS : placement 服务密码

**6. 创建数据库(表):**

```shell
[root@controller ~]# su -s /bin/sh -c "placement-manage db sync" placement
```

**7. 重启 httpd 服务：**

```shell
[root@controller ~]# systemctl restart httpd
```



### 安装 nova 服务

#### 控制节点

**1. 配置数据库 ：**

```shell
[root@controller ~]# mysql -u root -p

MariaDB [(none)]> CREATE DATABASE nova_api; # 创建 nova api 数据库

MariaDB [(none)]> CREATE DATABASE nova;  # 创建 nova 数据库

MariaDB [(none)]> CREATE DATABASE nova_cell0;  # 创建 nova_cell0 数据库

# 设置访问权限

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';

```

> NOVA_DBPASS : nova 数据库密码 ： nova123

**2. 执行 source 脚本：**

```shell
source admin-openrc.sh
```

**3. 创建 nova 用户信息：**

 Create the `nova` user: 

```shell
[root@controller ~]# openstack user create --domain default --password-prompt nova
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 40487b23b64b478aaf12f040e11952a4 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> Password : nova123

 Add the `admin` role to the `nova` user: 

```shell
[root@controller ~]# openstack role add --project service --user nova admin
```

 Create the `nova` service entity: 

```shell
[root@controller ~]# openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 53fbf33282c040f1846fad6a46e61c3e |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```

**4. 在 keystone 创建 endpoint ：**

```shell
[root@controller ~]# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6089e9ea84ee423486ecda3f0a6a634e |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53fbf33282c040f1846fad6a46e61c3e |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2fb1db0f89894aa596617f701ba185e7 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53fbf33282c040f1846fad6a46e61c3e |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e2efe7284c6b48fd9981bd8b33885035 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 53fbf33282c040f1846fad6a46e61c3e |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

```

**5. 安装 rpm 包：**

```shell
[root@controller ~]# yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler 
```

**6. 修改配置文件：**

 ` /etc/nova/nova.conf `

 In the `[DEFAULT]` section, enable only the compute and metadata APIs: 

```shell
[DEFAULT]
enabled_apis = osapi_compute,metadata
```

 In the `[api_database]` and `[database]` sections, configure database access: 

```shell
[api_database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api

[database]
# ...
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
```

> NOVA_DBPASS : nova 数据库密码 : nova123

 In the `[DEFAULT]` section, configure `RabbitMQ` message queue access: 

```shell
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
```

> RABBIT_PASS : mq 的密码 ： openstack123 

 In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access: 

```shell
[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = NOVA_PASS
```

> NOVA_PASS : nova 服务的密码 ： nova123 

In the `[DEFAULT]` section, configure the `my_ip` option to use the management interface IP address of the controller node: 

```shell
[DEFAULT]
# ...
my_ip = 192.168.8.211
```

 In the `[DEFAULT]` section, enable support for the Networking service: 

```shell
[DEFAULT]
# ...
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```

**7. 创建数据库（表）：**

```shell
# Populate the nova-api database
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova 
```

```shell
# Register the cell0 database
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

```shell
# Create the cell1 cell:
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
f3960e56-3c9b-4482-996e-022704456a69
```

```shell
# Populate the nova database:
[root@controller ~]# su -s /bin/sh -c "nova-manage db sync" nova
```

```shell
# Verify nova cell0 and cell1 are registered correctly:
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+----------+
|  Name |                 UUID                 |           Transport URL            |               Database Connection               | Disabled |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |               none:/               | mysql+pymysql://nova:****@controller/nova_cell0 |  False   |
| cell1 | f3960e56-3c9b-4482-996e-022704456a69 | rabbit://openstack:****@controller |    mysql+pymysql://nova:****@controller/nova    |  False   |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+----------+
```

**8. 验证并启动相关服务：**

```shell
# systemctl enable openstack-nova-api.service openstack-nova-consoleauth openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service 
  
# systemctl start openstack-nova-api.service openstack-nova-consoleauth openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

```shell
[root@controller ~]# ps -elf | grep nova
4 S nova      2955     1  0  80   0 - 107168 ep_pol 10:52 ?       00:00:05 /usr/bin/python2 /usr/bin/nova-novncproxy --web /usr/share/novnc/
4 S nova     13155     1  5  80   0 - 98598 ep_pol 14:06 ?        00:00:02 /usr/bin/python2 /usr/bin/nova-scheduler
4 S nova     13168     1  5  80   0 - 98942 ep_pol 14:06 ?        00:00:02 /usr/bin/python2 /usr/bin/nova-conductor
0 R root     13329 22004  0  80   0 - 28203 -      14:07 pts/0    00:00:00 grep --color=auto nova
4 S nova     32633     1  0  80   0 - 111575 poll_s 10:45 ?       00:01:36 /usr/bin/python2 /usr/bin/nova-api
1 S nova     32655 32633  0  80   0 - 113365 ep_pol 10:45 ?       00:00:00 /usr/bin/python2 /usr/bin/nova-api
1 S nova     32656 32633  0  80   0 - 113460 ep_pol 10:45 ?       00:00:00 /usr/bin/python2 /usr/bin/nova-api

```



#### 计算节点

**1. 安装软件包 ：** 

```shell
[root@compute ~]# yum install openstack-nova-compute -y 
```

**2. 修改配置文件**

 ` /etc/nova/nova.conf` :

 In the `[DEFAULT]` section, enable only the compute and metadata APIs: 

```shell
[DEFAULT]
enabled_apis = osapi_compute,metadata
```

 In the `[DEFAULT]` section, configure `RabbitMQ` message queue access: 

```shell
[DEFAULT]
transport_url = rabbit://openstack:RABBIT_PASS@controller
```

> RABBIT_PASS : 消息队列 mq 密码 ： openstack123

 In the `[api]` and `[keystone_authtoken]` sections, configure Identity service access: 

```shell


[keystone_authtoken]
www_authenticate_uri = http://controller:5000/
auth_url = http://controller:5000/
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = NOVA_PASS
```

> NOVA_PASS : nova 服务密码： nova123

 In the `[DEFAULT]` section, configure the `my_ip` option: 

```shell
[DEFAULT]
# ...
my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
```

> MANAGEMENT_INTERFACE_IP_ADDRESS : 计算节点管理网 ip ：192.168.8.210 

 In the `[DEFAULT]` section, enable support for the Networking service: 

```shell
[DEFAULT]
# ...
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
```

 Configure the `[neutron]` section of **/etc/nova/nova.conf**，安装了 neutron 服务之后在配置

 In the `[vnc]` section, enable and configure remote console access: 

```shell
[vnc]
# ...
enabled = true
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
```

 In the `[glance]` section, configure the location of the Image service API: 

```shell
[glance]
# ...
api_servers = http://controller:9292
```

 In the `[oslo_concurrency]` section, configure the lock path: 

```shell
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```

 In the `[placement]` section, configure the Placement API: 

```shell
[placement]
# ...
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = PLACEMENT_PASS
```

> PLACEMENT_PASS : placement 密码 ： placement123

**3. 确定计算节点是否支持虚拟机的硬件加速：**

```shell
[root@compute ~]# egrep -c '(vmx|svm)' /proc/cpuinfo
1
```

> 返回的值等于或大于1，则说明您的计算节点支持硬件加速，通常不需要其他配置，如果返回值等于0，说明您的计算节点不支持硬件加速，并且必须将libvirt配置为使用QEMU而不是KVM
>
> 编辑 ` /etc/nova/nova.conf`
>
> ```shell
> [libvirt]
> # ...
> virt_type = qemu
> ```

**4. 启动服务：**

```shell
[root@compute ~]# systemctl enable libvirtd.service openstack-nova-compute.service
[root@compute ~]# systemctl start libvirtd.service openstack-nova-compute.service
```

**5. Add the compute node to the cell database**

**以下命令在 controller 节点执行**

 Source the admin credentials to enable admin-only CLI commands, then confirm there are compute hosts in the database: 

```shell
[root@controller ~]# source admin-openrc.sh
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
|  5 | nova-compute | compute | nova | enabled | up    | 2020-08-07T06:44:43.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+
```

 Discover compute hosts: 

```shell
[root@controller ~]# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': f3960e56-3c9b-4482-996e-022704456a69
Checking host mapping for compute host 'compute': 00cf38d2-2973-4aec-baab-b4beb54a3422
Creating host mapping for compute host 'compute': 00cf38d2-2973-4aec-baab-b4beb54a3422
Found 1 unmapped computes in cell: f3960e56-3c9b-4482-996e-022704456a69
```

> 添加新的计算节点时，必须在控制器节点上运行`nova-manage cell_v2 discover_hosts`才能注册这些新的计算节点。 另外，您可以在 `/etc/nova/nova.conf` 中设置适当的间隔：
>
> ```shell
> [scheduler]
> discover_hosts_in_cells_interval = 300
> ```



### 安装 neutron 服务

#### 控制节点

**1. 配置 mysql 数据库**

```shell
[root@controller ~]# mysql -u root -p

MariaDB [(none)]> CREATE DATABASE neutron;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
  
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
```

> NEUTRON_DBPASS : neutron 数据库密码 ： neutron123 

**2. 在 keystone 创建 neutron 角色**

```shell
[root@controller ~]# source admin-openrc.sh
```

 Create the `neutron` user: 

```shell
[root@controller ~]# openstack user create --domain default --password-prompt neutron
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | bbfe591a8f314ab4a06c128849bd280e |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

> Password : neutron123 

Add the admin role to the neutron user:

```shell
[root@controller ~]# openstack role add --project service --user neutron admin
```

 Create the `neutron` service entity: 

```shell
[root@controller ~]# openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 5d25e596cc2e44a786d1e99eff63f434 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

**3. 在 keystone 创建**   Networking service API endpoints 

```shell
[root@controller ~]# openstack endpoint create --region RegionOne network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | d915f1dbfcc74e9480b99040171e0d1f |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5d25e596cc2e44a786d1e99eff63f434 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | c26c1f4751824deeb23a5531e983db00 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5d25e596cc2e44a786d1e99eff63f434 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1623fa7b681143c2a257a91fea964c8d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 5d25e596cc2e44a786d1e99eff63f434 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

```

**4. 配置网络选项：**（两种方式任选其一）

安装软件包：

```shell
[root@controller ~]# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y 
```

修改配置文件` /etc/neutron/neutron.conf`：

 In the `[database]` section, configure database access: 

```shell
[database]
# ...
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
```

> NEUTRON_DBPASS ： neutron 数据库密码 : neutron123

 In the `[DEFAULT]` section, enable the Modular Layer 2 (ML2) plug-in, router service, and overlapping IP addresses: 

```shell
[DEFAULT]
# ...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
```

 In the `[DEFAULT]` section, configure `RabbitMQ` message queue access: 

```shell
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
```

> RABBIT_PASS : mq 密码 ： openstack123

 In the `[DEFAULT]` and `[keystone_authtoken]` sections, configure Identity service access: 

```shell
[DEFAULT]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
```

>  NEUTRON_PASS : neutron 服务密码 ： neutron123

 In the `[DEFAULT]` and `[nova]` sections, configure Networking to notify Compute of network topology changes: 

```shell
[DEFAULT]
# ...
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[nova]
# ...
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS
```

> NOVA_PASS : nova 密码 ： nova123

 In the `[oslo_concurrency]` section, configure the lock path: 

```shell
[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp
```

配置 ml2 Plugin：`/etc/neutron/plugins/ml2/ml2_conf.ini`

 In the `[ml2]` section, enable flat, VLAN, and VXLAN networks: 

```shell
[ml2]
# ...
type_drivers = flat,vlan,vxlan
```

 In the `[ml2]` section, enable VXLAN self-service networks: 

```shell
[ml2]
# ...
tenant_network_types = vxlan
```

 In the `[ml2]` section, enable the Linux bridge and layer-2 population mechanisms: 

```shell
[ml2]
# ...
mechanism_drivers = linuxbridge,l2population
```

 In the `[ml2]` section, enable the port security extension driver: 

```shell
[ml2]
# ...
extension_drivers = port_security
```

 In the `[ml2_type_flat]` section, configure the provider virtual network as a flat network: 

```shell
[ml2_type_flat]
# ...
flat_networks = provider
```

 In the `[ml2_type_vxlan]` section, configure the VXLAN network identifier range for self-service networks: 

```shell
[ml2_type_vxlan]
# ...
vni_ranges = 1:1000
```

 In the `[securitygroup]` section, enable ipset to increase efficiency of security group rules: 

```shell
[securitygroup]
# ...
enable_ipset = true
```

配置结果：

```shell
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population

[ml2_type_flat]
flat_networks = provider
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true

```

配置 linux bridge agent：` /etc/neutron/plugins/ml2/linuxbridge_agent.ini` 

 In the `[linux_bridge]` section, map the provider virtual network to the provider physical network interface: 

```shell
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
```

> PROVIDER_INTERFACE_NAME : 配置 linux-bridge 对应哪一个 物理网口  : ens160

 In the `[vxlan]` section, enable VXLAN overlay networks, configure the IP address of the physical network interface that handles overlay networks, and enable layer-2 population: 

```shell
[vxlan]
enable_vxlan = true
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = true
```

> OVERLAY_INTERFACE_IP_ADDRESS : vxlan 的 local ip : 192.168.8.211

 In the `[securitygroup]` section, enable security groups and configure the Linux bridge iptables firewall driver: 

```shell
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

确认 linux bridge 功能 ：

```shell
net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-ip6tables
```

> 需要了解

配置 L3 agent : ` /etc/neutron/l3_agent.ini `

```shell
[DEFAULT]
interface_driver = linuxbridge
```

配置 dhcp agent : ` /etc/neutron/dhcp_agent.ini `

```shell
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

**5.配置 metadata agent：**

` /etc/neutron/metadata_agent.ini `

```shell
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = METADATA_SECRET
```

> METADATA_SECRET : metadata  密码：  metadata 

**6. 配置 nova 服务使用 networking 服务：**

` /etc/nova/nova.conf `

In the `[neutron]` section, configure access parameters, enable the metadata proxy, and configure the secret: 

```shell
[neutron]
url = http://controller:9696
auth_url = http://controller:5000
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

>  NEUTRON_PASS : neutron 服务密钥： neutron123 ，METADATA_SECRET：metadata 密钥 ： metadata

**7. 建立软链接**

 `  /etc/neutron/plugins/ml2/ml2_conf.ini ` 到  `/etc/neutron/plugin.ini` 的软连接：

```shell
[root@controller ~]# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

**8. 执行脚本建立数据表：**

```shell
[root@controller ~]# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

**9. 重启 nova api 服务：**

```shell
[root@controller ~]# systemctl restart openstack-nova-api.service
```

**10. 启动 neutron 相关服务：**

```shell
[root@controller ~]# systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service

[root@controller ~]# systemctl start neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

L3 服务需要启动 L3 Agent ：

```shell
[root@controller ~]# systemctl enable neutron-l3-agent.service
[root@controller ~]# systemctl start neutron-l3-agent.service
```



#### 计算节点

**1. 安装 linux-bridge 软件包：**

```shell
[root@compute ~]# yum install openstack-neutron-linuxbridge ebtables ipset -y
```

**2. 修改配置文件**

 `/etc/neutron/neutron.conf`：

在配置文件中注释掉  **[database]**  因为计算节点不会直接访问数据库

 In the `[DEFAULT]` section, configure `RabbitMQ` message queue access: 

```shell
[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
```

> RABBIT_PASS : mq 密码 ： openstack123

 In the `[DEFAULT]` and `[keystone_authtoken]` sections, configure Identity service access: 

```shell
[DEFAULT]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
```

> NEUTRON_PASS : neutron 密码 ： neutron123 

 In the `[oslo_concurrency]` section, configure the lock path: 

```shell
[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp
```

**3. 配置网络选项：**

`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`

 In the `[linux_bridge]` section, map the provider virtual network to the provider physical network interface: 

```shell
[linux_bridge]
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
```

> PROVIDER_INTERFACE_NAME : linux bridge 对应的网卡名称 ： ens160

 In the `[vxlan]` section, enable VXLAN overlay networks, configure the IP address of the physical network interface that handles overlay networks, and enable layer-2 population: 

```shell
[vxlan]
enable_vxlan = true
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = true
```

> OVERLAY_INTERFACE_IP_ADDRESS : vxlan 的 local ip : 192.168.8.210

 In the `[securitygroup]` section, enable security groups and configure the Linux bridge iptables firewall driver: 

```shell
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

验证 linux bridge 能力;

```shell
net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-ip6tables
```

**4. 修改nova 配置文件：**

/etc/nova/nova.conf 

```shell
[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
```

> NEUTRON_PASS : neutron 密钥 ： neutron123

**5. 重启 nova compute 服务：**

```shell
[root@compute ~]# systemctl restart openstack-nova-compute.service
```

**6. 启动 linux bridge 服务：**

```shell
[root@compute ~]# systemctl enable neutron-linuxbridge-agent.service
[root@compute ~]# systemctl start neutron-linuxbridge-agent.service
```

> 使用 `ps -elf | grep nuetron` 查看服务
>
> controller ： neutron-server、neutron-linuxbridge-agent、l3-agent、neutron-dhcp-agent、neutron-metadata-agent
>
> compute ：neutron-linuxbridge-agent



----------------------------------------



[Reference1>> ]( https://docs.openstack.org/install-guide/index.html )

[Reference2>> ]( https://blog.csdn.net/weixin_39992639/article/details/79033584 )

[Reference3>> ]( https://developer.aliyun.com/article/704987 )