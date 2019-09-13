---
title: redis安装攻略
date: 2016-11-15 13:57:01
categories: redis
tags: redis
---

<img src= "http://i1.piimg.com/1949/e5482fcc749bc356.jpg" style="float:right;margin-top:10px;margin-left:15px;height:140px;" />

本文主要介绍怎样在linux环境下安装redis，官方文档的安装教程也是非常简单的http://redis.io/download。在了解了`redis`的使用场景以及基本原理后，要系统学习它，首先得让它运行起来，然后在慢慢练习并熟悉相关命令，下面让我们一起动手来试一试。
<!-- more -->

### 安装前的准备
#### 准备安装环境及软件
1.linux版本:CentOS7
2.redis版本:redis-3.0.5.tar.gz

#### 下载redis软件
1.使用`wget`方式下载。
```
wget http://download.redis.io/releases/redis-3.0.5.tar.gz
```
2.直接下载
直接在网上下载redis安装包`redis-3.0.5.tar.gz`，然后放在linux服务器上，具体放置路径可以自定义:
```
cd /usr/local/src/
```
以上两种方式都可行，实际情况视个人习惯与具体的环境来决定。

### 开始安装redis
在安装前，默认linux环境已经装好了。
1.解压redis-3.0.5.tar.gz文件，得到redis二进制源码文件：redis-3.0.5
```
[root@localhost src]# ls
redis-3.0.5  redis-3.0.5.tar.gz
[root@localhost src]# cd redis-3.0.5/
```

2.进入源码文件夹redis-3.0.3进行编译
```
[root@localhost src]# cd redis-3.0.5
[root@localhost redis-3.0.5]# make
```

3.成功编译完成后，进入src文件进行安装
```
[root@localhost redis-3.0.5]# cd src/
[root@localhost src]# make install
```

OK，到此，redis在linux的安装就已完成了。

### 组织reis配置和启动文件
`redis`默认安装完成后，其相关的启动文件默认在`/usr/local/bin`目录下。
```
[root@localhost src]# cd /usr/local/bin
[root@localhost bin]# ls
redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-sentinel  redis-server
```
为了方便使用和管理，安装完成后，我将redis的启动文件组织了一下，统一放在专门的`/usr/local/redis/bin`目录中进行管理:
```
[root@localhost bin]# cp -r /usr/local/bin/ /usr/local/redis/
[root@localhost bin]# ls
redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-sentinel  redis-server
[root@localhost bin]# pwd
/usr/local/redis/bin
```
OK，启动和配置文件组织好了，下面就可以启动起来运行试一试了。

### 启动redis服务
首先进入上面存放了redis启动和配置相关文件的专门的管理目录，在启动redis。
```
[root@localhost bin]# ./redis-server 
10545:C 25 Feb 22:50:07.408 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
10545:M 25 Feb 22:50:07.408 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10545
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

10545:M 25 Feb 22:50:07.410 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10545:M 25 Feb 22:50:07.410 # Server started, Redis version 3.0.5
10545:M 25 Feb 22:50:07.410 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10545:M 25 Feb 22:50:07.410 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10545:M 25 Feb 22:50:07.410 * DB loaded from disk: 0.000 seconds
10545:M 25 Feb 22:50:07.410 * The server is now ready to accept connections on port 6379
```

`注意`：目前的启动方式，redis只在前台运行，也就是当前命令窗口关闭，redis服务也就关闭了。如果要让其在后台运行，只需要修改其配置文件即可。

### 编辑redis.conf配置文件
在上述的安装方式中，默认启动加载文的是`/usr/local/redis-3.0.5`路径下的`redis.conf`配置。和上面一样，复制一份到统一的目录下，方便管理，然后修改`redis.conf`中的`daemonize no`为`daemonize yes`，即以守护进程运行。
```
[root@localhost redis-3.0.5]# cp /usr/local/redis-3.0.5/redis.conf /usr/local/redis/conf
```

然后在启动时制动配置文件，如：
```
[root@localhost bin]# ./redis-server ../conf/redis.conf 
[root@localhost bin]# ps -ef | grep redis
root      10713      1  0 22:58 ?        00:00:00 ./redis-server *:6379
root      10717   6898  0 22:58 pts/1    00:00:00 grep --color=auto redis
```
reis已经成功在后台运行了，其进程id是10713

### 连接redis服务
```
[root@localhost bin]# ./redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

### 关闭redis服务
1.通过pkill命令来关闭redis服务
```
[root@localhost bin]# pkill redis
```

2.通过kill进程号来关闭
```
[root@localhost bin]# ps -ef | grep redis
root      3730     1  0 23:21 ?        00:00:00 ./redis-server *:6379           
root      3751  3574  0 23:24 pts/2    00:00:00 grep redis
[root@localhost bin]# kill -9 3730
```
到这里，关于redis的简单安装、启动、关闭都ok，整个过程还是很简单。