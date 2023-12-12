---
layout: post
title: Redis 学习笔记 - 实战篇
categories: [Learn]
description: 黑马程序员 Redis 教程学习笔记
keywords: Redis, Learn
---

> 课程链接：[bilibili](https://www.bilibili.com/video/BV1cr4y1671t?)

# 项目 - 黑马点评

> 项目地址：[Github](https://github.com/226wyj/redis-learn/tree/main/hm-dianping)

![](/images/blog/redis/action/intro.png)

项目架构

![](/images/blog/redis/action/architecture.png)

## 共享 Session（单点登录）

> P24 - P34

如果不使用 Redis，常规流程如下：

(1) 基于 Session 实现短信登录

![](/images/blog/redis/action/login/session-login-workflow.png)

(2) 基于登录校验拦截器进行验证

![](/images/blog/redis/action/login/interceptor.png)

**问题**

Session 共享问题：多台 Tomcat 并不共享 session 存储空间，当请求切换到不同的 tomcat 服务器时候导致数据丢失

![](/images/blog/redis/action/login/session-share-issue.png)

虽然 Tomcat 支持 session 拷贝，但存在延迟，因此并未广泛使用。Session 的替代方案应满足：

- 数据共享
- 内存存储
- key - value 结构

因此 Redis 就是该问题的解决方案

(3) 基于 Redis 实现共享 session 登录

**key 如何设计**
为了安全性，随机生成 token，而不是拼接用户信息，防止恶意伪造或爆破

**选用何种 value 数据结构存放用户信息**

string 还是 hash？有两种方案：

- 先在程序中将对象进行 JSON 序列化，再以 string 类型写入
- 直接以 hash 数据结构写入

因为用户信息是对象，我们这里选择 hash 类型作为 value 的类型，占用的内存更少、且支持对单个字段的增删改查。具体流程如下：

![](/images/blog/redis/action/login/redis-login-workflow.png)

Redis 代替 session 需要考虑的问题：

- 选择合适的数据结构
- 选择合适的 key
- 选择合适的存储粒度

**登录拦截器优化**

之前我们的拦截器只会拦截需要访问用户信息的 URL，如果用户一直在操作无需访问个人信息的 URL 如主页，则拦截器就不会刷新 Redis，从而导致登录失效。解决办法是在登录拦截器前再加一个拦截器，用来拦截一切路径，其作用是负责刷新 token 有效期。

![](/images/blog/redis/action/login/interceptor-optimization.png)

## 缓存

> P35 - P47

**什么是缓存？**

![](/images/blog/redis/action/cache/cache.png)

缓存作用

- 降低后端负载
- 提高读写效率，降低相应时间

缓存成本

- 数据一致性成本
- 代码维护成本
- 运维成本

### 实现

流程

![](/images/blog/redis/action/cache/cache-workflow.png)

缓存更新策略

![](/images/blog/redis/action/cache/cache-update.png)

业务场景

- 低一致性需求：使用内存淘汰机制，例如店铺类型的查询缓存
- 高一致性需求：主动更新，并以超时剔除作为兜底方案，例如店铺详情查询的缓存

**主动更新策略**

Cache Aside 是最常用的方式

- Cache Aside Pattern：由缓存的调用者，在更新数据库的同时更新缓存
- Read / Write Through Pattern：缓存与数据库整合为一个服务，由服务来维护一致性。调用者调用该服务，无需关心缓存一致性问题
- Write Behind Caching Pattern：调用者只操作缓存，由其他线程异步地将缓存数据持久化到数据库，保证最终一致

操作缓存和数据库时有三个问题需要考虑：

1. 删除缓存还是更新缓存
   - 更新缓存：每次更新数据库都更新缓存，无效写操作较多 ❌
   - 删除缓存：更新数据库时让缓存失效，查询时再更新缓存 ✅
2. 如何保证缓存与数据库的操作同时成功 / 失败？
   - 单体系统，将缓存与数据库操作放在一个事务
   - 分布式系统，利用 TCC 等分布式事务
3. 先操作缓存还是先操作数据库？
   - 先删除缓存，再操作数据库
   - 先操作数据库，再删除缓存

关于缓存和数据库的操作顺序，可能存在如下问题：

![](/images/blog/redis/action/cache/cache-aside-problem.png)

上面两种方案中，最终我们选择右边的方案，也就是先操作数据库再删除缓存。因为如果出现图中的问题，线程 2 需要在线程 1 更新数据库的这一段很短的时间内就完成数据库的写操作，这种事情发生的可能性很低，因为写缓存是在内存中，一般速度很快。

为了避免图中的问题真的发生，我们在写缓存的时候可以加上一个超时时间

### 缓存可能遇到的问题

#### 缓存穿透 - Cache Penetration

攻击者可以恶意请求数据库中不存在的数据，从而使得每次查询都要绕过缓存查数据库，增大数据库的压力。

解决方案：

- 缓存空值：缓存一个空值并设置 TTL
- 布隆过滤：在客户端和 Redis 中间加一个布隆过滤器，每次查询前会询问过滤器数据是否存在，如果不存在会直接拒绝请求，阻止继续向下查询

![](/images/blog/redis/action/cache/cache-penetration.png)

预防做法：

- 增强对请求数据的校验，比如 `id > 0`
- 增强对数据格式的控制，比如 id 设置为 10 位，不为 10 位的直接拒绝
- 增强用户权限校验
- 通过限流来保护数据库

#### 缓存雪崩 - Cache Avalanche

大量 `key` 同时失效或者 Redis 宕机导致大量请求访问数据库，带来巨大压力

解决思路：

- 不让 key 同时失效
- 尽量不让 Redis 宕机

![](/images/blog/redis/action/cache/cache-avalanche.png)

#### 缓存击穿 - Hotspot Invalid

也叫热点 key 失效问题

![](/images/blog/redis/action/cache/hotspot-invalid.png)

两种解决方案：

- 互斥锁：只有一个线程会负责缓存重建，其他拿不到锁的线程必须等待
- 逻辑过期

![](/images/blog/redis/action/cache/hotspot-invalid-solution.png)

两种方式都使用了互斥锁来降低缓存重建的开销，方案优缺点对比如下：

![](/images/blog/redis/action/cache/solution-compare.png)

可以将上述两种方式封装成一个缓存工具类：

![](/images/blog/redis/action/cache/cache-tool.png)