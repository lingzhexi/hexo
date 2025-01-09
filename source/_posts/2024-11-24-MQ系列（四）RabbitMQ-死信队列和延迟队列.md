---
title: MQ系列（四）| RabbitMQ 死信队列和延迟队列
tags:
  - 消息队列
  - 架构
categories:
  - 消息队列
  - 架构
cover: https://s2.loli.net/2024/12/18/bENsgfpKC4dyTiG.webp
abbrlink: 40901
date: 2024-11-24 14:22:03

---

# 死信队列

## 死信是什么

死信：无法被消费的消息。由于特定的原因导致队列中的某些消息无法被消费，这些消息没有后续的处理，就会变成死信。当消息在队列中无法被正常消费时，会被发送到死信队列中。

## 死信来源

消息 TTL
队列达到最大长度
消息拒签(`basicNack` 或 `basicReject`)且重入队列为`false`(`requeue=false)`

## 死信架构

![image-20241211220211008](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141840091.png) 

### 消息TTL

| 名称       | 交换机          | 路由键     | 类型   | 特征        | 参数                                                         |
| ---------- | --------------- | ---------- | ------ | ----------- | ------------------------------------------------------------ |
| 普通交换机 | normal_exchange | `zhangsan` | direct | /           | /                                                            |
| 普通队列   | normal_queue    | `zhangsan` | /      | TTL DLX DLK | `x-dead-letter-exchange`：`dead_exchange`<br />`x-dead-letter-routing-key`：`lisi`<br />`x-message-ttl`: `10 * 1000` |
| 死信交换机 | dead_exchange   | `lisi`     | direct | /           | /                                                            |
| 死信队列   | dead_queue      | `lisi`     | /      | /           | /                                                            |

生产者发送消息10条消息到队列 `normal-queue`，消费者C1关闭不去消费，消息存活时间10s到后，10条消息进去死信队列 `dead-queue`，

![image-20241211220303413](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141840525.png) 

消费端`C2`开启，开始消费死信队列里的消息

![image-20241211222528449](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141840940.png) 

 ### 消息被拒

核心代码：在接收消息的地方  

```java
//消息拒签 requeue 设置 false 表示不重入队
channel.basicReject(deliveryTag,false)
//消息不确认，multiple=false 不批量 requeue=false 不重入队
//channel.basicNack(deliveryTag,false,false)
```

发送10条消息到队列

![image-20241211225554204](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841649.png) 

 消费端C2拒绝一条消息

![image-20241211230004126](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841516.png) 

### 队列大小最大长度

将 `x-message-ttl` 属性改成 `x-max-length : 6`，重新启动，发送10个消息，死信队列接收到4个消息

| 名称     | 交换机       | 路由键     | 类型 | 特征        | 参数                                                         |
| -------- | ------------ | ---------- | ---- | ----------- | ------------------------------------------------------------ |
| 普通队列 | normal_queue | `zhangsan` | /    | TTL DLX DLK | `x-dead-letter-exchange`：`dead_exchange`<br />`x-dead-letter-routing-key`：`lisi`<br />`x-max-length`: `6` |

![image-20241211223706816](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841306.png) 

# 延迟队列

## 概念

	延迟队列,队列**内部是有序**的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望 在指定时间到了**以后或之前取出和处**理，简单来说，延时队列就是用来存放需要**在指定时间被处理**的元素的队列。

## 场景

1. 订单在十分钟之内未支付则自动取消  
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。  
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。  
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。 
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

### 分析场景特点：X 事件发生之后之前 X 时间完成 X 任务

定时轮询：可能可以解决问题，但如果短期数据量很多，活动期间甚至百万千万的数据，轮询就不能解决了	

- ☑️账单一周账单未结算的自动结算 

- ❌订单十分钟内未支付则关闭

## RabbitMQ TTL

	最大存活时间 `TTL： Time to Live`，`RabbitMQ` 中**消息**或**队列**的属性，表面消息或队列的**最大存活时间**
	
	如果一条消息设置了 `TTL` 属性或者进入了设置`TTL` 属性的队列，那么这 条消息如果在`TTL` 设置的时间内没有被消费，则会成为"死信"。如果同时配置了队列的TTL 和消息的 `TTL`，那么较小的那个值将会被使用，有两种方式设置 `TTL`。

### 消息 TTL

另一种方式便是针对每条消息设置TTL

 ![image-20241212115827412](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841893.png)

### 队列 TTL

第一种是在创建队列的时候设置队列的 【`x-message-ttl`】属性 

![image-20241211231446238](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841078.png) 

## 整合`SpringBoot`

### 架构：队列TTL

创建两个队列 QA 和 QB，两者队列 TTL 分别设置为 10S 和 40S，然后在创建一个交换机 X 和死信交 换机 Y，它们的类型都是direct，创建一个死信队列 QD，它们的绑定关系

![image-20241211233441601](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841879.png)  

#### 依赖

```xml
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-amqp</artifactId>
     </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
    <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
```

#### 配置

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672 
spring.rabbitmq.username=guest 
spring.rabbitmq.password=guest
```

#### 配置类：`TtlQueueConfig` 

```java
@Configuration
public class TtlQueueConfig {
    //普通交换机和队列
    public static final String X_EXCHANGE = "X";
    public static final String QUEUE_A = "QA";
    public static final String QUEUE_B = "QB";
    //普通路由键
    public static final String XA_ROUTING_KEY = "XA";
    public static final String XB_ROUTING_KEY = "XB";
    //死信交换机和队列
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    public static final String DEAD_LETTER_QUEUE = "QD";
    //死信路由键
    public static final String Y_DEAD_LETTER_ROUTING_KEY = "YD";

    //声明普通交换机 X
    @Bean("xExchange")
    public DirectExchange xExchange() {
        return new DirectExchange(X_EXCHANGE);
    }

    //声明死信交换机 Y
    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    //声明普通队列 A 绑定到对应死信交换机 Y 上
    @Bean("queueA")
    public Queue queueA() {
        Map<String, Object> args = new HashMap<>(3);
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        args.put("x-dead-letter-routing-key", Y_DEAD_LETTER_ROUTING_KEY);
        args.put("x-message-ttl", 10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
    }

    //声明普通队列 B ttl 40s 并绑定到对应死信交换机Y
    @Bean("queueB")
    public Queue queueB() {
        Map<String, Object> args = new HashMap<>(3);
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        args.put("x-dead-letter-routing-key", Y_DEAD_LETTER_ROUTING_KEY);
        args.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
    }

    //声明死信队列QD
    @Bean("queueD")
    public Queue queueD() {
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }

    //声明普通队列 A 绑定到 普通交换机 X
    @Bean
    public Binding queueBindingX(@Qualifier("queueA") Queue queueA,
                                 @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueA).to(xExchange).with(XA_ROUTING_KEY);
    }

    //声明普通队列 B 绑定到 普通交换机 X
    @Bean
    public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueB).to(xExchange).with(XB_ROUTING_KEY);
    }

    @Bean
    public Binding deadLetterBindingY(@Qualifier("queueD") Queue queueD,
                                  @Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(queueD).to(yExchange).with(Y_DEAD_LETTER_ROUTING_KEY);
    }


}
```

#### 生产者：`SendMsgController`

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("sendMsg/{msg}")
    public void sendMsg(@PathVariable String msg) {
        log.info("当前时间：{},发送一条消息给两个TTL队列：{}", new Date(), msg);
        rabbitTemplate.convertAndSend("X", "XA", msg);
        rabbitTemplate.convertAndSend("X", "XB", msg);
    }
}
```

