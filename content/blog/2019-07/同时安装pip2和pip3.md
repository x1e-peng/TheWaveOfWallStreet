---
date: "2019-07-01T20:21:48+08:00"
draft: false
title: "mac下python2和python3同时安装pip2和pip3"
tags: ["python"]
categories: ["python"]
toc: true
---

## 序
今天使用pip安装gearman时，遇到一个坑，记录一下。

安装时，提示报错，然后发现时python版本问题。pip对应的python版本是python3.7。
```shell
➜  member_avatar git:(master) pip -V                                 
pip 19.1.1 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)

```
因此，我需要将我的python2，也安装一个对应的pip。然后将pip3指向python3。

## 安装pip2和pip3
由于mac虽然自带了python，但是却没有自带pip。所以我们需要用mac里面python自带easy_install.py来安装。
```shell
sudo easy_install pip
```
这时会将pip优先指向系统默认的python版本，也就是python2，此时：
```shell
➜  member_avatar git:(master) pip -V                                 
pip 19.1.1 from /Library/Python/2.7/site-packages/pip-19.1.1-py2.7.egg/pip (python 2.7)
```
这就是我们想要的pip了。然后因为从Python3.4开始，内置了pip包管理器，不需要单独安装pip3。所以pip3也有了。
```shell
➜  member_avatar git:(master) pip3 -V
pip 19.1.1 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)
```