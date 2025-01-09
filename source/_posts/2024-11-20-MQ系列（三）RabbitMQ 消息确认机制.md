---
title: MQ系列（三）| RabbitMQ 消息确认机制
tags:
  - 消息队列
  - 架构
categories:
  - 消息队列
  - 架构
cover: https://s2.loli.net/2024/12/18/dSrsLijuqOReo1X.jpg
abbrlink: 38624
date: 2024-11-20 14:22:03
---

## 	RabbitMQ 消息确认机制

> :heavy_exclamation_mark::heavy_exclamation_mark::heavy_exclamation_mark:温馨提示：基于`JDK17`、`SpringBoot 2.1.8.RELEASE` 版本，由于`RabbitMQ` 在 `SpringBoot3+` 的配置项有所不同， 所以请严格按照该本版来使用，挖一坑：【后续会出一个`SpringBoot3+`版本的配置相关教程】

## 架构

![RabbitMQ 消息确认机制-可靠抵达.drawio](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528538.svg) 

##　概念

保证消息不丢失，可靠抵达，可以使用事务消息，性能下降250倍 为此引入**确认机制**

- 生产者确认回调：`publisher` `confirmCallback`
- 生产者退回回调：`publisher` `returnCallback`未投递到`queue`退回模式
- 消费者确认：`consumer` `ack`确认机制

### `ComfirmCallback`【生产者确认回调】

- **概念：**`ComfirmCallback`是生产者消息确认机制的一部分。当生产者发送消息到 `RabbitMQ` 的交换器（`Exchange`）后，`RabbitMQ` 会返回一个确认消息给生产者，这个确认过程可以通过 `ConfirmCallback` 来处理。
- **原理：**生产者发送消息时，会为每条消息关联一个 `CorrelationData` 对象，这个对象可以包含一些自定义的信息，用于跟踪消息。当消息成功发送到交换器后，`RabbitMQ` 会触发 `ConfirmCallback` 接口中的【 `confirm`】 方法。

### `ReturnCallback`【生产者退回回调】

- **概念：**`ReturnCallback` 用于处理消息**无法被正确路由**到队列的情况。当生产者发送消息到交换器后，如果交换器无法将消息路由到任何队列（例如，没有匹配的绑定规则或者队列不存在），消息会被退回给生产者，这个退回过程可以通过 `ReturnCallback` 来处理。
- **原理：**生产者需要配置消息退回机制，并且实现 `ReturnCallback` 接口。当消息被退回时，`ReturnCallback` 接口中的 【`returnedMessage`】 方法会被触发。

### `BasicAck`【消费者确认】

- **概念：** `BasicAck`是消费者确认消息的一种方式。在 `RabbitMQ` 中，消费者接收到消息后，需要向 `RabbitMQ` 服务器确认消息已经被正确处理，这样 `RabbitMQ` 才会从队列中删除该消息。`BasicAck` 是手动确认模式下用于确认消息的方法之一。
- **原理：**消费者在手动确认模式下，从队列中接收消息并进行处理。当处理完成且没有出现问题时，消费者可以使用 Channel 对象的`basicAck`方法来确认消息。`basicAck`方法需要传入两个参数：`deliveryTag`和`multiple`。`deliveryTag`是消息的唯一标识，由 RabbitMQ 服务器分配；`multiple`是一个布尔值，用于表示是否确认多条消息。

##  生产者确认回调 `ConfirmCallback`

### 添加配置

```properties
# 开启生产者消息确认机制
spring.rabbitmq.publisher-confirms=true
```

### 添加 `RabbitMQConfig`

**自定义 `confirmCallback#confirm`** 

- `CorrelationData`：当前消息唯一关联数据【消息的唯一Id】
- `ack`：是否成功收到状态
- `cause`：失败原因

```java
@Configuration
@Slf4j
public class RabbitMQConfig {
     @Autowired
     RabbitTemplate rabbitTemplate;
    
     @PostConstruct //创建RabbitMQConfig对象后，执行这个方法
     public void initRabbitTemplate() {
        //设置确认回调
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
        /**
         * @param correlationData 当前消息的唯一关联数据（这个消息的唯一id）
         * @param ack 消息是否成功收到
         * @param cause  失败的原因
        */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        	log.info("confirm=>correlationData【{}】=>ack【{}】=>cause【{}】 ", correlationData, ack, cause);
        }
    });
}
```

