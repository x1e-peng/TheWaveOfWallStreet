+++
date = "2019-08-02T15:29:37-07:00"
draft= false
title= "git三对象commit、tree、blob"
tags= ["git"]
categories= ["杂技浅尝"]
toc= true
+++

## git对象
Git 是一套内容寻址文件系统。从核心上来看不过是简单地存储键值对（key-value）。它允许插入任意类型的内容，并会返回一个键值，通过该键值可以在任何时候再取出该内容。

可以通过底层命令 hash-object 来示范这点，传一些数据给该命令，它会将数据保存在 .git 目录并返回表示这些数据的键值。
```git
➜  xp329486175.github.io git:(master) find .git/objects
.git/objects/22
.git/objects/25
➜  xp329486175.github.io git:(master) find .git/objects/25 -type f
.git/objects/25/d9818432ffbdb20f482dbefc07b38634b9b748
.git/objects/25/1263c13a2ad0ffe6916dd8c9a28cb3e46b1442
.git/objects/25/1ee10240462a1ecf1cb7e4765a0554b4af154a
.git/objects/25/5b178360d88dcb5cd9ed13b8a4ed666b632cb0
.git/objects/25/9c460600f35789e3c8df91a510f198b16e1129
.git/objects/25/25a02051c063f2ade525cda2eafc259f71e53c
.git/objects/25/3a3e88518e606c1187e9b8a9cf8243c0c45280
.git/objects/25/d451b1f647ea9a5bbfed9bfd8edf3b237df0d0

```

git有四个常用的对象（或者称为组件）概念：

> tag    
`commit`    
`tree`     
`blob`     

首先我们可以通过`find .git/objects -type f`命令查看git有哪些git对象。

git是通过后三个组件管理着GIT的所有版本文件。
## commit对象
commit顾名思义，每次commit提交都会生成一个commit对象。所以每一个commit对象就是记录git的每一次变更。

用通俗的话来说就是，`commit对象就是当前项目所有目录及文件的一个快照。`

## tree对象和blob对象

{{% center %}}<img name="touchbar-config" src="/images/blog/2019-08/git.png" width='500px'/>{{% /center %}}

Git 以一种类似 UNIX 文件系统但更简单的方式来存储内容。所有内容以 tree 或 blob 对象存储，其中 tree 对象对应于 UNIX 中的目录，
blob 对象则大致对应于 inodes 或文件内容。一个单独的 tree 对象包含一条或多条 tree 记录，每一条记录含有一个指向 blob 或子 tree 对象的 SHA-1 指针，
并附有该对象的权限模式 (mode)、类型和文件名信息。

为了方便记忆，我们可以这么认为：

>`tree 对象，通俗的说就是目录和文件名。`     

>`blob对象，通俗的说就是文件内容。`

值得注意的是，**对于blob对象来说，只要文件内容相同，git就认为它是同一个blob对象，跟文件名无关。blob的设计大大节约了存储空间。**

## git cat-file
我们可以通过`git cat-file`命令来在命令行查看整个git对象存储结构。

首先我们用`git log`找一个commit对象。
```git
➜  xp329486175.github.io git:(master) git log  
commit 5c29befea1c4dc544583666520a0ca35b5151ae5
Author: xiepeng <xiepeng@mama.cn>
Date:   Wed Jul 31 21:01:51 2019 +0800

    master
    git简析
```

然后我们`用git cat-file`命令看这个 commit 对象下面的 tree 对象。
```git
➜  xp329486175.github.io git:(master) git cat-file -p 5c29befea1         
tree 12fe1d356036efa3f9e7ac98f369624b42d588dc
parent 036276adc3ce8a43f47ebdba19936fa6756dbf51
author xiepeng <xiepeng@mama.cn> 1564578111 +0800
committer xiepeng <xiepeng@mama.cn> 1564578111 +0800

master
git简析

```

发现有一个 tree 对象，再查看 tree 对象下面有哪些 blob 对象和子tree 对象。
```git
➜  xp329486175.github.io git:(master) git cat-file -p 12fe1d35603         
100644 blob 62c893550adb53d3a8fc29a1584ff831cb829062    .gitignore
100644 blob d102583b2b303f2c77136813d59953e9293c6960    404.html
100644 blob 1c7ad93cb45e7d8dd3da16a2a1fb03b1b5834d32    README.md
040000 tree 6a0a8db36e0cda43ad6288c25526150b5192b3c0    about
040000 tree 81a199cd46c304fbc3143f07e640e8c8a5adf5f5    blog
040000 tree a37946098d20f0c73d22be32f3844a6e52a85756    categories
040000 tree f46510fc4e8d58647508f305a39c389848d0e1a7    css
040000 tree f98f783e46621ffa563de2ff23075d3d8e532498    fonts
040000 tree 3546860d55b5c37a10d30a6a6275b0e4193173e0    images
040000 tree 0f32769c70b6f5a6b003dd9e33e80c623c17f914    img
100644 blob 818f928c4d5b7f3b67a7cd59b075c4c4cbd85d03    index.html
100644 blob 1a9f5c16b9360dd958912221838fa64416f6baf0    index.xml
040000 tree f7e78a875c449b7ccf31e1eccf07e3b163145f81    js
040000 tree e50e6fcc6868a06d589ec665c915f1b539665c40    moment
100755 blob 79824580b8c7ed0ef56829c1fdef2cc9665e46f2    robots.txt
040000 tree 2d8cfab567bf61c238cd1ea7c61ccd68b18e7c9c    series
100644 blob 41091bc6099f94280081a4ecd1d3343abf0e3a25    sitemap.xml
040000 tree 954d5b2dc0a1336d0870360a675851f595eb2f02    tags
```

有多个 blob 对象和多个 tree 对象。再看一个blob对象，这里我们看 README.md 对应的 blob 对象。
```git
➜  xp329486175.github.io git:(master) git cat-file -p 1c7ad93cb          
# xp329486175.github.io
xpblog        
```

可以看的 blob 对象就是 README.md 文件里面的内容。