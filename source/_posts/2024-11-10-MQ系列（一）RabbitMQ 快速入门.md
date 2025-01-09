---
title: MQ系列（一）| RabbitMQ 快速入门
tags:
  - 消息队列
  - 架构
categories:
  - 消息队列
  - 架构
cover: 'https://s2.loli.net/2024/12/18/Zh2tIFuJkaPSTwM.webp'
summary: 消息队列
abbrlink: 46172
date: 2024-11-10 14:22:03
---

# RabbitMQ 快速入门

> 官网：[https://www.rabbitmq.com/](https://www.rabbitmq.com/)
>
> 入门教程：[https://www.rabbitmq.com/tutorials](https://www.rabbitmq.com/tutorials)
>
> 最新版本：`4.0.2`
>
> 版本参考：`JDK17、Maven Or Gradle`

## 1、简介

RabbitMQ是一个可靠且成熟的消息传递和流代理，易于部署在云环境、本地和本地机器上。它目前被全球数百万人使用。
<img src="https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/rabbitmq-logo-with-name.svg" alt="rabbitmq-logo-with-name"  />

## 2、为什么使用

公司业务场景核心：解耦、异步、削峰

### 2.1、解耦

`A`系统发数据给到`BCD`系统，如果`E`系统需要接入？`C`系统不需要了？A系统的负责人就需要来回修改接口对接其他系统。

 ![未解耦](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202203061747025.png)

如果使用`MQ`，`A`系统产生一条数据，发送到`MQ`中，那个系统需要数据自己去`MQ`消费。如果新的系统需要数据，直接从`MQ`中消费；某个系统不需要数据的话，取消消费这个`MQ`即可。这样`A`系统不需要考虑谁发送数据给谁，不需要考虑是否调用成功、失败超时等问题。

![MQ解耦](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202203062104532.png) 

**总结**：通过一个`MQ`，`Pub/Sub`发布订阅消息模型，`A`系统就和其他系统彻底耦合了。

#### 2.2.1、项目应用

 车站系统通过控制命令下发给各个设备，其中车站的设备通常包含：**闸机**、**半自动售票机**、**自动售票机**、**手持机等设备**。

如果按照常规的同步方式来对接不同的设备，这将使得系统冗余的代码很多。当车站增减一个设备就可能需要重新对接接口，造成系统**耦合性很高**，这样的效率不高且不优雅。

所以当系统需要发送命令（**生产一个数据**），将数据放到`MQ`中，不需要知道哪个设备接收消息成功或者失败，其中需要消费的设备自己去订阅并且获取相应的消息即可。这样就可以达到系统下发设备控制命令，不同设备响应。

### 2.2、异步

 `A`系统接收请求，需要本地入库，还需要`BCD`三个系统入库，本地入库（`3ms`），`BCD`（`300ms+400ms+500ms`），用户体验很差等待时间太长。业内请求需要做到 `200ms` 以内，对用户几乎无感。

![异步-1](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202203062204267.png) 

 使用MQ，A系统连续发送3条消息到消息队列，假如消耗5ms，请求花了 5 + 3 = 8ms ，对于用户来说就是点了一个按钮返回很快。

![异步-2](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/img/2022/03/202203062210072.png) 

### 2.3、削峰

 每天一段时间，A系统风平浪静，每秒请求数量就`50`个。结果每次一到 `12:00~13:00`，每秒并发请求数量突然暴增到5k+条。但是系统是直接基于`MySQL`，大量请求涌入`MySQL`，每秒执行约`5k`条`SQL`， 一般情况下`MySQL` 每秒可抗 `2k`请求，`5k`的请求可能打死`MySQL`，导致无法使用。 一旦过了高峰，到了下午就到了低峰期，每秒请求数量 `50` 左右，对整个系统没有多少压力了。

![未削峰](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202203062301225.png) 

如果使用MQ，每秒 5k 请求写入 MQ , A系统每秒最多处理 2k 个请求，因为 MySQL每秒最多请求 2k 个请求。A系统从MQ中慢慢拉取请求，每秒2k个请求，不超过自己每秒最大的请求数量即可。所以再高峰期，A系统不会挂掉。而MQ每秒进 5k ，出 2k，请求就会在高峰期积压可能多大十几万甚至百万的消息再 MQ中。 

![MQ削峰](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202203062328446.png) 

总结：短暂的挤压后是可允许的，等到高峰期过后，每秒进入`MQ`的消息降低很多，但是系统依然按照 2k 的请求取消费，`A`系统很快的就会把挤压解决掉了。

### 2.4、并行

 ![img](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202409252215400.jpg)

### 2.5、排队

![img](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202409252215401.jpg) 

## 3、消息队列工具 RabbitMQ

### 3.1、常见MQ产品

- ActiveMQ：基于`JMS`（`Java Message Service`）协议，`java`语言，`jdk`

- RabbitMQ：基于`AMQP`协议，`erlang`语言开发，稳定性好

- RocketMQ：基于`JMS`，阿里巴巴产品，目前交由`Apache`基金会

- Kafka：分布式消息系统，高吞吐量



### 3.2、RabbitMQ基础概念

![RabbitMQ组件](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/RabbitMQ组件.jpg)		Broker：简单来说就是消息队列服务器实体

	Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列
	
	Queue：消息队列载体，每个消息都会被投入到一个或多个队列
	
	Binding：绑定，它的作用就是把 exchange和 queue按照路由规则绑定起来
	
	Routing Key：路由关键字， exchange根据这个关键字进行消息投递
	
	vhost：虚拟主机，一个 broker里可以开设多个 vhost，用作不同用户的权限分离
	
	producer：消息生产者，就是投递消息的程序
	
	consumer：消息消费者，就是接受消息的程序
	
	channel：消息通道，在客户端的每个连接里，可建立多个 channel，每个 channel代表一个会话任务

### 3.3、五种消息模型

RabbitMQ提供了6种消息模型，但是第6种其实是RPC，并不是MQ，因此不予学习。那么也就剩下5种。

但是其实3、4、5这三种都属于订阅模型，只不过进行路由的方式不同。

![img](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/post/202409252215403.jpg) 

- **基本消息模型**：生产者–>队列–>消费者


- **`work`消息模型**：生产者–>队列–>多个消费者竞争消费


- **订阅模型**-**`Fanout`**：广播模式，将消息交给所有绑定到交换机的队列，每个消费者都会收到同一条消息


- **订阅模型-`Direct`**：定向，把消息交给符合指定 **`rotingKey`** 的队列


- **订阅模型-`Topic` 主题模式**：通配符，把消息交给符合**`routing pattern`（**路由模式） 的队列

## 4、消息不丢失

### 4.1、`MQ`角度

1. 生产者不丢数据

2. MQ服务器不丢数据
3. 消费者不丢数据

保证消息不丢失有两种**实现方式**：

- 开启事务模式 (不推荐)

- 消息息确认模式（生产者，消费者）

说明：开启事务会大幅**降低消息发送及接收效率**，使用的**相对较少**，因此我们生产环境一般都采取消息**确认模式**

### 4.2、消息确认

##### 1、消息持久化

如果希望RabbitMQ重启之后消息不丢失，那么需要对以下3种实体均配置持久化

- **Exchange**
- **Queue**
- **message**

声明exchange时设置持久化（`durable = true`）并且不自动删除 (`autoDelete = false`)

```java
boolean durable = true;
boolean autoDelete = false;
channel.exchangeDeclare("dlx", TOPIC, durable, autoDelete, null)
```

声明queue时设置持久化（`durable = true`）并且不自动删除 (`autoDelete = false`)

```java
boolean durable = true;
boolean autoDelete = false;
channel.queueDeclare("order-summary-queue", durable, false, autoDelete, queueArguments);
```

发送消息时通过设置（`deliveryMode=2`）持久化消息：

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                    .contentType("application/json")
                    .deliveryMode(2)
                    .priority(0)
                    .build();
channel.basicPublish("order", "order.created", false, properties, "sample-data".getBytes())
```

##### 2、发送确认

有时，业务处理成功，消息也发了，但是我们并不知道消息是否成功到达了`rabbitmq`，如果由于网络等原因导致业务成功而消息发送失败，那么发送方将出现不一致的问题，此时可以使用`rabbitmq`的发送确认功能，即要求`rabbitmq`显式告知我们消息是否已成功发送。

##### 3、手动消费确认

有时，消息被正确投递到消费方，但是消费方处理失败，那么便会出现消费方的不一致问题。比如:订单已创建的消息发送到用户积分子系统中用于增加用户积分，但是积分消费方处理却都失败了，用户就会问：我购买了东西为什么积分并没有增加呢？

要解决这个问题，需要引入消费方确认，即只有消息被成功处理之后才告知`rabbitmq`以`ack`，否则告知`rabbitmq`以`nack`