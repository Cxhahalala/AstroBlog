---
title: "Redis消息队列"
published: 2025-06-11
description: "Redis 消息队列全景对比：List、PubSub 与 Stream 三大方案详解，重点解析 Stream 消费者组与 ACK 确认机制。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817162903558.png"
tags: ["redis","java","消息队列"]
category: "Redis"
author: "Cx"
draft: false
---

Redis消息队列详解：介绍了Redis实现消息队列的三种方式 - List、PubSub和Stream。重点讲解了Stream的ACK确认机制、消费者组模式，以及如何通过Java代码实现通用的异步消息处理系统。Stream方式具备消息可回溯、多消费者支持、阻塞读取和消息确认等特性，是生产环境的推荐选择。
<!-- more -->
# Redis消息队列
## 什么是消息队列
什么是消息队列：字面意思就是存放消息的队列。最简单的消息队列模型包括3个角色：

* 消息队列：存储和管理消息，也被称为消息代理（Message Broker）
* 生产者：发送消息到消息队列
* 消费者：从消息队列获取消息并处理消息
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611130958494.png)


使用队列的好处在于**解耦**：以快递配送为例，快递员（生产者）将快递放入快递柜（消息队列），用户（消费者）随时从快递柜取货。这种异步处理方式避免了快递员等待用户在家的时间浪费。

在秒杀场景中：用户下单后，系统快速校验条件并将订单信息发送到消息队列，后台服务异步处理订单，既实现了系统解耦，又提升了响应速度。

Redis 作为消息队列的优势：虽然有 Kafka、RabbitMQ 等专业 MQ 中间件，但如果项目中已使用 Redis，直接利用其消息队列功能可以降低部署复杂度和学习成本。

## 基于List实现的消息队列
**使用List结构模拟消息队列**
消息队列（Message Queue），字面意思就是存放消息的队列。而Redis的list数据结构是一个双向链表，很容易模拟出队列效果。

队列是入口和出口不在一边，因此我们可以利用：`LPUSH` 结合 `RPOP`、或者 `RPUSH` 结合 `LPOP`来实现。
不过要注意的是，当队列中没有消息时`RPOP`或`LPOP`操作会返回null，并不像JVM的阻塞队列那样会阻塞并等待消息。因此这里应该使用`BRPOP`或者`BLPOP`来实现阻塞效果。

基于List的消息队列有哪些优缺点？
优点：

* 利用Redis存储，不受限于JVM内存上限
* 基于Redis的持久化机制，数据安全性有保证
* 可以满足消息有序性

缺点：

* 无法避免消息丢失
* 只支持单消费者

## 基于PubSub的消息队列

PubSub（发布订阅）是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。

`SUBSCRIBE channel [channel]` ：订阅一个或多个频道
`PUBLISH channel msg` ：向一个频道发送消息
`PSUBSCRIBE pattern[pattern]` ：订阅与pattern格式匹配的所有频道

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131522983.png)

基于PubSub的消息队列有哪些优缺点？
优点：

* 采用发布订阅模型，支持多生产、多消费

缺点：

* 不支持数据持久化
* 无法避免消息丢失
* 消息堆积有上限，超出时数据丢失

## 基于Stream的消息队列

Stream 是 Redis 5.0 引入的一种新数据类型，可以实现一个功能非常完善的消息队列。

发送消息的命令：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131629420.png)

例如

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131707823.png)

读取消息的方式之一：XREAD

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131737856.png)

例如，使用XREAD读取第一个消息：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131806935.png)

XREAD阻塞方式，读取最新的消息：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131909342.png)


在业务开发中，我们可以循环调用 `XREAD` 阻塞方式来查询最新消息，从而实现持续监听队列的效果，伪代码如下

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611131931997.png)

注意：当我们指定起始ID为 `$` 时，代表读取最新的消息，如果我们处理一条消息的过程中，又有超过1条以上的消息到达队列，则下次获取时也只能获取到最新的一条，会出现漏读消息的问题

STREAM类型消息队列的 `XREAD` 命令特点：

* 消息可回溯
* 一个消息可以被多个消费者读取
* 可以阻塞读取
* 有消息漏读的风险