#### 消费者：`DeadLetterQueueConsumer` 

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {
    @RabbitListener(queues = {"QD"})
    public void receiveD(Message message, Channel channel) {
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到死信队列信息：{}", new Date(), msg);
    }
}
```

发起一个请求 http://localhost:8080/ttl/sendMsg/嘻嘻嘻

![image-20241212111944595](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841495.png) 

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息，  然后被消费掉，这样一个延时队列就打造完成了。

### 优化：动态设置`Ttl`的队列

在这里新增了一个队列 QC,绑定关系如下,该队列不设置TTL 时间

![image-20241212112115951](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841653.png)

#### 配置类 `MsgTtlQueueConfig`

```java
@Component 
public class MsgTtlQueueConfig { 
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y"; 
    public static final String QUEUE_C = "QC"; 
 
    //声明队列 C 死信交换机 
    @Bean("queueC") 
    public Queue queueB(){ 
        Map<String, Object> args = new HashMap<>(3); 
        //声明当前队列绑定的死信交换机 
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE); 
        //声明当前队列的死信路由 key 
        args.put("x-dead-letter-routing-key", "YD"); 
        //没有声明 TTL 属性 
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build(); 
    } 
	//声明队列 B 绑定 X 交换机 
    @Bean 
    public Binding queuecBindingX(@Qualifier("queueC") Queue queueC, 
    							  @Qualifier("xExchange") DirectExchange xExchange){ 
   	 	 	return BindingBuilder.bind(queueC).to(xExchange).with("XC"); 
    } 
} 
```

#### 生产者发送消息

核心代码： `correlationData.getMessageProperties().setExpiration(ttlTime);` 

```java
@GetMapping("sendExpirationMsg/{message}/{ttlTime}") 
public void sendMsg(@PathVariable String message,@PathVariable String ttlTime) { 
    rabbitTemplate.convertAndSend("X", "XC", message, correlationData->{ 	
        correlationData.getMessageProperties().setExpiration(ttlTime); 
        return correlationData; 
	}); 
	log.info("当前时间：{},发送一条时长{}毫秒 TTL 信息给队列 C:{}", new Date(),ttlTime, message); 
} 
```

```shell
# 发起请求 
http://localhost:8080/ttl/sendExpirationMsg/你好 1/20000 
http://localhost:8080/ttl/sendExpirationMsg/你好 2/2000
```

![image-20241212112615811](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841093.png) 

**分析**：**RabbitMQ 只会检查第一个消息是否过期**，如果消息过期则丢到死信队列，  如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行

### 安装延时插件

1. 在官网上下载 https://www.rabbitmq.com/community-plugins.html，下载 `rabbitmq_delayed_message_exchange`

2. 将插件复制到RabbitMQ 容器的 /plugins 路径，

   ```
   # 复制插件到容器（rabbitmq）的路径(/plugins)
   docker cp D:/rabbitmq_delayed_message_exchange-4.0.2.ez rabbitmq:/plugins
   ```

3. 执行命令

   ```
   rabbitmq-plugins enable rabbitmq_delayed_message_exchange
   ```

   ![image-20241212113757789](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841999.png) 

4. 插件生效

![image-20241212113809910](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841191.png)

### 优化：延时队列插件 `delayed`

在这里新增了一个队列`delayed.queue`,一个自定义交换机 `delayed.exchange`，绑定关系如下:

![image-20241212113930549](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841798.png)

#### 配置类 `DelayedQueueConfig`

在我们自定义的交换机中，这是一种新的交换类型，该类型**消息**支**持延迟投递**机制 消息传递后并不会立即投递到目标队列中，而是存储在 【**`mnesia`**】(**一个分布式数据系统**)表中，当**达到投递时间**时，**才投递**到目标队列中。

```java
@Configuration
public class DelayedQueueConfig {
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

