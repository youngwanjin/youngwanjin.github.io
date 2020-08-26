---
layout: post
title: 关闭 UseDNS 加速 ssh 登录
categories: [Linux]
description: some word here
keywords: Linux
---

# 关闭 UseDNS 加速 ssh 登录

有时侯ssh登录服务器时，总是要稍等一下才能连接上，这是因为OpenSSH服务器有一个DNS查找选项UseDNS默认情况下是打开的。

UseDNS 选项打开状态下，当客户端试图登录SSH服务器时，服务器端先根据客户端的IP地址进行DNS PTR反向查询出客户端的主机名，然后根据查询出的客户端主机名进行DNS正向A记录查询，验证与其原始IP地址是否一致，这是防止客户端欺骗的一种措施，但一般我们的是动态IP不会有PTR记录，所以没有必要打开这个选项。

```bash
[root@controller ~]# vim /etc/ssh/sshd_config
# 添加注释
#UseDNS yes
# 添加配置
UseDNS no
# 重启服务
[root@controller ~]# systemctl restart sshd
```



