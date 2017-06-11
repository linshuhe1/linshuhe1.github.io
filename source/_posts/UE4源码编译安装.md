---
title: UE4笔记（1） 源码编译和安装
date: 2017-01-21 10:57:00
tags: UE4
categories: 游戏前端
photos:
- http://img.blog.csdn.net/20170121112115334
---
>序言
近来UE4渐渐有了抬头的趋势，并且因为其在VR游戏设计中表现突出的渲染性能而饱受开发者青睐，更令人惊叹不已的是这个引擎的源码是开源的，这对于希望研究引擎底层代码的码农来说，优秀得有点过分啦。

<!--more-->

---

###一、引擎安装：
关于引擎安装，其实有两种方式：
- 通过官方提供的Laucher下载引擎、工具和资源，具体步骤：登录Epic Games账号后，打开[https://www.unrealengine.com/dashboard](https://www.unrealengine.com/dashboard)，在第一项中选择平台下载安装引擎所需的EpicGamesLauncherInstaller；
![获取引擎](http://img.blog.csdn.net/20170116153109625)
- 下载Github官方源码，然后使用Visual Studio进行编译得到引擎工具。

###二、UE4源码下载：
打开UE4的[官网](https://www.unrealengine.com)链接，注册一个Epic Games的个人账号。登录账号后，参考官方[如何链接您的Github账户以下载虚幻引擎4源代码](https://www.unrealengine.com/zh-CN/ue4-on-github)的相关说明，即可通过github下载完整的Unreal Engine源码。

---

###三、源码编译：
下载源码后，解压到**不包含中文**的本地目录下：
![目录结构](http://img.blog.csdn.net/20170116153304065)
- 编译需要借助Visual Studio编程工具，所以需提前安装2013版或2012版的VS；
- 在解压根目录找到``GenerateProjectFiles.bat``文件，双击此文件即可生成源码项目的``UE4.sh``文件；
- 双击上述生成的.sh文件，会在VS中打开源码，带导入完毕，选中``UE4.sh``，``右键``->``生成``，等待编译完成。
- 生成完毕之后，在当前源码根目录下，打开``Engine\Binaries``，会有适用于不同平台的的文件夹，打开win64，打开``UE4Editor.exe``即可打开UE4编辑器。

---

###四、缺陷说明：
使用源码编译的引擎工具有一点不足，那就是访问MarketPlace时必须通过Laucher，而只要安装并启动了官方的Laucher，则会自动下载最新的引擎工具包（大概7G），那此时机器上就会同时存在两个版本的UE4（官方Laucher下载安装的和自己编译得到的），不过使用官方Laucher在MarketPlace中下载的资源，在自己编译得到的引擎工具中是可以正常使用的。


