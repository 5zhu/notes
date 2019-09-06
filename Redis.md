---
title: Redis 浅谈
date: 2018-02-03
description:  Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。
categories:
 - Redis
copyright: true
tags: 
 - Redis
photos: 
 - "https://imgchr.com/i/FnbxsI"
---
## Redis

#### 初始redis
> redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。它的性能极高，性能极高 读的速度是110000次/s,写的速度是81000次/s 。

- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

#### Redis的应用场景
- 缓存，提升性能的常用方式
- 队列，Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。
- 数据存储

#### Linux 下安装Redis
	 
	 $ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
	 $ tar xzf redis-2.8.17.tar.gz
	 $ cd redis-2.8.17
	 $ make
     
     //启动
	 $ ./redis-server

#### Redis数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

###### String（字符串）
string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

     redis 127.0.0.1:6379> SET name "chiD"
     OK
     redis 127.0.0.1:6379> GET name
     "chiD"

###### Hash（哈希）
Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

     redis> HMSET myhash name "chiD" age "26"
     "OK"
     redis> HGET myhash name
     "chiD"
     
###### List（列表）
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

     redis 127.0.0.1:6379> lpush mylist java
     (integer) 1
     redis 127.0.0.1:6379> lpush mylist python
     (integer) 2
     redis 127.0.0.1:6379> lpush mylist ruby
     (integer) 3
     
     redis 127.0.0.1:6379> lrange mylist 0 10
        1) "ruby"
        2) "python"
        3) "java"

###### Set（集合）
Redis的Set是string类型的无序集合。
sadd 命令添加一个 string 元素到 key 对应的 set 集合中，成功返回1，如果元素已经在集合中返回 0，如果 key 对应的 set 不存在则返回错误。

      redis 127.0.0.1:6379> sadd myset redis
     (integer) 1
     redis 127.0.0.1:6379> sadd myset java
     (integer) 1
     redis 127.0.0.1:6379> sadd myset java
     (integer) 0
     
###### zset(sorted set：有序集合)
Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。  

     zadd key score member 
     
     