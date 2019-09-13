---
title: docker安装指南
date: 2017-04-22 20:00:14
categories: docker
tags: docker
---

Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、bare metal、OpenStack 集群和其他的基础应用平台。
<!-- more -->

### Docker通常用于如下场景
1. web应用的自动化打包和发布。
2. 自动化测试和持续集成、发布。
3. 在服务型环境中部署和调整数据库或其他的后台应用。
4. 适用于微服务系统部署。

### 基于CentOS安装Docker
在centos中安装docker非常简单，只需要一个命令即可搞定。
```shell
yum install docker
```

### 启动docker
在docker安装完成后，并没有启动docker，所以此时执行docker命令会报以下错误按提示：
```shell
[root@localhost ~]# docker info
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```
此时我们需要启动docker后台进程服务：
```shell
[root@localhost ~]# systemctl start docker.service
```
我们也可以将docker后台进程服务设置成开机启动
```shell
[root@localhost ~]# systemctl enable docker.service
```
在启动好了docekr后，我们在来验证下docker是否安装成功且正常启动了：
```shell
[root@localhost ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.6
Storage Driver: devicemapper
 Pool Name: docker-8:3-34594283-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 11.8 MB
 Data Space Total: 107.4 GB
 Data Space Available: 14.26 GB
 Metadata Space Used: 581.6 kB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.147 GB
 Thin Pool Minimum Free Space: 10.74 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: false
 Deferred Deletion Enabled: false
 Deferred Deleted Device Count: 0
 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
 WARNING: Usage of loopback devices is strongly discouraged for production use. Use `--storage-opt dm.thinpooldev` to specify a custom block storage device.
 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
 Library Version: 1.02.107-RHEL7 (2015-10-14)
Logging Driver: journald
Cgroup Driver: systemd
Plugins:
 Volume: local
 Network: null host bridge overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
Security Options: seccomp selinux
Kernel Version: 3.10.0-327.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
Number of Docker Hooks: 2
CPUs: 1
Total Memory: 977.9 MiB
Name: localhost.localdomain
ID: O6KF:7D3F:R62W:OL6K:FREK:FHGR:5YPL:BQZM:OAOF:5EFF:RY47:VZOB
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
Insecure Registries:
 127.0.0.0/8
Registries: docker.io (secure)
```
如果显示为类似上面的信息，则证明docker已经成功安装并启动了。