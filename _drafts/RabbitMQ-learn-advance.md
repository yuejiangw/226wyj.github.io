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

默认情况下 RabbitMQ 会把消息存放在内存中，尽管效率高但会有两个问题：

- 一旦 MQ 宕机，内存中的消息会消失
- 内存空间有限，当消费者故障或处理过慢时，会导致消息积压，引发 MQ 阻塞

### 数据持久化

可以通过配置让交换机、队列、以及发送的消息都持久化，这样队列中的消息会持久化到磁盘，MQ 重启消息依然存在

### Lazy Queue

惰性队列，接收到消息后直接存入磁盘而非内存（内存中只保留最近消息，默认 2048 条），消费者要消费消息时才会从磁盘中读取加载到内存，可以支持数百万条的消息存储

要设置一个队列为 Lazy Queue，只需要在声明队列时指定 `x-queue-mode` 为 lazy 即可

```java
// 直接创建 Queue 对象
@Bean
public Queue lazyQueue() {
   return QueueBuilder
            .durable("lazy.queue")
            .lazy() // 开启 lazy 模式
            .build();
}

// 基于 @RabbitListener 注解创建 Queue
@RabbitListener(queuesToDeclare = @Queue(
   name = "lazy.queue",
   durable = "true",
   arguments = @Argument(name = "x-queue-mode", value = "lazy")
))
public void listenLazyQueue(String msg) {
   log.info("接收到 lazy.queue 的消息: {}", msg);
}
```

## 消费者的可靠性

### 消费者确认机制

为了确认消费者是否成功处理消息，RabbitMQ 提供了消费者确认机制（Consumer Acknowledgement），当消费者处理消息结束后应该向 RabbitMQ 发送一个回执，有三种值：

- ack：成功处理消息，MQ 从队列中删除消息
- nack：消息处理失败，MQ 需要再次投递消息
- reject：消息处理失败并拒绝该消息，MQ 从队列中删除消息

什么时候会 reject：消息本身是有问题的，consumer 处理的时候会报错

[](/images/blog/rabbitmq/advance/consumer-ack.png)

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1
          acknoledge-mode: none # none, 关闭 ack; manual, 手动 ack; auto: 自动 ack
```

我们把 ack mode 设置成 `auto` 之后会使可靠性大大增强，只有消费者处理成功之后消息才会被移除，否则消息会被重新投递

### 消费失败处理

如果我们只是把 ack mode 设置成 auto 就不管了，那么消费者出现异常后，消息会不断 requeue 到队列，再重新发送给消费者，然后再次出现异常，从而无限循环。我们可以使用 Spring 的 retry 机制，在消费者出现异常时利用本地重试，而不是无限制 requeue 到 mq 队列

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1
          retry:
            enabled: true # 开启本地重试
            initial-interval: 1000ms # 初始的失败等待时长为 1s
            multiplier: 1 # 下次失败的等待时长倍数，下次等待时长 = multiplier * (last-interval)
            max-attempts: 3 # 最大重试次数
            stateless: true # true 无状态 false 有状态，如果业务中包含事务使用 false
```

重试多次依旧失败则需要有 `MessageRecoverer` 接口来处理

- RejectAndDontRequeueRecoverer: 重试耗尽后直接 reject，默认就是这种方式
- ImmediateRequeueMessageRecoverer: 重试耗尽后返回 nack，消息重新入队
- RepublishMessageRecoverer: 重试耗尽后将失败消息投递到指定的交换机

### 业务幂等性

即业务多次重复处理的结果和只处理一次的结果是相同的

#### 方案一：唯一消息 id

- 给每个消息都设置一个唯一的 id，与消息一起投递给消费者
- 消费者收到消息后处理自己的业务，业务处理成功后将消息 id 保存到数据库
- 如果下次又收到相同消息，去数据库查询判断是否存在，存在则为重复消息放弃处理

```java
@Bean
public MessageConverter messageConverter() {
    // 1. 定义消息转换器
    Jackson2JsonMessageConverter jjmc = new Jackson2JsonMessageConverter();
    // 2. 配置自动创建消息 id，用于识别不同消息，也可以在业务中基于 id 判断是否是重复消息
    jjmc.setCreateMessageIds(true);
    return jjmc
}
```

缺点：对原有业务有侵入，并且增加了写数据库步骤，性能会有影响

#### 方案二：业务判断

结合业务逻辑，基于业务本身做判断

## 延迟消息

