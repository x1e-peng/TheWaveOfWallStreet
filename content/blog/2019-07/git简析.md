---
date: "2019-07-31T20:18:48+08:00"
draft: false
title: "git简析"
tags: ["git"]
categories: ["杂技浅尝"]
toc: true
---

## git简单介绍
平时经常用到git，这里就做一个简单总结。首先盗个图：
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-07/git.png" />{{% /center %}}

几个专有名词的意思：

>`Workspace：工作区`    
 `Index / Stage：暂存区`    
 `Repository：仓库区（或本地仓库）`    
 `Remote：远程仓库`    
 
 
## 值得注意的几个命令
1、有时候git远端仓库有某个分支，而我在gitlab上merge时，却没有看到这个分支。此时我们需要用到`fetch`这个命令，将远端分支同步到本地仓库。
```git
# 同步所有分支
git fetch

#同步单个分支
git fetch [remote]
```

2、在工作区做了一些修改，想要撤销这些修改。此时我们需要用到`checkout`这个命令。将工作区的修改撤销。
```git
git checkout .
```

3、已经将工作区的修改commit到本地仓库，发现这些commit的代码都是错误的，不想要了，想直接恢复到上一个commit。此时我们需要用到`reset`这个命令。
```git
git reset --hard HEAD^1
```
`HEAD`头指针指向最新一次commit，`HEAD^1`表示最新一次commit的前一次commit。     
`--hard`表示将修改去掉，如果不加，表示保留修改到工作区。
```git
git reset HEAD^1
```

4、已经将工作区新增文件add到暂存区，想要撤销回工作区。     
从2和3我们可以知道，如果是工作区的撤销，我们通常用checkout。如果是暂存区或者本地仓库，我们就用reset。
所以，如果是想将暂存区的修改撤销到工作区：
```git
git reset HEAD
```
如果修改不想要了：
```git
git reset --hard HEAD
```

