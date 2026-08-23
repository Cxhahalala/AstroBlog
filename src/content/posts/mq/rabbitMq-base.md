---
title: "RabbitMq-基础知识"
published: 2025-07-13
description: "RabbitMQ 快速入门指南：微服务异步通信必要性分析、RabbitMQ 核心概念及 Spring AMQP 快速集成教程。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817163236565.png"
tags: ["RabbitMQ","消息队列","Spring AMQP"]
category: "消息队列"
draft: false
---

# RabbitMQ 快速入门指南

## 1. 为什么需要消息队列 (MQ)？

在现代微服务架构中，服务间的通信至关重要。传统的同步调用（如 Feign）虽然直观，但存在一些问题：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712211710484.png)

- **性能瓶颈**：调用方必须等待服务方响应，链路上的任何延迟都会累积，影响整体性能。
- **耦合度高**：服务之间紧密依赖，一个服务的变更可能需要修改多个相关服务。
- **级联失败**：如果一个服务出现故障，依赖它的所有上游服务都可能失败，导致“雪崩效应”。



**异步消息队列 (MQ)** 提供了一种解决方案。它允许服务之间通过发送和接收消息进行通信，而无需直接相互调用。

**异步调用的优势：**
1.  **服务解耦**：消息的发送者和接收者没有直接依赖，可以独立开发、部署和扩展。
2.  **提升性能**：发送方将消息放入队列后即可返回，无需等待处理结果，提高了系统吞吐量。
3.  **故障隔离**：单个服务的暂时性故障不会影响其他服务，消息会暂存在队列中，待服务恢复后继续处理。
4.  **削峰填谷**：可以平滑处理突发流量，防止系统因瞬时请求过载而崩溃。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213340677.png)

## 2. RabbitMQ 核心概念与安装

### 2.1 MQ 技术选型

| 特性 | RabbitMQ | RocketMQ | Kafka |
| :--- | :--- | :--- | :--- |
| 开发语言 | Erlang | Java | Scala & Java |
| 吞吐量 | 中等 (万级/秒) | 高 (十万级/秒) | 极高 (百万级/秒) |
| 延迟 | 微秒级 | 毫秒级 | 毫秒级 |
| 可靠性 | 高 | 高 | 较高 |
| 核心优势 | 成熟稳定，社区活跃，功能全面 | 为金融场景设计，高可靠 | 高吞吐，流处理生态强大 |

**选择建议**：对于大多数业务场景，**RabbitMQ** 提供了优秀的综合性能、低延迟和高可靠性，是入门和使用的绝佳选择。

### 2.2 使用 Docker 安装

通过 Docker 可以一键安装并运行 RabbitMQ，包含管理后台：

```bash
docker run \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=123456 \
  --name rabbitmq \
  --hostname rabbitmq \
  -p 15672:15672 \
  -p 5672:5672 \
  -d \
  rabbitmq:3.8-management
```

- `5672`: AMQP 协议端口，用于收发消息。
- `15672`: Web 管理后台端口。

安装后，访问 `http://localhost:15672`，使用 `admin/123456` 登录。

windows可以直接安装windows版本的RabbitMQ，安装完成后，启动RabbitMQ服务。
### 2.3 核心概念

- **Publisher**: 消息生产者。
- **Consumer**: 消息消费者。
- **Queue**: 队列，存储消息的缓冲区。
- **Exchange**: 交换机，接收来自生产者的消息，并根据规则将消息路由到一个或多个队列。
- **Binding**: 绑定，定义 Exchange 和 Queue 之间的关联关系。
- **Virtual Host**: 虚拟主机，用于逻辑隔离，不同 vhost 之间的 Exchange、Queue 互不影响。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712211921159.png)
## 3. Spring AMQP 实战

Spring AMQP 为 Spring Boot 应用集成 RabbitMQ 提供了强大的支持。

### 3.1 环境准备

