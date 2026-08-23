---
title: "SpringCloud 微服务核心面试题汇总"
published: 2026-08-23
description: "全面整理 Spring Cloud 微服务核心面试题：服务注册与发现（Eureka vs Nacos）、Ribbon 负载均衡、服务雪崩与熔断降级（Sentinel/Hystrix）、SkyWalking 链路追踪监控、分布式限流与网关等核心要点。"
tags: ["Java", "Spring Cloud", "微服务", "面试"]
category: "面试"
draft: false
---

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/325854c9df94bdad.png)
# SpringCloud常用组件
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/65ba72593489afdf.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3db44c1c555e701d.png)
## 注册中心
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/cbb1de629ccfaa40.png)
### Eureka
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bdb11d85152f8fe4.png)
上图体现了服务注册和服务发现，那当user-service宕机呢
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5ef8e6aa24dd9dc2.png)
> 注册中心有一个心跳机制，当某个实例超过时间没有发送心跳，则从注册中心减去这个实例
**服务注册和发现是什么意思？Spring Cloud 如何实现服务注册发现？**<br>我们当时项目采用的eureka作为注册中心，这个也是spring cloud体系中的一个核心组件
- 服务注册：服务提供者需要把自己的信息注册到eureka，由eureka来保存这些信息，比如服务名称、ip、端口等等
- 服务发现：消费者向eureka拉取服务列表信息，如果服务提供者有集群，则消费者会利用负载均衡算法，选择一个发起调用
- 服务监控：服务提供者会每隔30秒向eureka发送心跳，报告健康状态，如果eureka服务90秒没接收到心跳，从eureka中剔除
### Nacos
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/432b3e9ad3c1873a.png)
**我看你之前也用过nacos、你能说下nacos与eureka的区别?**<br>
- Nacos与eureka的共同点(注册中心)<br>① 都支持服务注册和服务拉取<br>② 都支持服务提供者心跳方式做健康检测
- Nacos与Eureka的区别(注册中心）<br>①Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式<br>② 临时实例心跳不正常会被剔除，非临时实例则不会被剔除<br>③ Nacos支持服务列表变更的消息推送模式，服务列表更新更及时<br>④ Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式
- Nacos还支持了配置中心，eureka则只有注册中心，也是选择使用nacos的一个重要原因
> ap即高可用模式，cp即强一致模式
## 负载均衡
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3380772ff8645e59.png)
### Ribbon负载均衡流程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/897adc7dc13a793d.png)
### Ribbon负载均衡策略有哪些？
- **RoundRobinRule**：简单轮询服务列表来选择服务器
- **WeightedResponseTimeRule**：按照权重来选择服务器，响应时间越长，权重越小
- **RandomRule**：随机选择一个可用的服务器
- BestAvailableRule：忽略那些短路的服务器，并选择并发数较低的服务器
- RetryRule：重试机制的选择逻辑
- AvailabilityFilteringRule：可用性敏感策略，先过滤非健康的，再选择连接数较小的实例
- **ZoneAvoidanceRule**：以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询
**ZoneAvoidanceRule为Ribbon的默认策略**
### 自定义负载均衡
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e477be978e438194.png)
### 总结
**你们项目负载均衡如何实现的？**<br>微服务的负载均衡主要使用了一个组件Ribbon，比如，我们在使用feign远程调用的过程中，底层的负载均衡就是使用了ribbon<br>**Ribbon负载均衡策略有哪些？**<br>RoundRobinRule：简单轮询服务列表来选择服务器<br>WeightedResponseTimeRule：按照权重来选择服务器，响应时间越长，权重越小<br>RandomRule：随机选择一个可用的服务器<br>ZoneAvoidanceRule：区域敏感策略，以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询(默认)<br>**如果想自定义负载均衡策略如何实现？**<br>提供了两种方式：<br>1，创建类实现IRule接口，可以指定负载均衡策略（全局)<br>2，在客户端的配置文件中，可以配置某一个服务调用的负载均衡策略（局部)
## 服务雪崩 熔断降级
### 服务雪崩
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/483cad64b3aba833.png)
### 服务降级
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/04a9f77f3dda5450.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a0bdac7b369172b8.png)
### 熔断机制
当前接口已经降级，但仍然有大量请求，则会触发熔断机制
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6a10d69808d401b7.png)
### 总结
**什么是服务雪崩，怎么解决这个问题？**
- 服务雪崩：一个服务失败，导致整条链路的服务都失败的情形
- 服务降级：服务自我保护的一种方式，或者保护下游服务的一种方式，用于确保服务不会受请求突增影响变得不可用，确保服务不会崩溃，一般在实际开发中与feign接口整合，编写降级逻辑
- 服务熔断：默认关闭，需要手动打开，如果检测到10秒内请求的失败率超过50%，就触发熔断机制。之后每隔5秒重新尝试请求微服务，如果微服务不能响应，继续走熔断机制。如果微服务可达，则关闭熔断机制，恢复正常请求
> 服务降级针对的是某个接口，而服务熔断针对的是整个服务
## 微服务的监控
当调用链路的某个服务出现问题可以快速的定位到服务
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/50d920fef5833291.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/43d7040d6f3d531a.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a0d96d675006f2d0.png)
可以看到服务的性能，以及接口的响应时间
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/34be24f673298ccb.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/711f1e94a605e802.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7251642c7e26774f.png)
> 告警可以发送邮件，对接企业微信什么的
**你们的微服务是怎么监控的？**<br>我们项目中采用的skywalking进行监控的<br>1，skywalking主要可以监控接口、服务、物理实例的一些状态。特别是在压测的时候可以看到众多服务中哪些服务和接口比较慢，我们可以针对性的分析和优化。<br>2，我们还在skywalking设置了告警规则，特别是在项目上线以后，如果报错，我们分别设置了可以给相关负责人发短信和发邮件，第一时间知道项目的bug情况，第一时间修复
# 微服务业务问题
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/60f83d7b38c1a0d5.png)
## 限流
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/59563bba50b9bc8c.png)
Tomcat是每台机器一个Tomcat，如果是单机项目这么做限流可以，但如果是分布式项目则不行
Nginx和网关在微服务项目中都处于外层，重点介绍Nginx和网关
### Ngnix限流
#### 控制速率
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a365a1f113fd40e0.png)
回答时重点介绍漏桶模型即可，不用介绍参数
#### 控制并发
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/710b531589f6406f.png)
### 网关限流
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ab1dcd643d09ccde.png)
令牌存储在Redis，和Nginx不同的漏桶不同的是，Ngnix漏桶是固定速率放出请求，请求整体是平滑的。而网关桶则是生成令牌是固定速率的，所以可能导致请求数量有波动。比如1秒三个令牌，来了6个请求，那么前三个请求把令牌消耗完，这一秒又产生了三个令牌，会导致一秒发送了6个请求。
### 总结
**你们项目中有没有做过限流？怎么做的？**
**常见的限流算法有哪些？**<br>1，先来介绍业务，什么情况下去做限流，需要说明QPS具体多少<br>●我们当时有一个活动，到了假期就会抢购优惠券，QPS最高可以达到2000，平时10-50之间，为了应对突发流量，需要做限流<br>● 常规限流，为了防止恶意攻击，保护系统正常运行，我们当时系统能够承受最大的QPS是多少（压测结果）
2，nginx限流<br>● 控制速率（突发流量），使用的漏桶算法来实现过滤，让请求以固定的速率处理请求，可以应对突发流量● 控制并发数，限制单个ip的链接数和并发链接的总数<br>3，网关限流<br>在spring cloud gateway中支持局部过滤器RequestRateLimiter来做限流，使用的是令牌桶算法● 可以根据ip或路径进行限流，可以设置每秒填充平均速率，和令牌桶总容量
## CAP和BASE
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/97b1fec8a25c86e6.png)
### CAP
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b57c7bc0b4cc17c4.png)
#### 一致性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ca475d5b366b19a8.png)
#### 可用性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/76d297201a19458a.png)
#### 分区可用性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7685e2df2b4b85ea.png)
#### 总结
● 分布式系统节点之间肯定是需要网络连接的，分区（P)是必然存在的<br>● 如果保证访问的高可用性（A）,可以持续对外提供服务，但不能保证数据的强一致性--\> AP
● 如果保证访问的数据强一致性（C）,就要放弃高可用性 --\>CP
### BASE理论
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5e71771f3f664acd.png)
### 总结
解释一下CAP和BASE
<br>
- CAP定理(一致性、可用性、分区容错性)
1. 分布式系统节点通过网络连接，一定会出现分区问题(P)
2. 当分区出现时，系统的一致性（C）和可用性（A）就无法同时满足
- BASE理论
1. 基本可用
2. 软状态
3. 最终一致
- 解决分布式事务的思想和模型：
1. 最终一致思想：各分支事务分别执行并提交，如果有不一致的情况，再想办法恢复数据(AP)
2. 强一致思想：各分支事务执行完业务不要提交，等待彼此结果。而后统一提交或回滚（CP)
## 分布式事务
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5d5ab0f80df31335.png)
### Seata分布式事务
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/25dc5338f201aa47.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8d7d014215dce602.png)
即CP模式，保证数据的强一致性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e0c5991cac145829.png)
即AP模式，高可用模式
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/dbf7ee6bff97f10c.png)
例如转账服务，Try会把金额冻结，而非直接转账，Confirm后才会实际转账，Cancel即把金额解冻
TCC模式也是AP模式，但Try Confirm Cancel都需要我们编写代码实现，代码耦合度较高。
### MQ分布式事务
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f0b55da71d568a23.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5f47c281eafc0102.png)
Mq有可靠性，写入后支付宝一定可以读取到对应的MQ消息
## 接口幂等性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/515605b51ba3a9a9.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/96fcf910fe77a47e.png)
接口幂等性主要考虑新增和更新操作
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9b8068b336e4dd4c.png)
购买请求业务处理完后，Redis的token被删除，此时多次点击提交订单，由于Redis不存在这个token，所以直接返回，不做业务处理，保证了幂等性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2bc2af0edae92225.png)
点击多次，但只有一个线程能拿到锁处理业务，保证了业务的幂等性

