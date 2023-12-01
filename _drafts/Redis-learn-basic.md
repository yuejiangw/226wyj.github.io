---
layout: post
title: Redis 学习笔记 - 基础篇
categories: [Learn]
description: 黑马程序员 Redis 教程学习笔记
keywords: Redis, Learn
---

> 课程：https://www.bilibili.com/video/BV1cr4y1671t?

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
