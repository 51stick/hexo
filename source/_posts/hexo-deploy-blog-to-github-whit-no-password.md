---
title: Hexo配置ssh key配置
date: 2016-10-06 20:46:44 
categories: hexo
tags: hexo
---

在使用hexo地`hexo-deployer-git`插件进行发布`hexo blog`时， 有时会提醒没无正确的提交权限的错误，到导致无法提交博文到网上上。本文主要讲解如何设置github的ssh key以及免ssh key提交博文。
<!-- more -->

### 前置说明
本文使用的是`windows系统`，所以后续讲述的处理方案均指的是在windows系统下的操作。

### 问题描述
在利用hexo写好博文，使用`hexo d`命令发布时，有时会遇到如下问题:
```shell
Permission denied (publickey).
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Permission denied (publickey).
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```
该问题明确说明，没有许可的提交权限。解决该问题的办法是配置`github`的`ssh key`, 下面来看看具体的配置过程是怎样的。

### 利用git生成ssh key
1.查看`ssh key`文件是否已存在。如果有经了，可以直接跳过该步骤：
```shell
$ ls -al ~/.ssh
total 14
drwxr-xr-x 1 Administrator None    0 Feb 18 11:38 ./
drwxr-xr-x 1 Administrator None    0 Feb 18 11:33 ../
-rw-r--r-- 1 Administrator None 1679 Feb 18 11:33 id_rsa
-rw-r--r-- 1 Administrator None  410 Feb 18 11:33 id_rsa.pub
-rw-r--r-- 1 Administrator None  407 Feb 18 11:38 known_hosts
```
该机器上已经存在`ssh key`文件，所以为了后续讲解，就先删除然后再生成，windows系统中，默认生成的路径为`~/.ssh/`。

2.生成`ssh key`文件
```shell
$ ssh-keygen -t rsa -C "your email"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:FRFyN04Om1vZmjlT1vkrcygf2CK6CtDHgux5MPrhygE chenliang901114@sina.com
The key's randomart image is:
+---[RSA 2048]----+
|        . *o+    |
|         o X + ..|
|          + = +..|
|. o .    . o *  .|
|E* o o  S . *   .|
|+ = o       oo. .|
|.+.o     . + * o |
|.oo..   . . + =  |
|.oo  ..o.    .   |
+----[SHA256]-----+
```
`注意`：
1.讲`ssh-keygen -t rsa -C "your email"`换成你的github的email即可。
2.`Enter file in which to save the key `：指定`ssh key`的生成路径。不用输入，默认是`~/.ssh/`路劲，即`/c/Users/Administrator/.ssh/id_rsa`。
3.`Enter passphrase (empty for no passphrase)`：这个推荐不输入，直接回车，原因是如果输入了，那么以后每次提交github时，都要输入这个密码来验证权限。在提交时比较麻烦费事。如果不输入，那么提交github时则不用验证在输入验证了，一步到位，直接提交。

### 在github上配置ssh key
找到刚才生成ssh key文件中的`id_rsa.pub`的文件，打开，粘贴出来配置到github上，具体步骤如下：
1.登录github点击又上交的下拉按钮(View profile and more)。
2.选择`Settings`选项。
3.进入`Personal settings`界面，选择左边菜单中的`SSH and GPG keys`项。
4.点击右上角的`New SSH key`按钮，将`id_rsa.pub`的内容粘贴至Key中，Title此时可以随便输入。
5.最后点击`Add SSH key`，OK，至此，github ssh key 配置完成。
![示例图](http://p1.bpimg.com/1949/35d3a74df95a3b0e.png)

### 验证ssh key 配置
```shell
$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.253.113' (RSA) to the list of known hosts.
Hi 51stick! You've successfully authenticated, but GitHub does not provide shell access.
```
到此，说明github提交权限已经配置ok了。

### 验证Hexo发布到github
输入hexo发布命令，发布的最后将看到如下结果：
```shell
Warning: Permanently added the RSA host key for IP address '192.30.253.112' to the list of known hosts.
Branch master set up to track remote branch master from git@github.com:51stick/51stick.github.io.git.
To github.com:51stick/51stick.github.io.git
   c22fcb4..b9077d8  HEAD -> master
INFO  Deploy done: git
```
完美，到此，一件发布hexo博文到github配置完成。