1.  **添加依赖**：在 `pom.xml` 中加入 `spring-boot-starter-amqp`。

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    ```

2.  **配置连接**：在 `application.yml` 中配置 RabbitMQ 连接信息。

    ```yaml
    spring:
      rabbitmq:
        host: localhost
        port: 5672
        virtual-host: /
        username: admin
        password: 123456
    ```

### 3.2 简单队列模式

最简单的模式，一个生产者发送消息到指定队列，一个消费者从中获取消息。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212120586.png)
**生产者 (Publisher)**:
使用 `RabbitTemplate` 发送消息。

**首先在在控制台创建队列**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212222861.png)


```java
@Autowired
private RabbitTemplate rabbitTemplate;

@Test
public void testSimpleQueue() {
    String queueName = "simple.queue";
    String message = "Hello, RabbitMQ!";
    rabbitTemplate.convertAndSend(queueName, message);
}
```

**消费者 (Consumer)**:
使用 `@RabbitListener` 注解监听队列。Spring AMQP 会自动创建队列。

```java
@Component
public class SimpleQueueListener {
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueue(String message) {
        System.out.println("收到消息：" + message);
    }
}
```

### 3.3 工作队列 (Work Queue) 模式

一个队列，多个消费者。一条消息只会被其中一个消费者处理，适用于任务分发。

```java
// 消费者 1
@RabbitListener(queues = "work.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1 处理消息：" + msg);
    Thread.sleep(20); // 模拟耗时
}

// 消费者 2
@RabbitListener(queues = "work.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2 处理消息：" + msg);
    Thread.sleep(200); // 模拟耗时
}
```

默认情况下，RabbitMQ 会平均分配消息。如果消费者处理能力不同，会导致任务积压。可以配置 **“能者多劳”**（公平分发）：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只取1条消息，处理完再取下一条
```

这样，处理快的消费者将承担更多任务，提高整体效率。

## 4. 交换机 (Exchange) 模式

在实际应用中，生产者通常不直接与队列交互，而是将消息发送给交换机，由交换机进行路由。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212358523.png)

### 4.1 Fanout (广播)

将收到的消息广播给所有绑定到该交换机的队列。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212409910.png)

- **场景**：系统通知、配置更新等。

```java
// 生产者
String exchangeName = "my.fanout";
rabbitTemplate.convertAndSend(exchangeName, "", "这是一条广播消息");

// 消费者1
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("fanout.queue1"), // 声明队列
    exchange = @Exchange(name = "my.fanout", type = ExchangeTypes.FANOUT) // 声明交换机并绑定
))
public void listenFanout1(String msg) {
    System.out.println("消费者1 收到广播：" + msg);
}

// 消费者2
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("fanout.queue2"),
    exchange = @Exchange(name = "my.fanout", type = ExchangeTypes.FANOUT)
))
public void listenFanout2(String msg) {
    System.out.println("消费者2 收到广播：" + msg);
}
```

### 4.2 Direct (直连)

根据 `RoutingKey` 精确匹配，将消息发送到 `RoutingKey` 完全一致的队列。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212506723.png)

- **场景**：按日志级别处理、定向任务分发。

```java
// 生产者
String exchangeName = "my.direct";
rabbitTemplate.convertAndSend(exchangeName, "error", "红色警报！");
rabbitTemplate.convertAndSend(exchangeName, "info", "普通信息。");

// 消费者1 (只接收 error 和 info)
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("direct.queue1"),
    exchange = @Exchange(name = "my.direct", type = ExchangeTypes.DIRECT),
    key = {"error", "info"}
))
public void listenDirect1(String msg) {
    System.out.println("消费者1 收到消息：" + msg);
}

// 消费者2 (只接收 error 和 warning)
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("direct.queue2"),
    exchange = @Exchange(name = "my.direct", type = ExchangeTypes.DIRECT),
    key = {"error", "warning"}
))
public void listenDirect2(String msg) {
    System.out.println("消费者2 收到消息：" + msg);
}
```

### 4.3 Topic (主题)

通过通配符进行模糊匹配，路由规则更灵活。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712212619849.png)
- **通配符**:
  - `*`: 匹配一个单词。
  - `#`: 匹配零个或多个单词。
