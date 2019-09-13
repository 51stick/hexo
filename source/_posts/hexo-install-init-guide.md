---
title: hexo环境安装与初始化指南
date: 2016-10-05 23:50:54
categories: hexo
tags: hexo
---

本文主要讲述怎样在windows环境下安装并初始化hexo博客站点，涉及了hexo的基本安装。详细的配置请参考文所指的官网的指导。其中，文章中会使用到git工具，在此也默认读者会使用该工具，如果不熟悉的，请百度一下参考并学会使用。

<!-- more -->

### 准备Hexo环境所需软件
1. [下载 Git](https://git-scm.com/)
2. [下载 node.js](https://nodejs.org/en/)

### 安装Hexo使用环境
1. 安装 Git
2. 安装 node.js
3. 设置npm淘宝镜像站，原因是在国内使用npm下载速度慢。配置方式如下：

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### 检查Hexo环境是否安装成功
打卡git客户端，以此输入如下命令：
```
git --version
node -v
npm -v
```
如果能查看到相关软件的版本信息，则证明安装成功。

### 安装Hexo
以此在git客户端中执行如下命令：
```
cnpm install -g hexo-cli
cnpm install hexo --save
```
以上两个命令就是安装hexo，安装完成后，输入如下命令来验证hexo是否安装成功：
```
$ hexo -v
```
如何hexo安装成功，输入上述命令则会看到类似以下的信息：
```
hexo: 3.2.2
hexo-cli: 1.0.2
os: Windows_NT 6.1.7601 win32 x64
http_parser: 2.7.0
node: 6.9.2
v8: 5.1.281.88
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 57.1
modules: 48
openssl: 1.0.2j
```

### 初始化Hexo静态站点
按顺序执行如下命令
```
hexo init <folder>
cd <folder>
cnpm install
```
至此，会在指定文件夹下生成相应的站点文件。详细可以参考[hexo指导文档](https://hexo.io/zh-cn/docs/)。
### 插件安装
```
cnpm install hexo-server --save
cnpm install hexo-admin --save
cnpm install hexo-generator-archive --save
cnpm install hexo-generator-feed --save
cnpm install hexo-generator-search --save
cnpm install hexo-generator-tag --save
cnpm install hexo-deployer-git --save
cnpm install hexo-generator-sitemap --save
cnpm install hexo-renderer-jade --save
cnpm install hexo-renderer-sass --save
```
在使用其他的一些功能时，需要插件的支持，因此可以选择性的安装所需插件。

### 主题安装
下载主题
```
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
主题安装完成后，主题的名字就是主题所在文件夹的名字，要启用新的主题，只需要修改站点根目录的`_config.yml`文件的`theme`配置项，如：
```
theme: next
```

### 运行Hexo静态站点
Hexo安装完成后，在初始化站点下使用如下命令集合运行起来：
1. hexo g：编译生成静态资源文件
2. hexo s：运行hexo
3. hexo new  post postName：该命令会在your-hexo-site\source\_posts文件夹下生成一个叫postName的博客文件。

### 更多参考
更多关于Hexo请查询考[hexo官方指南](https://hexo.io/)
更多关于`next`主题请参考[hexo-theme-next官方指南](http://theme-next.iissnan.com/getting-started.html)