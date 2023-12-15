---
layout: post
title: RabbitMQ 学习笔记 - 基础篇
categories: [Learn]
description: 黑马程序员 RabbitMQ 教程学习笔记
keywords: RabbitMQ, Learn
---

> 课程链接：[bilibili](https://www.bilibili.com/video/BV1mN4y1Z7t9/?vd_source=734a4a3d12292363fc3078169ddd7db2)

## 同步和异步

### 同步调用

优势：

- 时效性强，等待到结果后才返回

问题：

- 拓展性差：各个服务耦合度高，不易拓展
- 性能下降：多个服务需要依次等待上一个服务执行完毕才可以开始运行，性能低
- 级联失败：如果中间有某一个服务宕机的话则后续的所有服务都无法继续运行，并且该停机可能会传播到上级服务中，引发全线崩溃

### 异步调用

![](/images/blog/rabbitmq/basic/async.png)

![](/images/blog/rabbitmq/basic/async-architecture.png)

异步调用的优势和劣势与同步调用刚好是反过来的

优势：

- 耦合度低，拓展性强
- 异步调用，无需等待，性能好
- 故障隔离，下游服务故障不影响上游业务
- 缓存消息，流量削峰填谷

劣势：

- 不能立即得到调用结果，时效性差
- 不确定下游业务是否执行成功
- 业务安全依赖于 Broker （消息代理）的可靠性

## MQ 技术选型

![](/images/blog/rabbitmq/basic/selection.png)

使用场景：

- Kafka：吞吐量很高的情况，比如日志收集
- RabbitMQ：可靠性高，语言不限于 Java，消息延迟低
- RocketMQ：可靠性高，Java 项目

## 数据隔离

## SpringAMQP

## work模式

## MQ消息转换器

## 发布订阅模式

## 消息堆积问题处理
