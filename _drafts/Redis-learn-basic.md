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

## SQL 和 NoSQL 数据库对比

注意使用场景的差异

![](/images/blog/redis/basic/SQL%20vs%20NoSQL.png)

## Redis 特征

1. 支持多种数据结构
2. 单线程，每个命令的执行具备原子性，中途不会执行其他命令（指命令的处理始终是单线程的，自 6.x 起改为多线程接受网络请求）
3. 高性能，低延时（基于内存，IO 多路复用，良好编码）
4. 支持数据持久化
5. 支持主从、分片集群
6. 支持多语言客户端
