---
title: Python Web 1 —— python和MongoDB安装
date: 2016-09-03 15:28:22
tags: Python
categories: python
---

> 做了很长时间的客户端，主要从事过Android软件开发和Unity 3D的游戏开发，之前还看过一段时间的Java Web，但是由于没有实际的应用，所以就搁置了很久。最近突然有对服务器后台编程产生了浓厚的兴趣，想试着用Python + Mongo DB进行游戏后台的开发。

### Python
Python是一门具有强类型(即变量类型是强制要求的)、动态性、隐式类型(不需要做变量声明)、大小写敏感(var和VAR代表了不同的变量)以及面向对象等特点的编程语言。**Python是一种解释型的语言，相比于C语言，Python的运行速度慢，且不能进行加密。**
<!--more-->

#### 一、Python的安装：
由于我使用的开发环境是Mac OS，所以自带了Python，但由于10.10自带的版本是2.7的，所以我们需要重新安装3.x的版本，安装方法：

 1.没有安装Homebrew的先安装此Mac插件，安装方法就是打开终端，输入以下指令：
```ruby
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
2.使用以下指令安装Python3：

- 先查看当前计算机的python版本：**python**

![这里写图片描述](http://img.blog.csdn.net/20160611163033678)
- 安装新版本：**brew install python3**

![这里写图片描述](http://img.blog.csdn.net/20160611163202196)
这里我的安装出错了，出错不可怕，解决就好了，不难看出这个错误出现的原因是对/usr/local/share/man/man3目录的权限不够，那么解决方法：**sudo chown -R linshuhe /usr/local/share/man/man3**
![这里写图片描述](http://img.blog.csdn.net/20160611172025531)
但是，这里又出现了多版本的python共存和版本切换的问题了：
![这里写图片描述](http://img.blog.csdn.net/20160611172635305)
假如不想修改或者删除系统自带的python，我们可以直接通过以下方式指定使用哪个版本执行代码：
![这里写图片描述](http://img.blog.csdn.net/20160611173455348)

除了上述方法之外，还可以直接到官网下载最新版本，直接安装即可。
 
____

### Mongo DB
#### 一、安装
使用Homebrew进行安装是比较简单的安装方式：
1.更新Homebrew:**brew update**
![这里写图片描述](http://img.blog.csdn.net/20160610091546638)
2.开始安装mongodb:**brew install mongodb**
![这里写图片描述](http://img.blog.csdn.net/20160610091527985)
3.根据安装完成最后的提示，启动mongodb:
**mongod —config /usr/local/etc/mongod.conf**

4.下载一个可视化管理工具Robomongo：下载的地址为：
https://robomongo.org/download
![这里写图片描述](http://img.blog.csdn.net/20160610091457232)
____
