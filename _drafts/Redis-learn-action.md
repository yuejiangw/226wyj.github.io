---
layout: post
title: Redis 学习笔记 - 实战篇
categories: [Learn]
description: 黑马程序员 Redis 教程学习笔记
keywords: Redis, Learn
---

> 课程链接：[bilibili](https://www.bilibili.com/video/BV1cr4y1671t?)

项目 - 黑马点评

![](/images/blog/redis/action/intro.png)

项目架构

![](/images/blog/redis/action/architecture.png)

## 共享 Session（单点登录）

如果不使用 Redis，常规流程如下：

(1) 基于 Session 实现短信登录

![](/images/blog/redis/action/session-login-workflow.png)

(2) 基于登录校验拦截器进行验证

![](/images/blog/redis/action/interceptor.png)

**问题**

Session 共享问题：多台 Tomcat 并不共享 session 存储空间，当请求切换到不同的 tomcat 服务器时候导致数据丢失

虽然 Tomcat 支持 session 拷贝，但存在延迟，因此并未广泛使用。Session 的替代方案应满足：

- 数据共享
- 内存存储
- key - value 结构

因此 Redis 就是该问题的解决方案