## 基于Stream的消息队列-消费者组

消费者组（Consumer Group）：将多个消费者划分到一个组中，监听同一个队列。具备下列特点：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611132105316.png)

创建消费者组

`XGROUP CREATE key groupName ID [MKSTREAM]`

key：队列名称
groupName：消费者组名称
ID：起始ID标示，$代表队列中最后一个消息，0则代表队列中第一个消息
MKSTREAM：队列不存在时自动创建队列
其它常见命令：

 **删除指定队列的消费者组**

 `XGROUP DESTROY key groupName`

 **删除指定队列消费者组中的消费者**
 
 `XGROUP DELCONSUMER key groupname consumername`

**从消费者组读取消息**

`XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]`


* group：消费组名称
* consumer：消费者名称，如果消费者不存在，会自动创建一个消费者
* count：本次查询的最大数量
* BLOCK milliseconds：当没有消息时最长等待时间
* NOACK：无需手动ACK，获取到消息后自动确认
* STREAMS key：指定队列名称
* ID：获取消息的起始ID：

">"：从下一个未消费的消息开始
其它：根据指定id从pending-list中获取已消费但未确认的消息，例如0，是从pending-list中的第一个消息开始

消费者监听消息的基本思路：

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611132607947.png)

**XREADGROUP 命令特点：**

* 消息可回溯
* 可以多消费者争抢消息，加快消费速度
* 可以阻塞读取
* 没有消息漏读的风险
* 有消息确认机制，保证消息至少被消费一次

**对比**

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250611132643105.png)


# Redis的Stream消息队列实现异步操作

## 发送消息到Stream队列

```java
import lombok.extern.slf4j.Slf4j; // 新增导入
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.stream.MapRecord;
import org.springframework.data.redis.connection.stream.RecordId;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;
// com.alibaba.fastjson.JSON 如果需要，请确保此依赖存在
// import com.alibaba.fastjson.JSON; 

@Service
@Slf4j // 新增注解
public class RedisStreamProducer {
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    /**
     * 发送消息到Stream队列
     * @param streamKey Stream队列的key
     * @param messageData 消息数据
     * @return 消息ID
     */
    public String sendMessage(String streamKey, Map<String, Object> messageData) {
        try {
            // 将消息发送到Stream队列
            RecordId recordId = stringRedisTemplate.opsForStream()
                    .add(streamKey, messageData);
            log.info("消息发送成功，消息ID: {}", recordId.getValue());
            return recordId.getValue();
        } catch (Exception e) {
            log.error("发送消息失败: {}", e.getMessage());
            throw new RuntimeException("消息发送失败", e);
        }
    }
    
    /**
     * 发送业务消息的示例
     */
    public void sendBusinessMessage(Object businessData) {
        Map<String, Object> messageMap = new HashMap<>();
        messageMap.put("businessId", businessData.getId());
        messageMap.put("businessType", businessData.getClass().getSimpleName());
        messageMap.put("data", JSON.toJSONString(businessData));
        messageMap.put("timestamp", System.currentTimeMillis());
        
        sendMessage("stream:business-queue", messageMap);
    }
}
```

## 消费消息的通用实现

