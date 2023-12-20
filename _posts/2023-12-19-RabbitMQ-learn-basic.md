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

## 认识和安装

### 安装

在 Mac 环境下通过 Homebrew 安装

```shell
brew install rabbitmq
```

启动服务，默认端口是 15672

```shell
brew services start rabbitmq
```

访问管理页面

- url: http://localhost:15672
- username: guest
- password: guest

### 基本介绍

**发送模型**

publisher 生产者发消息给 exchange (交换机)，exchange 负责将消息路由给 queue (队列)，consumer (消费者) 监听队列从而拿到消息

![](/images/blog/rabbitmq/basic/basic-concept.png)

交换机是负责路由和转发消息的，它本身没有存储消息的能力

**快速入门**

创建一个新的 queue

![](/images/blog/rabbitmq/basic/create-queue.png)

exchange 绑定 queue

![](/images/blog/rabbitmq/basic/exchange-bind.png)

Publish message

![](/images/blog/rabbitmq/basic/publish-message.png)

Get message

![](/images/blog/rabbitmq/basic/get-message.png)

## 数据隔离

添加用户

![](/images/blog/rabbitmq/basic/add-user.png)

这里我们添加了一个名为 `hmall` 的管理员账户，尽管它可以看到我们在上一步骤中创建的 `hello.queue1` 和 `hello.queue2` 两个队列，但它却无权访问（get message 会出错）。这是因为上面两个队列是在不同的 virtual host 中创建的，可见 RabbitMQ 通过 Virtual Host 实现了数据隔离。

创建虚拟主机（Virtual host）

![](/images/blog/rabbitmq/basic/add-virtual-host.png)

添加了新的 virtual host 之后我们再次查看 exchanges 页面，会发现这次会多出我们 virtual host 下的 exchange

![](/images/blog/rabbitmq/basic/all-exchange.png)

## work模式

一个队列上绑定多个消费者的模式叫 work 模式，这样做的目的是为了增加处理能力，避免消息堆积

默认情况下，`RabbitMQ` 会将消息依次轮询投递给绑定在队列上的每一个消费者，但这并没有考虑到不同消费者的消费能力的区别，可能会导致消息堆积

**解决方案：**

修改 `application.yml`，设置 `preFetch` 值为 1， 确保同一时刻最多投递给消费者 1 条消息，实现能者多劳

![](/images/blog/rabbitmq/basic/preFetch.png)

## 交换机

交换机的作用是：

- 接收 publisher 发送的消息
- 将消息按照规则路由到与之绑定的队列

### 交换机类型

#### Fanout：广播

Fanout Exchange 会将接收到的消息广播到每一个跟其绑定的 queue，所以也叫广播模式

#### Direct：定向

![](/images/blog/rabbitmq/basic/direct-exchange.png)

#### Topic：话题

![](/images/blog/rabbitmq/basic/topic-exchange.png)

Topic 交换机是最灵活的一种交换机，可以模拟 Direct 和 Fanout，并且可以使用通配符，如果不确定要使用哪种 exchange 则推荐使用 Topic

Direct 交换机与 Topic 交换机的差异

- Topic 交换机中 routing key 可以是多个单词，以 `.` 分割
- Topic 交换机与队列绑定时的 binding key 可以指定通配符
- `#` 代表 0 个或多个词
- `*` 代表 1 个词

## SpringAMQP

#### AMQP & Spring AMQP

**AMQP**：Advanced Message Queuing Protocol, 用来在应用程序之间传递业务消息的开放标准，该协议与平台语言无关

**Spring AMQP**：基于 AMQP 协议定义的一套 API 规范，提供了模板来发送和接收消息，包含两部分，其中 `spring-amqp` 是基础抽象，`spring-rabbit` 是底层的默认实现

#### 快速入门

项目地址：[Github](https://github.com/226wyj/rabbitmq-learn/tree/main/mq-demo)

使用步骤：

- 引入 `spring-boot-starter-amqp` 依赖
- 配置 `rabbitmq` 服务端信息
- 利用 `RabbitTemplate` 发送消息
- 利用 `@RabbitListener` 注解声明要监听的队列，监听消息

注意：对于 RabbitMQ，控制台页面的端口是 15672，但正常队列的端口是 5672

#### 通过 SpringAMQP 配置 RabbitMQ

常用类：

![](/images/blog/rabbitmq/basic/queue+exchange.png)

基于 `@Configuration` 注解和 `@Bean` 注解声明和配置 Exchange，以 Fanout 为例：

![](/images/blog/rabbitmq/basic/exchange-sample.png)

推荐使用基于 `@RabbitListener` 注解的方式来进行配置而不是使用 `@Bean` 进行依赖注入，后者在配置 Direct Exchange 的时候对于每个 Bean 每次只能指定一个 key，如果一个 Queue 要是想绑定多个 key 的话就要声明多个 Bean，比较麻烦

![](/images/blog/rabbitmq/basic/annotation-based-exchange-config.png)

## MQ消息转换器

Spring 中默认使用 JDK 自带的 ObjectOutputStream 进行序列化，因此如果我们想给 MQ 发送一个 Object，则会被 convert 成一个字节码序列

![](/images/blog/rabbitmq/basic/default-converter.png)

解决：使用 JSON 序列化替代默认的 JDK 序列化

![](/images/blog/rabbitmq/basic/json-converter.png)