### 测试：生产者确认

```java
@Slf4j
@RestController
public class ProducerController {

    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * 发送消息
     *
     * @param num
     */
    @GetMapping("/send")
    public void sendMessage(@RequestParam("num") int num) {
        for (int i = 0; i < num; i++) {
            if (i % 2 == 0) {
                OrderReturnReasonEntity data = new OrderReturnReasonEntity();
                data.setId(1L).setCreateTime(new Date()).setName("测试-" + i);
                rabbitTemplate.convertAndSend("hello-java-exchange", "hello-java", data);
            } else {
                OrderEntity data = new OrderEntity();
                data.setOrderSn(UUID.randomUUID().toString());
                rabbitTemplate.convertAndSend("hello-java-exchange", "hello-java", data);
            }
        }
        log.info("发送消息: {}条完成", num);
    }
}
```

消息发送成功，生产者确认回调生效，**注意下这里的`correlationData`的数据为`null`**

![image-20241209143121904](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528539.png) 

###　修改下发送信息

`ProducerController`#`sendMessage`中添加当前消息的唯一`id`

```java
rabbitTemplate.convertAndSend("hello-java-exchange", "hello-java", data,new CorralationData(UUID.randomUUID().toString()));
```

- 这里`correlationData.getId()`（也就是`UUID`）可以帮助开发者在多个消息发送场景中，唯一地标识每条消息，从而准确地跟踪某一条特定消息的发送状态，是发送成功还是失败。

###　测试2：消息唯一Id

![image-20241209143421885](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528540.png)

## 生产者回退回调 `ReturnCallback`

`confirm` 模式只能保证消息到达 `broker`，不能保证消息准确投递到目标 `queue` 里，我们需要保证消息一定要投递到目标 `queue` 里，此时就需要用到 
`return` 退回模式。

### 添加配置

```properties
spring.rabbitmq.publisher-returns=true # 开启生产者消息抵达队列的确认
spring.rabbitmq.template.mandatory=true # 只要抵达队列，以异步发送优先回调 return confirm,【发送端确认，默认false】，当交换机无法找到队列时，false【直接丢弃数据】，true【会将消息返回给生产者】
```

###　`RabbitMQConfig` 配置类添加

	只有当前消息不能抵达队列才会触发这个回调

```java
//设置消息抵达队列的确认回调
    rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
        /**
          * 只要消息没有投递给指定的队列，就触发这个失败回调
          * @param message  投递失败的消息详细信息
          * @param replyCode  回复的状态码
          * @param replyText  回复的文本内容
          * @param exchange  当时这个消息发给哪个交换机
          * @param routingKey  当时这个消息用哪个路由键
        */
        @Override
        public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
            log.error("消息发送失败，消息：{}，失败码：{}，失败原因：{}，发送的交换机：{},路由键：{}", message, replyCode, replyText, exchange, replyCode);
	}
});
```

### 修改发送消息的路由键

	`ProducerController`#`sendMessage`发送消息核心代码修改，将其中一个路由键修改成  `hello2-java`（或者修改成没有可绑定的队列即可）

```java
rabbitTemplate.convertAndSend("hello-java-exchange", "hello2-java", data);// 修改这个路由键为 hello2-java
```

### 测试：生产者退回回调

执行发送消息，结果如下

![image-20241209143647892](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528541.png)

- 消息成功到达 `Broker` 服务器，消息确认机制生效，打印 `confirm` 相关信息
- 消息接收失败，生产者回退模式生效，其中 **失败原因**：【`NO_ROUTE`】没有路由到队列，其中**路由键**：【`hello2-java`】，交换机和失败码等信息都打印出来

## 消费者确认：Ack

	消费者收到消息，成功处理发送 `Ack` 给 `Broker`	
	
	消费者收到消息自动确认，但是无法确认消息是否被处理完成或者成功处理，需要手动开启`ack`

### 测试：默认自动 ack