- **场景**：复杂业务分类，如 `region.user.action`。

```java
// 生产者
String exchangeName = "my.topic";
rabbitTemplate.convertAndSend(exchangeName, "china.weather", "中国天气晴朗");
rabbitTemplate.convertAndSend(exchangeName, "japan.news", "日本发布新闻");

// 消费者1 (关心所有中国的消息)
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("topic.queue1"),
    exchange = @Exchange(name = "my.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopic1(String msg) {
    System.out.println("中国频道收到：" + msg);
}

// 消费者2 (关心所有新闻)
@RabbitListener(bindings = @QueueBinding(
    value = @Queue("topic.queue2"),
    exchange = @Exchange(name = "my.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopic2(String msg) {
    System.out.println("新闻频道收到：" + msg);
}
```

## 5. 声明队列和交换机

在之前我们都是基于 RabbitMQ 控制台来创建队列、交换机。但是在实际开发时，队列和交换机是程序员定义的，将来项目上线，又要交给运维去创建。那么程序员就需要把程序中运行的所有队列和交换机都写下来，交给运维。在这个过程中是很容易出现错误的。

因此推荐的做法是由程序启动时检查队列和交换机是否存在，如果不存在自动创建。

### 5.1 基本 API

Spring AMQP 提供了一个 `Queue` 类，用来创建队列：
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213519947.png)

```java
// 创建简单队列
Queue queue = new Queue("queue.name");

// 创建持久化队列（默认）
Queue durableQueue = new Queue("durable.queue", true);

// 创建非持久化队列
Queue nonDurableQueue = new Queue("temp.queue", false);

// 创建自动删除队列
Queue autoDeleteQueue = new Queue("auto.delete.queue", true, false, true);
```

Spring AMQP 还提供了一个 `Exchange` 接口，来表示所有不同类型的交换机：
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213546337.png)

```java
// 直接创建交换机实例
FanoutExchange fanoutExchange = new FanoutExchange("my.fanout");
DirectExchange directExchange = new DirectExchange("my.direct");
TopicExchange topicExchange = new TopicExchange("my.topic");
```

我们可以自己创建队列和交换机，不过 Spring AMQP 还提供了 `ExchangeBuilder` 来简化这个过程：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213600505.png)

```java
// 使用 ExchangeBuilder 创建交换机
FanoutExchange fanout = ExchangeBuilder.fanoutExchange("my.fanout").build();
DirectExchange direct = ExchangeBuilder.directExchange("my.direct").durable(true).build();
TopicExchange topic = ExchangeBuilder.topicExchange("my.topic").durable(false).build();
```

而在绑定队列和交换机时，则需要使用 `BindingBuilder` 来创建 `Binding` 对象：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213614459.png)

```java
// 绑定队列到 Fanout 交换机
Binding binding = BindingBuilder.bind(queue).to(fanoutExchange);

// 绑定队列到 Direct 交换机
Binding binding = BindingBuilder.bind(queue).to(directExchange).with("routing.key");

// 绑定队列到 Topic 交换机
Binding binding = BindingBuilder.bind(queue).to(topicExchange).with("routing.pattern");
```

### 5.2 Fanout 示例

在 `consumer` 中创建一个类，声明队列和交换机：

```java
package com.itheima.consumer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hmall.fanout");
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```

### 5.3 Direct 示例

Direct 模式由于要绑定多个 KEY，会非常麻烦，每一个 Key 都要编写一个 binding：

```java
package com.itheima.consumer.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DirectConfig {

    /**
     * 声明交换机
     * @return Direct类型交换机
     */
    @Bean
    public DirectExchange directExchange(){
        return ExchangeBuilder.directExchange("hmall.direct").build();
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue directQueue1(){
        return new Queue("direct.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithRed(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithBlue(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("blue");
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue directQueue2(){
        return new Queue("direct.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithRed(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithYellow(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("yellow");
    }
}
```

### 5.4 基于注解声明

