---
title: nginx安装攻略
date: 2017-02-15 22:38:48
categories: nginx
tags: nginx
---

<img src= "http://i1.piimg.com/1949/f9bce752a3999037.png" style="float:right;margin-top:10px;margin-left:15px;" />

最近在工作中需使用配置中心，将配置文件集中起来，做统一的管理，于是打算先研究一下百度的`disconf`，看看能否引入项目中过来使用，首先，在安装`disconf`时，需要使用到`nginx`，于是准备在linux上先把nginx装上。以前也装上`nginx`研究过，感觉安装还是比较的简单，所以也就没做笔记。这回借这个机会，顺便把安装笔也一同给补上，作为以后学习回顾的记录。
<!-- more -->

### 前言
`nginx`是用C语言开发的，推荐安装在linux环境上运行，所以本文的linux使用的是centos7，当然window上也能安装，一键式安装，相对来说更简单。好了，下面就将安装的过程及顺序记录下来(不同的linux版本安装顺序大致相同，由于系统自带的工具不同，可能安装相对来说有点出入，但需要的依赖环境和组件是相同的)。

### 安装GNU编译器套件(gcc)
作用是用来编译`nginx`源代码的，不多说了，想要了解更多的，请自行google或[百度一下](https://www.baidu.com/)。
```shell
yum install gcc-c++
```

正常情况下，如果看到如下信息，则表示`gcc-c++`成功安装完成。
```shell
Installed:
  gcc-c++.x86_64 0:4.8.5-11.el7                                                 

Dependency Installed:
  libstdc++-devel.x86_64 0:4.8.5-11.el7                                         

Dependency Updated:
  cpp.x86_64 0:4.8.5-11.el7               gcc.x86_64 0:4.8.5-11.el7            
  libgcc.x86_64 0:4.8.5-11.el7            libgomp.x86_64 0:4.8.5-11.el7        
  libstdc++.x86_64 0:4.8.5-11.el7        

Complete!
```

###  安装 PCRE pcre-devel
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用的是`pcre`来解析正则表达式，所以需要在 linux上安装`pcre`库，`pcre-devel`是使用 `pcre`的一个二次开发库。nginx也需要此依赖库。
```shell
yum install -y pcre pcre-devel
```

正常情况下，如果看到如下信息，则表示`PCRE pcre-devel`成功安装完成。
```shell
Total                                              657 kB/s | 899 kB  00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : pcre-8.32-15.el7_2.1.x86_64                                  1/3 
  Installing : pcre-devel-8.32-15.el7_2.1.x86_64                            2/3 
  Cleanup    : pcre-8.32-15.el7.x86_64                                      3/3 
  Verifying  : pcre-8.32-15.el7_2.1.x86_64                                  1/3 
  Verifying  : pcre-devel-8.32-15.el7_2.1.x86_64                            2/3 
  Verifying  : pcre-8.32-15.el7.x86_64                                      3/3 

Installed:
  pcre-devel.x86_64 0:8.32-15.el7_2.1                                           

Updated:
  pcre.x86_64 0:8.32-15.el7_2.1                                                 

Complete!
```

### 安装zlib
`zlib`库提供了很多种压缩和解压缩的方式,nginx使用`zlib`对http包的内容进行gzip，所以需要在Centos上需要安装此依赖库。
```shell
yum install -y zlib zlib-devel
```

正常情况下，如果看到如下信息，则表示`zlib`成功安装完成。
```shell
Total                                              182 kB/s | 140 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : zlib-1.2.7-17.el7.x86_64                                     1/3 
  Installing : zlib-devel-1.2.7-17.el7.x86_64                               2/3 
  Cleanup    : zlib-1.2.7-15.el7.x86_64                                     3/3 
  Verifying  : zlib-devel-1.2.7-17.el7.x86_64                               1/3 
  Verifying  : zlib-1.2.7-17.el7.x86_64                                     2/3 
  Verifying  : zlib-1.2.7-15.el7.x86_64                                     3/3 

Installed:
  zlib-devel.x86_64 0:1.2.7-17.el7                                              

Updated:
  zlib.x86_64 0:1.2.7-17.el7                                                    

Complete!
```

### 安装OpenSSL
`OpenSSL`是一个强大的安全套接字层密码库，包括主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议。nginx不仅支持http协议，还支持 https（即在ssl协议上传输http），所以需要在Centos安装此依赖库。
```shell
yum install -y openssl openssl-devel
```

正常情况下，如果看到如下信息，则表示`OpenSSL`成功安装完成。
```shell
Total                                              827 kB/s | 8.7 MB  00:10     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : libcom_err-1.42.9-9.el7.x86_64                              1/42 
  Updating   : libsepol-2.5-6.el7.x86_64                                   2/42 
  Updating   : libselinux-2.5-6.el7.x86_64                                 3/42 
  ......                     
  Verifying  : systemd-python-219-19.el7.x86_64                           40/42 
  Verifying  : libselinux-2.2.2-6.el7.x86_64                              41/42 
  Verifying  : 1:openssl-1.0.1e-42.el7.9.x86_64                           42/42 

Installed:
  openssl-devel.x86_64 1:1.0.1e-60.el7_3.1                                      

Dependency Installed:
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7  krb5-devel.x86_64 0:1.14.1-27.el7_3 
  libcom_err-devel.x86_64 0:1.42.9-9.el7    libkadm5.x86_64 0:1.14.1-27.el7_3   
  libselinux-devel.x86_64 0:2.5-6.el7       libsepol-devel.x86_64 0:2.5-6.el7   
  libverto-devel.x86_64 0:0.2.5-4.el7      

Updated:
  openssl.x86_64 1:1.0.1e-60.el7_3.1       systemd.x86_64 0:219-30.el7_3.6      

Dependency Updated:
  e2fsprogs.x86_64 0:1.42.9-9.el7                                               
  e2fsprogs-libs.x86_64 0:1.42.9-9.el7                                          
  krb5-libs.x86_64 0:1.14.1-27.el7_3                                            
  krb5-workstation.x86_64 0:1.14.1-27.el7_3                                     
  libcom_err.x86_64 0:1.42.9-9.el7                                              
  libgudev1.x86_64 0:219-30.el7_3.6                                             
  libselinux.x86_64 0:2.5-6.el7                                                 
  libselinux-python.x86_64 0:2.5-6.el7                                          
  libselinux-utils.x86_64 0:2.5-6.el7                                           
  libsepol.x86_64 0:2.5-6.el7                                                   
  libss.x86_64 0:1.42.9-9.el7                                                   
  openssl-libs.x86_64 1:1.0.1e-60.el7_3.1                                       
  systemd-libs.x86_64 0:219-30.el7_3.6                                          
  systemd-python.x86_64 0:219-30.el7_3.6                                        
  systemd-sysv.x86_64 0:219-30.el7_3.6                                          

Complete!
```

### 下载nginx
1.官网下载：[https://nginx.org/en/download.html](https://nginx.org/en/download.html)
2.使用wget命令下:wget -c https://nginx.org/download/nginx-1.10.1.tar.gz

```shell
[root@localhost opt]# wget -c https://nginx.org/download/nginx-1.10.1.tar.gz
--2017-02-24 06:11:12--  https://nginx.org/download/nginx-1.10.1.tar.gz
Resolving nginx.org (nginx.org)... 206.251.255.63, 95.211.80.227, 2001:1af8:4060:a004:21::e3, ...
Connecting to nginx.org (nginx.org)|206.251.255.63|:443... connected.
ERROR: cannot verify nginx.org's certificate, issued by ‘/C=EN/CN=Sample CA 2’:
  Unable to locally verify the issuer's authority.
To connect to nginx.org insecurely, use `--no-check-certificate'.
```
如果看到如上的提示信息，说明`nginx.com`证书验证失败，提示使用无证书下载，即加上提示中的`--no-check-certificate`参数。如下：

```shell
[root@localhost ~]# cd /opt
[root@localhost opt]# wget -c --no-check-certificate https://nginx.org/download/nginx-1.10.1.tar.gz
```

### 解压nginx
```shell
[root@localhost opt]# tar -zxvf nginx-1.10.1
```

### 配置nginx
进入nginx的解压目录
```shell
[root@localhost opt]# cd nginx-1.10.1/
[root@localhost nginx-1.10.1]# ./configure
```

说明：`./configure`命令使用的是nginx默认的配置方式。其默认的相关配置会在配置完成后显示出来，如：
```shell
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + md5: using system crypto library
  + sha1: using system crypto library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

当然，也可以自定义配置安装，如：
```shell
./configure \
--prefix=/usr/local/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/conf/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```
这里我就使用默认的配置安装。

### 编译安装
```shell
[root@localhost nginx-1.10.1]# make & make install
......
test -d '/usr/local/nginx/logs' \
	|| mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/html' \
	|| cp -R html '/usr/local/nginx'
test -d '/usr/local/nginx/logs' \
	|| mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/opt/nginx-1.10.1'
[1]+  Done                    make
```

看到如上信息，表示`nginx`安装成功。
### 启动nginx
1.首先查找默认配置中安装的路径
```shell
[root@localhost nginx-1.10.1]# whereis nginx
nginx: /usr/local/nginx
```

2.启动nginx
```shell
[root@localhost nginx-1.10.1]# cd /usr/local/nginx
[root@localhost nginx]# cd sbin/
[root@localhost sbin]# ./nginx 
```

然后在浏览器访问`http://localhost`，正常情况下就能看到如下页面信息。

![Nginx首界面](http://p1.bpimg.com/1949/1cde0025231d46e8.png)
到此，nginx就安装成功了。

`注意`：ngxin安装好，默认监听的是本机的`80端口`，如果是安装在虚拟机中，有可能本地实体机无法访问到虚拟机中的nginx主界面，原因可能是`80端口`被防火墙屏蔽了，解决办法是暴露`80端口`即可。如：“
```sheel
[root@localhost ~]# /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

3.其他相关命令
3.1 `/nginx -s stop`：强制关闭，会先查询到nginx的进程id，然后再使用kill命令杀死进程。
3.2 `/nginx -s quit`：待nginx进程处理完所有任务安全的进行停止。
3.3 `/nginx -s reload`：平滑的重新加载配置文件，即执行完当前正在处理的任务，然后再重新加载修改后的配文件。