`ProducerController` 添加一个发送消息方法

```java
@GetMapping("sendMQ/{num}")
public void sendMQ(@PathVariable int num) {
    for (int i = 0; i < num; i++) {
        OrderReturnReasonEntity data = new OrderReturnReasonEntity();
        data.setId(1L).setCreateTime(new Date()).setName("测试-");
        rabbitTemplate.convertAndSend("hello-java-exchange", "hello-java", 
                                      data,new CorrelationData(UUID.randomUUID().toString()));
    }
    log.info("发送消息: {}条完成", num);
}
```

发送10条消息

![image-20241209162941438](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528542.png)  

客户端接收到消息，开始处理，处理一条消息完成后，接收下一条消息宕机 

![image-20241209160547226](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528543.png) 

收到消息处理一条完成，队列剩下9条消息

![image-20241209161006108](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528544.png)

![image-20241209163428598](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528546.png)  

此时直接结束服务，代表宕机，队列中的**未确认**的消息自动被确认

![image-20241209163452903](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528547.png) 



###　手动ack ：添加配置

```properties
spring.rabbitmq.listener.simple.acknowleage-mode=maunal # 手动ack消息
```

发送10条消息，收到后模拟宕机，发现消息不会自动确认

![image-20241209164445649](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528548.png) 

![image-20241209164430742](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528549.png) 

宕机后，消息回到准备状态，没有确认 

![image-20241209164704294](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141529496.png)  

### 修改接收消息代码

添加消费者消息确认`ack`

```java
@RabbitHandler
public void receiveOrderReturnReason(Message message, OrderReturnReasonEntity content, Channel channel) {
    log.info("接收到消息：{}", content);
    //消息体
    byte[] body = message.getBody();
    //消息头配置
    MessageProperties messageProperties = message.getMessageProperties();
    log.info("消息处理完成：消息体内容：{}", content.getName());
    //channel内按顺序自增
    long deliveryTag = messageProperties.getDeliveryTag();
    log.info("deliveryTag:{}", deliveryTag);
    //签收获取，非批量模式
    try {
       if (deliveryTag % 2 == 0) {
           channel.basicAck(deliveryTag, false);
           log.info("签收货物：{}", deliveryTag);
       } else {
           // 拒签 requeue=false丢弃 requeue=true 发回服务器，服务器重新入队
           // long deliveryTag, boolean multiple, boolean requeue
           channel.basicNack(deliveryTag, false, true);
           // long deliveryTag, boolean requeue
           // channel.basicReject(deliveryTag, false);
           log.info("拒绝签收货物：{}", deliveryTag);
       }
    } catch (IOException e) {
        //网络中断
    }
}
```

- 消息确认ack，从消息头中获取 `deliveryTag`
- `deliveryTag`：是消息传递标签，它是一个正整数，用于唯一标识一条消息的投递。这个标签主要用于消息确认机制。
  - 消息投递顺序：在通道内【`channel`】内，消息按照顺序被投递，并且【`deliveryTag` 】值是单调递增的
  - 重试机制：可以根据未确认`deliveryTag`重新将消息发送给其他消费者或者在一定时间后重新发送给同一消费者。
- `channel.basicAck(deliveryTag,false)` 手动确认，`false` 非批量
- `channel.basicNack(deliveryTag,false,false)` 拒绝确认
  - `deliveryTag`标识消息的标签， `multiple=false` 非批量，`requeue=false`丢弃( `requeue=true` **发回服务器，服务器重新入队**)
- `channel.basicReject(deliverTag,false)` 拒绝确认，不能批量

### 测试：重新入队`requeue=true`

	发送10条消息，`channel.basicNack(deliveryTag,false,true)` 中 `requeue=true` ，消息重新入队，再次消费

![image-20241209203537538](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528551.png) 

所有消息消费完毕

![image-20241209203737712](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528350.png) 

### 测试：丢弃消息 `requeue=false`

发送10条消息，`channel.basicNack(deliveryTag,false,false)` 中 `requeue=false` ，消息直接丢弃![image-20241209203750543](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528299.png) 

拒绝的消息直接丢弃

![image-20241209203831722](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141528604.png) 