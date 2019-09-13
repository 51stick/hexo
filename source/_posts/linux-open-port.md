---
title: 如何在linux上开放指定端口号?
date: 2017-04-14 19:20:14
categories: linux
tags: linux
---

有时候，在linux上访问一个应用程序时，明明已将正常启动，但就是无法访问，此时可能的原因之一是程序的访问端口号没有对外开放，导致无法正确访问，本文将介绍如何在linux上开放指定端口号。
<!-- more -->

### 方法一：开放指定端口
```
# 开放8081端口  permanent为永久开放
firewall-cmd --zone=public --add-port=8081/tcp --permanent

# 重新读取配置
firewall-cmd --reload

# 查看已经开放的端口
firewall-cmd --list-all
```

### 方法二：开放指定端口
`注意`：这种开放端口号的方式重启后设置可能会失效。
```
iptables -A INPUT -ptcp --dport 8081 -j ACCEPT
```

### 方法三：开放指定端口

#### 修改iptables文件
`注意`：这种开放端口的修改方式重启后不会失效。
编辑`/etc/sysconfig/iptables`文件插入以下内容：
```
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8081 -j ACCEPT
```

#### 重启服务iptables服务
```
service iptables restart
```