**分布式服务的接口幂等性如何设计?**<br>幂等:多次调用方法或者接口不会改变业务状态，可以保证重复调用的结果和单次调用的结果一致<br>如果是新增数据，可以使用数据库的唯一索引<br>如果是新增或修改数据
- 分布式锁，性能较低
- 使用token+redis来实现，性能较好<br>第一次请求，生成一个唯一token存入redis，返回给前端<br>第二次请求，业务处理，携带之前的token，到redis进行验证，如果存在，可以执行业务，删除token;如果不存在，则直接返回，不处理业务
## 分布式任务调度
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d07b2d17c79465c2.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/76cea6a5ff4f2781.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/33f305c4384b6f99.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9ed62b86f510ecc4.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/01481801c414a1a4.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ecc66fc2bb409395.png)
### 总结
**为什么微服务中需要分布式任务调度（如 XXL-JOB）？**<br>
单机定时任务（如 Spring 的 `@Scheduled`）在微服务多实例部署下存在以下问题：<br>
1. **防止重复执行**：同一个微服务集群部署了多个节点，单机定时任务会导致所有实例同时触发，造成重复扣费、重复发短信等业务冲突。<br>
2. **解决单点故障（高可用）**：单机定时任务一旦所在机器宕机，任务就会漏跑；分布式任务调度支持**故障转移（Failover）**，当前节点不可用时自动由其他健康节点接管。<br>
3. **支持海量数据并行处理（分片广播）**：单机算力有限，分布式任务调度支持将海量数据拆分成多个分片，由集群中的多台机器并行处理，成倍提升执行效率。<br>
4. **统一可视化管理**：支持动态调整 Cron 表达式、查看执行日志、任务超时控制、失败自动重试及邮件/钉钉告警。
**xxl-job 任务调度的核心架构与流程？**<br>
- **调度中心（xxl-job-admin）**：负责任务的管理、配置与触发，根据路由策略向执行器发送调度请求（自身不跑业务代码）。<br>
- **执行器（微服务客户端）**：负责注册自身服务地址与心跳，接收调度请求并执行具体的业务逻辑。
**xxl-job 常见的路由策略有哪些？**<br>
- **轮询 / 随机**：多个执行器节点轮流或随机执行任务，实现负载均衡。<br>
- **故障转移（Failover）**：按照顺序进行心跳检测，若首选节点心跳异常则自动路由到下一个健康节点。<br>
- **分片广播（Sharding）**：调度中心同时触发集群中所有执行器节点，并下发当前节点的“分片索引”和“分片总数”。
**分片广播的业务场景与实现原理？（举例）**<br>
- **业务场景**：每个月给千万级用户生成账单 / 扫描全量超时订单。<br>
- **实现原理**：比如部署了 5 台机器，分片总数为 5，各节点分别拿到分片序号 0\~4。每台机器只处理 `id % 5 == 分片序号` 的数据，5 台机器并行处理，耗时缩短至原来的 1/5。
