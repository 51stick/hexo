---
title: git branch 常用命令
date: 2016-10-15 14:01:55
categories: git
tags: git
---

众所周知，git是现在主流的版本控制软件，工作中使用的也是比较多，本文列出了本人工作中用得比较多的几个命令，一方面作为笔记查看，其次也是个人的一个小小汇总。
<!-- more -->

### 查看分支
`git branch` : 只能查看本地分支。
`git branch -r` : 只能查看远程分支。
`git branch -a` : 查看所有分支，包括本地和远程服务骑上的分支。

---

### 创建分支
`git branch [branch-name]` : 创建本地分支，并停留在当前分支中。
`git branch -b [branch-name]` : 创建本地分支，并切换到新建分支中。
`git checkout [branch-name]` :  切换分支，在切换分支前，确保当前分支已经`commit`  。
`git push origin [branch-name]` :  将本地分支推送到远端服务器。

---

### 删除分支
`git branch -d [branch-name]` : 删除本地分支，删除前必须要合并本地分支。
`git branch -D [branch-name]` : 强制删除本地分支, 删除前不必一定要合并本地分支。
`git push origin :[branch-name]` : 删除指定的远程服务器分支。
`注意:`origin之后和冒号之前必须要空格，表示将空分支推送到远端，也就是删除远端分支。

---

### clone分支
`git clone <URL>` : 默认clone方式，会将整个仓库都clone下来，但是只会在本地创建一个master分支。
`git clone -b [branche-name] <URL>` : 只克仓库指定的分支，而不是将远端所有分支都clone下来。

### 重命名分支
`git branch -m origin devlop` ：将本地origin分支重命名为devlop分支。
重命名远程分支，即先删除远程分支，再将重命名后的分支推送到服务器上。