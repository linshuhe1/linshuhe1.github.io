---
title: 《从零开始搭建游戏服务器》redis操作指令和Jedis工具
date: 2017-03-03 10:48:37
tags: 
  - Java
  - Redis
categories: 服务器开发
---

>### 引言
上篇已经大致完成了redis的下载安装和简单的使用，接下来我们要真正地操作redis进行一些数据的增删改查操作。

<!--more-->

### 常用指令：
#### 1.增加或者修改已有数据的值：
若此key对应的value不存在，则创建这个键值对，若已存在，则修改此key的value数值：
```Shell
set key value
```
查询key是否存在：
```Shell
exists key
```
当然还可以设置失效时间：
```Shell
set key time value
```
可以通过指令查询key的存活时间：time to live
```Shell
ttl key
```
假如数值为整数，还可以进行递增（incr）递减（decr）操作：
![](http://img.blog.csdn.net/20170228115717776)

除了操作基本的数据类型，redis数据库还能操作``列表``、``集合``、``哈希表``等复杂的数据结构。

#### 2.查询：
通过key来找对应的value值是redis这种键值对结构数据库最大的优势，检索速度快：
```Shell
get key
```
当然也可以通过key的特征来模糊检索符合条件的key集合：
```Shell
keys pattern
```

#### 3.删
跟查询一致，可以通过key值来删除单条数据：
```Shell
del key
```
![](http://img.blog.csdn.net/20170228113002419)

#### 4.其他指令：
这是一些服务器管理常用的指令：
```Shell
info   #查看服务器信息
select <dbsize> #选择数据库索引  select 1
flushall #清空全部数据
flushdb  #清空当前索引的数据库
slaveof <服务器> <端口>  #设置为从服务器
slaveof no one #设置为主服务器
shutdown  #关闭服务
quit #退出客户端
```

### Redis可视化操作工具：
#### 1.工具简介：
``Redis Desktop Manager``（RedisDesktopManager，RDM）是一个快速、简单、支持跨平台的 Redis 桌面管理工具，基于 Qt 5 开发，支持通过 SSH Tunnel 连接。

#### 2.下载安装：
下载安装包：[Redis Desktop](https://redisdesktop.com/)

#### 3.连接查看：

---
### Jedis工具：
我们在进行系统开发时，通常不通过这种低效率的指令来操作Redis数据库，而是使用封装好的操作Redis数据库的工具，这里要说到的就是Redis官方提供的Jedis，这就是Redis提供的Java API对Redis进行操作。

#### 1.工具下载：
为了在项目中使用Jedis来操作Redis，需要下载对应的客户端开发包，这里我使用的版本是：
