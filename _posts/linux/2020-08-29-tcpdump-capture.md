---
layout: post
title: Linux tcpdump 抓包
categories: [Linux]
description: some word here
keywords: Linux
---

# Linux tcpdump 抓包

## tcpdump 使用

### 基本介绍

#### 类型关键字

+ host ：指明一台主机

```bash
host 192.268.9.9
```

+ net ：指明一个网络地址

```bash
net 192.168.0.0
```

+ port ： 指明端口

```bash
port 22
```

#### 方向关键字

+ src : IP包源地址

```bash
src 192.168.20.2
```

+ dst ：IP包目标地址

```bash
dts 192.168.20.3 / dst net 192.168.0.0
```

```bash
src or src / src and dst
```

#### 协议关键字

缺省表示所有协议的包

+ fddi、ip、arp、rarp、tcp、udp

#### 其他关键字

+  gateway、broadcast、less、greater 

#### 表达式

+ `!` or `not`
+ `&&` or `and`
+ `||` or `or`



### 参数详解

+ `-A`：以ASCII码打印每个报文（不包括链路层的头），这对分析网页来说很方便
+ `-a`：将网络地址和广播地址转变成名字
+ `-c <数据包数目>`：在收到指定的包的数目后，tcpdump就会停止
+ `-C`：用于判断用 `-w` 选项将报文写入的文件的大小是否超过这个值，如果超过了就新建文件（文件名后缀是1、2、3依次增加）
+ `-d`：将匹配信息包的代码以人们能够理解的汇编格式输出
+ `-dd`：将匹配信息包的代码以c语言程序段的格式输出
+ `-ddd`：将匹配信息包的代码以十进制的形式输出
+ `-D`：列出当前主机的所有网卡编号和名称，可以用于选项 `-i`
+ `-e`：在输出行打印出数据链路层的头部信息
+ `-f`：将外部的 Internet 地址以数字的形式打印出来
+ `-F <表达文件>` ：从指定的文件中读取表达式，忽略其它的表达式
+ `-i <网卡>`：监听主机的该网卡上的数据流，如果没有指定，就会使用最小网卡编号的网卡（在选项`-D`可知道，但是不包括环路接口），linux 2.2 内核及之后的版本支持 `any` 网卡，用于指代任意网卡
+ `-l`：如果没有使用 `-w` 选项，就可以将报文打印到标准输出终端（默认）
+ `-n`：显示 ip，而不是主机名
+ `-N`：不列出域名
+ `-O`：不将数据包编码最佳化
+ `-p`：不让网卡进入混杂模式
+ `-q`：快速输出，仅列出少数的传输协议信息
+ `-r <数据包文件>` ：从指定的文件中读取包(这些包一般通过`-w`选项产生)
+ `-s <数据包大小>`：指定抓包显示一行的宽度，`-s0`表示可按包长显示完整的包，经常和`-A`一起用，默认截取长度为60个字节，但一般ethernet MTU都是1500字节。所以，要抓取大于60字节的包时，使用默认参数就会导致包数据丢失
+ `-S`：用绝对而非相对数值列出TCP关联数
+ `-t`：在输出的每一行不打印时间戳
+ `-tt`：在输出的每一行显示未经格式化的时间戳
+ `-T <数据包类型>`：将监听到的包直接解释为指定类型的报文，常见的类型有rpc （远程过程调用）和snmp（简单网络管理协议）
+ `-v`：输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息
+ `-vv`：输出详细的报文信息
+ `-w <数据包文件>`：直接将包写入文件中，并不分析和打印结果
+ `expression`：用于筛选的逻辑表达式



### 命令示例

抓取经过第一个网卡的数据包

```bash
tcpdump
```

抓取指定网卡的数据包

```bash
tcpdump -i en0
```

抓取指定网卡，源ip或目的ip的数据包

```bash
tcpdump -i en0 host 10.10.10.10
```

抓取源mac地址为6c:41:6a:ac:01:42的数据包，个数为10 

```shell
tcpdump -i eth1 ether src 6c:41:6a:ac:11:42 -c 10  
```

抓取主机 10.10.10.10 和主机 10.10.10.11 或 10.10.10.12的通信 

```bash
tcpdump host 10.10.10.10 and \(10.10.10.11 or 10.10.10.12 \)
```

 抓取主机 10.10.10.10 除了和主机 10.10.10.11 之外所有主机通信的数据包 

```bash
tcpdump -n host 10.10.10.10 and ! 10.10.10.11
```

 抓取主机 10.10.10.10 除了和主机 10.10.10.11 之外所有主机通信的ip包 

```bash
tcpdump ip -n host 10.10.10.10 and ! 10.10.10.11
```

 抓取主机10.10.10.10发送的所有数据 

```bash
tcpdump -i en0 src host 10.10.10.10 # 源 ip 
```

 抓取主机 10.10.10.10 接收的所有数据 

```bash
tcpdump -i en0 dst host 10.10.10.10 # 目的ip
```

 抓取主机 10.10.10.10 所有在TCP 80端口的数据包 

```bash
tcpdump -i en0 host 10.10.10.10 and tcp port 80
```

 抓取HTTP主机 10.10.10.10 在80端口接收到的数据包 

```bash
tcpdump -i en0 host 10.10.10.10 and dst port 80
```

 抓取所有经过 en0，目的或源端口是 25 的网络数据 

```bash
tcpdump -i en0 port 25
# 源端口
tcpdump -i en0 src port 25
# 目的端口
tcpdump -i en0 dst port 25 
```

 抓取所有经过 en0，网络是 192.168上的数据包

```bash
tcpdump -i en0 net 192.168
tcpdump -i en0 src net 192.168
tcpdump -i en0 dst net 192.168
tcpdump -i en0 net 192.168.1
tcpdump -i en0 net 192.168.1.0/24
```

 协议过滤 

```bash
tcpdump -i en0 arp
tcpdump -i en0 ip
tcpdump -i en0 tcp
tcpdump -i en0 udp
tcpdump -i en0 icmp
```

 抓取所有经过 en0，目的地址是 192.168.1.254 或 192.168.1.200 端口是 80 的 TCP 数据 

```bash
tcpdump -i en0 '((tcp) and (port 80) and ((dst host 192.168.1.254) or (dst host 192.168.1.200)))'
```

 抓取所有经过 eth1，目标 MAC 地址是 00:01:02:03:04:05 的 ICMP 数据 

```bash
tcpdump -i eth1 '((icmp) and ((ether dst host 00:01:02:03:04:05)))'
```

 抓取所有经过 en0，目的网络是 192.168，但目的主机不是 192.168.1.200 的 TCP 数据 

```bash
tcpdump -i en0 '((tcp) and ((dst net 192.168) and (not dst host 192.168.1.200)))'
```

 

[Reference >> ]( https://www.jianshu.com/p/a62ed1bb5b20)

