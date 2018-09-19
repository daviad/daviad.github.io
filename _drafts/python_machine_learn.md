---
layout: post
title:  "python machine learn"
---

# 前言

学习笔记，记录
# note

查看安装的库  
pip3 list或者pip freeze  
查看过时的库  
pip3 list --outdated

查看python 帮助文档  
启动python doc server $ pydoc -p 8000 打开浏览器，访问http://localhost:8000即可查看文档，与windows类似。

同时安装 Python 2 和 python 3 解释器（或者 Anaconda 2 和 Anaconda 3），方便版本之间的切换，我们需要为两个版本安装对应的 pip
$ sudo apt-get install python-pip
$ sudo apt-get install python3-pip

运行 pip install仅能为 python 2.x 安装库  
那么如何设置才能实现对 python3 相关库的安装呢？
使用 pip3