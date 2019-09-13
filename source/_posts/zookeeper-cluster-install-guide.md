---
title: zookeeper集群搭建攻略
date: 2016-11-12 20:00:14
categories: zookeeper
tags: zookeeper
---

<img src= "http://p1.bqimg.com/1949/e72f4b926e408381.png" style="float:right;margin-top:10px;margin-left:15px;height:150px;" />

最近工作中使用到了`zookeeper`，于是决定着手研究一下`zookeeper`的原理、使用场景、搭建方式等，本文主要讲解`zookeeper`的安装（先把环境搭好运行起来再说别的）。后面后时间在具体分析`zookeeper`相关知识。`zookeeper`有三种运行模式，分别是`集群模式`、`伪集群模式`、`单机模式`，本文主要搭建的是集群模式，也是工作中使用的模式，所以也推荐直接搭建`zookeeper`的集群模式。
<!-- more -->

### 安装前的准备
#### 准备linux环境
```
虚拟机环境centos 7
```

#### 准备jdk环境
```
[root@localhost bin]# java -version
java version "1.7.0_75"
Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)
```

#### 准备zookeper
下载地址：[zookeeper-3.4.6.tar.gz](http://www.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz)

### 开始安装zookeeper
#### 集群模式
集群模式就是在多个服务节点上安装`zookeeper`，然后通过相关配置关联起来，形成一个集群。下面开始`zookeeper`的集群模式的搭建。
1.将下载好的`zookeeper-3.4.6.tar.gz`文件放置在`/opt`目录下并解压。如下：
```
[root@localhost opt]# tar -zxvf zookeeper-3.4.6.tar.gz 
[root@localhost opt]# ls
zookeeper-3.4.6  zookeeper-3.4.6.tar.gz
```

2.重命名解压包。如下：
```
[root@localhost opt]# mv zookeeper-3.4.6 zookeeper
[root@localhost opt]# ls
zookeeper  zookeeper-3.4.6.tar.gz
```

3 配置zookeeper，进入到zookeeper/conf目录下，拷贝配置文件示例`zoo_sample.cfg`得到`zoo.cfg`
```
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg
[root@localhost conf]# ls
configuration.xsl  log4j.properties  zoo.cfg  zoo_sample.cfg
```

4 编辑配置文件zoo.cfg
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/var/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

server.1=192.168.254.130:2888:3888
server.2=192.168.254.131:2888:3888
server.3=192.168.254.132:2888:3888
```
`注意:`
1.配置数据节点，`dataDir=/var/zookeeper`，该位置可以自定义。
2.添加服务节点配置，格式为`server.id=ip：port：port`，id被称为Server ID，范围在1~255。

5 配置myid文件。在数据节点指定的目录中，添加`myid`配置文件，id编号和zoo.cfg中的服务节点ip对应。该配置文件只有一行内容，并且是数字，对应服务器节点的Server ID。

6 其他机器节点做如上相应配置，对应的myid配置，参考zoo.cfg文件。

---

#### 伪集群模式
所谓伪集群，简单点说，集群所有服务节点都在一台机器上，但是还是以集群的方式对位提供服务。部分配置如下：
```
server.1=192.168.254.130:2888:3888
server.2=192.168.254.130:2889:3889
server.3=192.168.254.130:2890:3890
```

#### 单机模式
单机模式只是一种特殊的集群模式，为便于理解，可以粗略地认为集群中只有一个服务节点的集群。单机模式的部署步骤和集群模式的部署步骤基本一致，只是在zoo.cfg配置文件上有点差异，只配置本机服务节点即可。部分配置如下：
```
server.1=192.168.254.130:2888:3888
```

### 安装总结
通常来说，不管是企业还是个人研究，都已集群模式为主，伪集群和单机模式用的较少。

### 启动zookeeper
1.进入zk的bin目录
```
[root@localhost conf]# cd /opt/zookeeper/bin/
[root@localhost bin]# ls
README.txt  zkCleanup.sh  zkCli.cmd  zkCli.sh  zkEnv.cmd  zkEnv.sh  zkServer.cmd  zkServer.sh  zookeeper.out
[root@localhost bin]# 
```

2.启动zookeeper
```
[root@localhost bin]# ./zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@localhost bin]# 
```
启动全部的`zookeeper`服务节点，只要节点成功启动超过半数，即存活的`zookeeper`节点超过半数，`zooLeeper`集群就能正常工作。

接下来可以通过相关命令来查看`zookeeper`的相关信息：
1.`./zkServer.sh status`：查看是否是领导者还是跟随者。
2.通过telnet工具来查看客户端是否成功启动。
```
[root@localhost bin]# telnet 192.168.211.128 2181
Trying 192.168.211.128...
Connected to 192.168.211.128.
Escape character is '^]'.
```
注意：如果没有安装telnet，可以使用如下命令来安装:
```
yum install telnet
```

然后使用stat命令来查看集群的启动信息。
```
[root@localhost bin]# telnet 192.168.211.128 2181
Trying 192.168.211.128...
Connected to 192.168.211.128.
Escape character is '^]'.
stat
Zookeeper version: 3.4.6-1569965, built on 02/20/2014 09:09 GMT
Clients:
 /192.168.211.128:47573[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 4
Connection closed by foreign host.
```
`注意:`有可能在输入stat命令是报如下错误：

```
[root@localhost bin]# telnet 192.168.211.128 2181
Trying 192.168.211.128...
Connected to 192.168.211.128.
Escape character is '^]'.
stat
This ZooKeeper instance is not currently serving requests
Connection closed by foreign host.
```
以上错误原因有一下两点：
1.集群没有选出来leader，当集群里的结点只剩下一台，或者不足半数时，集群进入崩溃恢复阶段，该阶集群不对外提供服务，就会出现这个错误提示。
2.集群配置的端口被防火墙屏蔽，解决方法是暴露集群端口。如：
```
[root@localhost bin]# firewall-cmd --zone=public --add-port=2181/tcp --permanent
[root@localhost bin]# firewall-cmd --zone=public --add-port=2888/tcp --permanent
[root@localhost bin]# firewall-cmd --zone=public --add-port=3888/tcp --permanent
[root@localhost bin]# firewall-cmd --reload
```
然后关闭集群，重启启动，OK，解决问题，集群成功启动起来，客户端访问正常，到此，`zookeeper`集群的简单搭建也就完成了，接下来就可以直接在服务器上学习和使用其相关命令，也可以使用程序来连接兵与其通信。