基于 `@Bean` 的方式声明队列和交换机比较麻烦，Spring 还提供了基于注解方式来声明。

例如，我们同样声明 Direct 模式的交换机和队列：

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}
))
public void listenDirectQueue2(String msg){
    System.out.println("消费者2接收到direct.queue2的消息：【" + msg + "】");
}
```

是不是简单多了！

再试试 Topic 模式：

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```

**注解方式的优势**：
- 代码更简洁，减少了大量的配置类
- 队列、交换机、绑定关系一目了然
- 自动创建所需的队列和交换机
- 推荐在实际项目中使用

## 6. 消息序列化

Spring AMQP 默认使用 JDK 序列化，存在体积大、可读性差、有安全风险等问题。推荐使用 JSON 序列化。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250712213706447.png)

1.  **添加 Jackson 依赖** (如果项目中没有)。

    ```xml
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    ```

2.  **配置 `MessageConverter`**：在任意 `@Configuration` 类中声明一个 Bean。

    ```java
    import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
    import org.springframework.amqp.support.converter.MessageConverter;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class MqConfig {
        @Bean
        public MessageConverter jackson2JsonMessageConverter() {
            return new Jackson2JsonMessageConverter();
        }
    }
    ```

配置后，`RabbitTemplate` 在发送消息时会自动将对象转换为 JSON 字符串，消费者接收时也会自动反序列化为 Java 对象，前提是生产者和消费者两端都有对应的类定义。

**发送对象消息**:
```java
// 假设 User 是一个 POJO
User user = new User("zhangsan", 25);
rabbitTemplate.convertAndSend("object.queue", user);
```

**接收对象消息**:
```java
@RabbitListener(queues = "object.queue")
public void listenObjectQueue(User user) {
    System.out.println("收到用户对象：" + user);
}
```

## 7. 业务改造案例

将支付成功后的订单状态更新，由原来的同步调用改为异步消息通知。

### 7.1 配置 RabbitMQ

在 `application.yml` 中配置 RabbitMQ 连接信息。

```yaml
spring:
  rabbitmq:
    host: 192.168.150.101 # 你的虚拟机IP
    port: 5672 # 端口
    virtual-host: /hmall # 虚拟主机
    username: hmall # 用户名
    password: 123 # 密码
```

### 7.2 接收消息

在 `trade-service` 服务中定义一个消息监听类，监听支付成功消息。

```java
@Component
@RequiredArgsConstructor
public class PayStatusListener {

    private final IOrderService orderService;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "mark.order.pay.queue", durable = "true"),
            exchange = @Exchange(name = "pay.topic", type = ExchangeTypes.TOPIC),
            key = "pay.success"
    ))
    public void listenPaySuccess(Long orderId){
        orderService.markOrderPaySuccess(orderId);
    }
}
```

### 7.3 发送消息

修改 `pay-service` 服务下的 `PayOrderServiceImpl` 类中的 `tryPayOrderByBalance` 方法，在支付成功后发送消息。

```java
private final RabbitTemplate rabbitTemplate;

@Override
@Transactional
public void tryPayOrderByBalance(PayOrderDTO payOrderDTO) {
    // 1.查询支付单
    PayOrder po = getById(payOrderDTO.getId());
    // 2.判断状态
    if(!PayStatus.WAIT_BUYER_PAY.equalsValue(po.getStatus())){
        // 订单不是未支付，状态异常
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 3.尝试扣减余额
    userClient.deductMoney(payOrderDTO.getPw(), po.getAmount());
    // 4.修改支付单状态
    boolean success = markPayOrderSuccess(payOrderDTO.getId(), LocalDateTime.now());
    if (!success) {
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 5.修改订单状态
    // tradeClient.markOrderPaySuccess(po.getBizOrderNo());
    try {
        rabbitTemplate.convertAndSend("pay.topic", "pay.success", po.getBizOrderNo());
    } catch (Exception e) {
        log.error("支付成功的消息发送失败，支付单id：{}， 交易单id：{}", po.getId(), po.getBizOrderNo(), e);
    }
}
```

