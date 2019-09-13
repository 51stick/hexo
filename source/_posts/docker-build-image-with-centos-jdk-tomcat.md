---
title: 从零开始构建 centos+jdk8+tomcat8 Docker镜像
date: 2017-04-23 19:20:14
categories: docker
tags: docker
---

本文主要介绍从零开始构建一个基于centos环境，具有jdk，tomcat的docker镜像。展示了怎样从一个基础镜像构建一个自己需要的镜像。下面具体来看看如何操作。
<!-- more -->

### 开始说明
本文默认所在linux环境已经安装好了docker。

### 环境准备
首先介绍下本文是要的jdk和tomcat版本。
1. jdk-8u131-linux-x64.tar.gz
2. apache-tomcat-8.5.14.tar.gz

### 文件准备
将jdk和tomcat上传到指定的linux路径下并解压。如：
```shell
[root@localhost docker]# ls
apache-tomcat-8.5.14  jdk1.8.0_131
```

### 准备基础镜像文件
从`Docker Hub`拉去基础镜像文件。
```shell
[root@localhost ~]# docker pull centos
```
默认拉去的是tag为latest的centos镜像，镜像拉取成功后，我们可以使用`docker images`命令查看本地镜像文件。
```shell
[root@localhost docker]# docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos                  latest              a8493f5f50ff        4 weeks ago         192.5 MB         
```
此时`docker.io/centos`就是从`Docker Hub`拉取下来的基础镜像文件。

### 编写Dockerfile文件
```
FROM docker.io/centos:latest

MAINTAINER liang.chen "229408705@qq.com"

RUN mkdir -p /opt/java/jdk1.8.0_131
RUN mkdir -p /opt/tomcat/apache-tomcat-8.5.14

ADD ./jdk1.8.0_131/ /opt/java/jdk1.8.0_131/
ADD ./apache-tomcat-8.5.14/ /opt/tomcat/apache-tomcat-8.5.14/

ENV JAVA_HOME /opt/java/jdk1.8.0_131
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH .:$JAVA_HOME/lib:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /opt/tomcat/apache-tomcat-8.5.14
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin

EXPOSE 8080

CMD ["/opt/tomcat/apache-tomcat-8.5.14/bin/catalina.sh","run"]
```

### 编写构建脚本
```
#!/bin/bash
docker build --no-cache=true -t "repos_local/centos-jdk8-tomcat8:0.0.1" .
```
注意：此处的构建脚本只有一句简单的docker构建的命令，实际工作中，docker的构建脚本可能会涉及到其他很多逻辑。例如，在构建前拉去项目应用包，项目配置文件等。

### 开始构建镜像
执行构建脚本，如：
```shell
[root@localhost docker]# ./build.sh 
Sending build context to Docker daemon 586.2 MB
Step 1 : FROM docker.io/centos:latest
 ---> a8493f5f50ff
Step 2 : MAINTAINER liang.chen "229408705@qq.com"
 ---> Running in 601bdbffbde4
 ---> 26e8cdd86433
Removing intermediate container 601bdbffbde4
Step 3 : RUN mkdir -p /opt/java/jdk1.8.0_131
 ---> Running in 4baabb3936b6
 ---> 7b1738e4e96c
Removing intermediate container 4baabb3936b6
Step 4 : RUN mkdir -p /opt/tomcat/apache-tomcat-8.5.14
 ---> Running in d0afc9b3cd18
 ---> 298988f65c6a
Removing intermediate container d0afc9b3cd18
Step 5 : ADD ./jdk1.8.0_131/ /opt/java/jdk1.8.0_131/
 ---> 8f4398e2ac65
Removing intermediate container 532486b90ea9
Step 6 : ADD ./apache-tomcat-8.5.14/ /opt/tomcat/apache-tomcat-8.5.14/
 ---> 913dd4b486bf
Removing intermediate container 9d6e553ba0aa
Step 7 : ENV JAVA_HOME /opt/java/jdk1.8.0_131
 ---> Running in 19ae93f7e0dd
 ---> 01c2503c567e
Removing intermediate container 19ae93f7e0dd
Step 8 : ENV JRE_HOME $JAVA_HOME/jre
 ---> Running in 11689dfd11ec
 ---> c40790cf116b
Removing intermediate container 11689dfd11ec
Step 9 : ENV CLASSPATH .:$JAVA_HOME/lib:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ---> Running in 02576f380d9b
 ---> f64f603a91e3
Removing intermediate container 02576f380d9b
Step 10 : ENV CATALINA_HOME /opt/tomcat/apache-tomcat-8.5.14
 ---> Running in 09c7f880a31a
 ---> 8aac237520a3
Removing intermediate container 09c7f880a31a
Step 11 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin
 ---> Running in 9540a4f48a1c
 ---> f4718ff45225
Removing intermediate container 9540a4f48a1c
Step 12 : EXPOSE 8080
 ---> Running in 27a8ec657673
 ---> 7b80cd06456a
Removing intermediate container 27a8ec657673
Step 13 : CMD /opt/tomcat/apache-tomcat-8.5.14/bin/catalina.sh run
 ---> Running in 51365559218c
 ---> 8b730c606224
Removing intermediate container 51365559218c
Successfully built 8b730c606224
```
如此表示镜像已经构建成功，`jdk1.8.0_131`和`apache-tomcat-8.5.14`也已经成功的打进到我们新构建的镜像文件中了。

```shell
[root@localhost docker]# docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
repos_local/centos-jdk8-tomcat8   0.0.1               8b730c606224        1 hours ago         581.9 MB
docker.io/centos                  latest              a8493f5f50ff        4 weeks ago         192.5 MB
```
现在可以看到，除了基础镜像文件外，多了一个`repos_local/centos-jdk8-tomcat8`镜像文件，该文件就是我们上面构建的`centos+jdk+tomcat`的镜像文件了。

### 运行一个docker容器
最后，我们来根据新构建的镜像文件，运行一个容器，该容器会启动tomcat。
```shell
docker run -d -p 8081:8080 --name test-jdk8-tomcat8 repos_local/centos-jdk8-tomcat8:0.0.1
```
注意：
`-d`：后台运行docker容器。
`-p`：端口映射。宿主机port:容器port，如此在访问宿主机的ip+port时，就能访问到docker容器中的 应用端口。
`--name`：容器名称, 容器id难以记忆，因此可以通过容器名称对容器操作。

### 查看tomcat启动情况
由于启动docker容器时没有将tomcat的日志文件映射出来，因此可以直接进入容器中查看启动日志。
```shell
[root@localhost docker]# docker exec -it test-jdk8-tomcat8 /bin/bash
[root@8cb542667072 /]# 
```
此时用户名已将切换，证明已经进入到运行中的docker容器中了。
注意:
	`-i`：捕获标准输入输出。
	`-t`：分配一个终端或控制台。

最后在浏览器中测试下能否访问tomcat，如：http://192.168.244.128:8081

![Markdown](http://i4.buimg.com/1949/3cf760a6f497891b.png)

OK,汤猫也成功起来了，证明我们docker镜像构建成功，docker容器启动成功，容器中的应用也正常启动了。

### 问题总结
#### 开放指定端口
开始时，就算tomcat正常启动了，外面也无法正常访问tomcat主界面，因为有可能端口号没有开放出来，外部不能直接访问。

永久开放指定端口：
```
# 开放8081端口  permanent为永久开放
firewall-cmd --zone=public --add-port=8081/tcp --permanent

# 重新读取配置
firewall-cmd --reload

# 查看已经开放的端口
firewall-cmd --list-all
```