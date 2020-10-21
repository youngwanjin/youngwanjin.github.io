---
layout: post
title: iperf 使用
categories: [Linux]
description: iperf 使用
keywords: Linux,Network
---

# iperf 使用

iperf 是一个网络性能测试工具，可以测试 TCP 和 UDP 的带宽质量，可以测量最大 TCP 带宽，具有多种参数和UDP特性，iperf 可以测试带宽、吞吐量、延迟、丢包率等 ，我们可以使用 iperf 来测试路由器、防火墙、交换机的性能

iperf 是一个 c/s 架构，有 server 端和 client 端，分为两个版本 iperf2、iperf3，iperf2 是比较老的版本，建议使用 iperf3 比较准确

### 参数介绍

#### 服务端参数

```shell
-s 以服务器模式启动
-u 单线程 UDP 模式
-d 以守护进程模式运行
```

#### 客户端参数

```shell
-b 指定客户端通过UDP协议发送信息的带宽，默认值为1Mbit/s
-c 指定服务器地址　　
-d 同时进行双向传输测试
-n 指定传输的字节数
-r 单独进行双向传输测试
-t 指定Iperf测试时间，默认10秒
-F 指定需要传输的文件
-I 从标准输入（stdin）中读取要传输的数据 
-L 指定一个端口，服务器将利用这个端口与客户机连接
-P 客户端到服务器的连接数，默认值为1
-T 指定ttl值
```

#### 共用参数

```shell
-f --格式[k|m|K|M] 分别表示以Kbits、Mbits、KBytes、MBytes显示报告，默认是Mbits
-i 以秒为单位统计带宽值 
-l 读写缓冲区大小，默认是8KB
-m 显示最大的TCP数据段大小 (MTU - TCP/IP header)
-o 将报告和错误信息输出到文件
-p 指定服务器和客户端连接的端口
-u 使用udp协议
-w 指定TCP窗口大小，默认是8KB
-B 绑定一个主机地址或接口（当主机有多个地址或接口时使用该参数）
-C 兼容旧版本（当server端和client端版本不一样时使用）
-M 设定TCP数据包的最大mtu值
-N 设定TCP不延时
-V 传输ipv6数据包
```

### 使用示例

服务端启动一个 server 进程

```shell
iperf –s -p 5001 -u # udp 测试
```

客户端启动一个 client 进程

```shell
iperf -c SERVERIP -i 1 -t 30 -b 300k -u -p 5001  # udp 测试，数据包大小为300
```

tcp 测试 server 端

```shell
iperf –s –p 12345 –i 1 –M
```

tcp 测试 client 端

```shell
iperf –c SERVERIP –p 12345 –i 1 –t 10 –w 20K
```





[Reference1>>](https://www.nixops.me/articles/iperf-check-bandwidth.html)

[Reference2>>](http://www.enkichen.com/2017/06/06/iperf-introduce/)