```java
@Component
@Slf4j
public class RedisStreamConsumer implements Runnable {
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    // 配置信息
    private static final String STREAM_KEY = "stream:business-queue";
    private static final String GROUP_NAME = "business-group";
    private static final String CONSUMER_NAME = "consumer-1";
    
    @PostConstruct
    public void init() {
        // 启动时创建消费者组（如果不存在）
        try {
            stringRedisTemplate.opsForStream()
                    .createGroup(STREAM_KEY, GROUP_NAME, "0");
        } catch (Exception e) {
            // 消费者组已存在，忽略异常
            log.info("消费者组可能已存在: {}", e.getMessage());
        }
        
        // 启动消费者线程
        new Thread(this).start();
    }
    
    @Override
    public void run() {
        while (true) {
            try {
                // 从Stream中获取消息
                List<MapRecord<String, Object, Object>> messages = stringRedisTemplate.opsForStream()
                        .read(Consumer.from(GROUP_NAME, CONSUMER_NAME),
                              StreamReadOptions.empty().count(1).block(Duration.ofSeconds(2)),
                              StreamOffset.create(STREAM_KEY, ReadOffset.lastConsumed()));
                
                // 没有获取到消息，继续循环
                if (messages == null || messages.isEmpty()) {
                    continue;
                }
                
                // 处理消息
                for (MapRecord<String, Object, Object> message : messages) {
                    processMessage(message);
                }
                
            } catch (Exception e) {
                log.error("处理消息异常", e);
                handlePendingList();
            }
        }
    }
    
    /**
     * 处理单个消息
     */
    private void processMessage(MapRecord<String, Object, Object> message) {
        try {
            Map<Object, Object> messageData = message.getValue();
            
            // 提取消息内容
            String businessId = (String) messageData.get("businessId");
            String businessType = (String) messageData.get("businessType");
            String data = (String) messageData.get("data");
            
            log.info("开始处理消息 - ID: {}, 业务类型: {}", businessId, businessType);
            
            // 执行具体的业务逻辑（这里可以根据businessType分发到不同的处理器）
            handleBusinessLogic(businessType, data);
            
            // 手动确认消息
            stringRedisTemplate.opsForStream()
                    .acknowledge(STREAM_KEY, GROUP_NAME, message.getId());
            
            log.info("消息处理完成 - ID: {}", businessId);
            
        } catch (Exception e) {
            log.error("处理消息失败: {}", e.getMessage(), e);
            // 这里可以根据业务需要决定是否重试或丢弃消息
        }
    }
    
    /**
     * 执行具体的业务逻辑
     * 这里可以根据业务类型分发到不同的处理器
     */
    private void handleBusinessLogic(String businessType, String data) {
        switch (businessType) {
            case "Order":
                handleOrderLogic(data);
                break;
            case "Payment":
                handlePaymentLogic(data);
                break;
            case "Notification":
                handleNotificationLogic(data);
                break;
            default:
                log.warn("未知的业务类型: {}", businessType);
        }
    }
    
    private void handleOrderLogic(String data) {
        // 订单相关业务逻辑
        log.info("处理订单业务: {}", data);
        // 实际业务处理...
    }
    
    private void handlePaymentLogic(String data) {
        // 支付相关业务逻辑
        log.info("处理支付业务: {}", data);
        // 实际业务处理...
    }
    
    private void handleNotificationLogic(String data) {
        // 通知相关业务逻辑
        log.info("处理通知业务: {}", data);
        // 实际业务处理...
    }
    
    /**
     * 处理Pending List中的消息
     * 当消费者处理消息异常时，处理未确认的消息
     */
    private void handlePendingList() {
        while (true) {
            try {
                // 获取Pending List中的消息
                List<MapRecord<String, Object, Object>> pendingMessages = stringRedisTemplate.opsForStream()
                        .read(Consumer.from(GROUP_NAME, CONSUMER_NAME),
                              StreamReadOptions.empty().count(1),
                              StreamOffset.create(STREAM_KEY, ReadOffset.from("0")));
                
                if (pendingMessages != null && !pendingMessages.isEmpty()) {
                    for (MapRecord<String, Object, Object> message : pendingMessages) {
                        processMessage(message);
                    }
                } else {
                    // 没有更多pending消息，退出循环
                    break;
                }
            } catch (Exception e) {
                log.error("处理Pending List异常", e);
                break;
            }
        }
    }
}
```

## 总结

Redis 提供了多种方式实现消息队列，每种方式都有其适用场景：

1. **基于 List**：适合简单的单消费者场景，实现简单但功能有限。
2. **基于 PubSub**：适合消息广播场景，但不保证消息可靠性。
3. **基于 Stream**：功能最完善，支持消费者组、消息确认、故障恢复等高级特性，是生产环境的推荐选择。

在实际项目中，推荐使用 Stream 方式实现消息队列，它提供了完整的消息队列解决方案，既保证了消息的可靠性，又支持水平扩展。通过合理的代码架构设计，可以构建出既通用又高效的异步消息处理系统。