---
layout: post
title: RabbitMQ 学习笔记 - 高级篇
categories: [Learn]
description: 黑马程序员 RabbitMQ 教程学习笔记
keywords: RabbitMQ, Learn
---

> 课程链接：[bilibili](https://www.bilibili.com/video/BV1mN4y1Z7t9/?vd_source=734a4a3d12292363fc3078169ddd7db2)
> P16 - P31

## 发送者的可靠性

**生产者重连**

[](/images/blog/rabbitmq/advance/producer-retry.png)

**生产者确认**

[](/images/blog/rabbitmq/advance/producer-confirm.png)

如何处理生产者的确认消息？

* 尽管有四种情况，但我们真正需要处理的只有 NACK 这一种，即消息 publish 失败
* 生产者确认需要额外的网络和系统资源开销，尽量不要使用，如果一定要使用则无需开启 `Publisher-Return` 机制，只开启 `Publisher-Confirm` 即可，因为一般路由失败是代码问题
* 对于 nack 消息可以有限次数重试，依然失败则记录异常信息

## MQ 的可靠性

## 消费者的可靠性

## 延迟消息

