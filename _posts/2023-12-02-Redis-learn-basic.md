---
layout: post
title: Redis 学习笔记 - 基础篇
categories: [Learn]
description: 黑马程序员 Redis 教程学习笔记
keywords: Redis, Learn
---

> 课程链接：[blibili](https://www.bilibili.com/video/BV1cr4y1671t?)

## Redis 介绍

### 是什么？

- 基于内存的 K / V 存储中间件
- NoSQL 键值对数据库

Redis 不仅仅是数据库，它还能作为消息队列等

### SQL 和 NoSQL 数据库对比

注意使用场景的差异

![](/images/blog/redis/basic/SQL%20vs%20NoSQL.png)

### Redis 特征

1. 支持多种数据结构
2. 单线程，每个命令的执行具备原子性，中途不会执行其他命令（指命令的处理始终是单线程的，自 6.x 起改为多线程接受网络请求）
3. 高性能，低延时（基于内存，IO 多路复用，良好编码）
4. 支持数据持久化
5. 支持主从、分片集群
6. 支持多语言客户端
7. 默认 16 个数据库，不能配置名字，只能配置数量

## Redis 安装

基于 Mac Homebrew 安装的命令为：

```shell
brew install redis
```

核心配置

配置文件的位置在 `/opt/homebrew/etc/redis.conf`

```shell
# 允许任意 IP 访问
bind 0.0.0.0

# 后台启动 Redis 服务
daemonize yes

# 密码设置
requirepass xxx
```

启动 Redis 服务

```shell
# 借助 Homebrew 启动 Redis 服务
brew services start redis

redis-server
```

查看 Redis 有没有成功启动

```shell
ps -ef | grep redis
```

关闭 Redis 服务

```shell
brew services stop redis
```

### 客户端连接 Redis

通过命令行

```shell
# 1. 进入交互页面
redis-cli

# 2. 输入密码
auth <password>

# 3. 验证是否连接，如果成功连接会返回 PONG
ping
```

### 常用命令

命令集网站：http://www.redis.cn/commands.html

通用命令

- set key value
- get key
- keys pattern 模糊搜索多个 key，性能差（尤其主节点），生产环境不建议用
- del key
- exists key 判断 key 是否存在
- expire key 设置过期时间，到期时候 ttl key 会返回 -2，同时该 key 会被删除
- ttl key 查询剩余存活时间，未设置过期时间则为 -1 代表永久有效

## Redis 基本数据结构

![](/images/blog/redis/basic/type.png)

### String 类型

支持存储字符串、数字、浮点数（实际存储都是字节数组）

![](/images/blog/redis/basic/string%20format.png)

单 key 的 value 最大不能超过 512 MB

![](/images/blog/redis/basic/common_command.png)

实际使用时，通常用冒号连接多个词来拼接 key，比如 `[项目名]:[业务名]:[类型]:id`，在某些 GUI 工具中，会自动根据冒号来划分层级，浏览更方便。这个格式也并非固定，可以根据自己的需求来删除或添加词条。

如果 Value 是一个 Java 对象，例如一个 User 对象，则可以将对象序列化为 JSON 字符串后存储。

### Hash 类型

其 Value 是一个无序字典，类似于 Java 中的 HashMap 结构

![](/images/blog/redis/basic/Hash-type.png)

常用命令

> 其实就是在 String 命令名的基础上增加了 H 首字母

![](/images/blog/redis/basic/Hash-type-command.png)

### List 类型

理解为 Java 的 LinkedList 双向链表，特点是有序、元素可以重复、插入删除快、查找性能一般

常用命令

> 有点像操作一个双端队列

![](/images/blog/redis/basic/List-type-command.png)

### Set 类型

与 Java 中的 HashSet 类似，可以看做是一个 value 为 null 的 HashMap，因为也是一个哈希表，因此具备与 HashSet 类似的特征：

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

常用命令

![](/images/blog/redis/basic/Set-type-command.png)

### SortedSet 类型

可排序集合，与 Java 中的 TreeSet 有些相似，但底层数据结构差别很大。SortedSet 中的每一个元素都带有一个 score 属性，可以基于 score 属性对元素排序，底层的实现是一个跳表（SkipList）加哈希表。SortedSet 具有如下特性：

- 可排序
- 元素不重复
- 查询速度快

常用命令

![](/images/blog/redis/basic/SortedSet-type-command.png)

## Redis 客户端

### 客户端对比

可以在该网站查看所有客户端：https://redis.io/docs/connect/clients/

对于 Java，推荐 `Jedis`，`Lettuce`，`Redisson` 三种客户端

![](/images/blog/redis/basic/java-client-compare.png)

Spring Data Redis 同时兼容 Jedis 和 Lettuce

### Jedis 快速入门

[代码](https://github.com/226wyj/redis-learn/tree/main/jedis-demo)

基本使用步骤

1. 引入依赖
2. 创建 Jedis 对象，建立连接
3. 使用 Jedis，方法名与 Redis 命令一致
4. 释放资源

### SpringDataRedis

[代码](https://github.com/226wyj/redis-learn/tree/main/springdata-redis-demo)

![](/images/blog/redis/basic/spring-data-redis-intro.png)

Spring Data 整合封装了一系列数据访问的操作，Spring Data Redis 则是封装了对 Jedis、Lettuce 这两个 Redis 客户端的操作，提供了统一的 RedisTemplate 来操作 Redis。

RedisTemplate 针对不同的 Redis 数据结构提供了不同的 API，划分更明确

![](/images/blog/redis/basic/redisTemplate.png)

使用步骤

1. 引入 spring-boot-starter-data-redis 依赖
2. 在 application.yaml 配置 Redis 信息
3. 注入 RedisTemplate

**RedisTemplate 序列化**

RedisTemplate 默认使用 JDK 原生序列化器，可读性差、内存占用大，因此可以用以下两种方式来改变序列化机制：

1. 自定义 RedisTemplate，指定 key 和 value 的序列化器
2. (推荐) 使用自带的 StringRedisTemplate, key 和 value 都默认使用 String 序列化器，仅支持写入 String 类型的 key 和 value，因此需要自己将对象序列化成 String 来写入 Redis，从 Redis 读取数据时也要手动反序列化
