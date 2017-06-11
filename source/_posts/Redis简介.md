---
title: Redis数据库
date: 2016-09-08 10:29:03
tags: Redis
categories: 数据库
---

## Redis简介：
**Redis**是一种常用的Nosql（Not Only SQL，非关系型）数据库，遵守BSD协议，是一个高性能的**key-value数据库**，一般用来代替Memcached做缓存服务，同时，它也支持数据的持久化。

更详细的介绍可以查看[redis官网](http://redis.io/)，这里我们主要讲解一下如何安装和使用它。

<!--more-->

Redis相比于其他key-value缓存产品，具有以下特点：

- 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用；
- 不仅支持简单的key-value类型的数据，还提供对于list、set、zset、hash等数据结构的存储；
- 支持数据的备份，即master-slave模式的数据备份。

## Redis的安装和配置：
### 1.安装：
这里我下载的是最新的版本3.2.3：[redis-3.2.3.tar.gz](http://download.redis.io/releases/redis-3.2.3.tar.gz)

下载

### 2.配置：



## Redis的使用操作：
### 1.启动：

### 2.访问：

### 3.关闭：