    @Bean
    public Queue delayedQueue() {
        return new Queue(DELAYED_QUEUE_NAME);
    }

    // 自定义延迟交换机
    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> args = new HashMap<>();
        //自定义交换机的类型
        args.put("x-delayed-type", "direct");
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
    }

    // 绑定延迟队列 到 延迟交换机
    @Bean
    public Binding bindingDelayedQueue(@Qualifier("delayedQueue")Queue delayedQueue,
                                       @Qualifier("delayedExchange") CustomExchange delayedExchange  ) {
        return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
}
```

#### 消息生产者代码

```java
public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange"; 
public static final String DELAYED_ROUTING_KEY = "delayed.routingkey"; 

 @GetMapping("sendDelayMsg/{msg}/{delayTime}")
 public void sendMsg(@PathVariable String msg, @PathVariable Integer delayTime) {
 	rabbitTemplate.convertAndSend("delayed.exchange", "delayed.routingkey", msg, correlationData -> {
 		correlationData.getMessageProperties().setDelay(delayTime);
 		return correlationData;
 	});
 	log.info("当前时间：{},发送一条延迟{}毫秒信息给队列delayed.queue:{}", new Date(), delayTime, msg);
 }
```

#### 消息消费者代码：`DeadLetterQueueConsumer` 添加消费

```java
 @RabbitListener(queues = {"delayed.queue"})
 public void receiveDelayedQueue(Message message) {
 	String msg = new String(message.getBody());
 	log.info("当前时间：{},收到延时队列的消息：{}", new Date(), msg);
 }
```

```shell
# 发起请求： 
http://localhost:8080/ttl/sendDelayMsg/come on baby1/20000 
http://localhost:8080/ttl/sendDelayMsg/come on baby2/2000
```

![image-20241212114815590](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/2024/12/202412141841505.png) 

 第二个消息被先消费掉了，符合预期

## 总结

	延时队列在需要延时处理的场景下非常有用，使用 `RabbitMQ` 来实现延时队列可以很好的利用 `RabbitMQ` 的特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正 确处理的消息不会被丢弃。另外，通过 `RabbitMQ` 集群的特性，可以很好的解决单点故障问题，不会因为 单个节点挂掉导致延时队列不可用或者消息丢失。 