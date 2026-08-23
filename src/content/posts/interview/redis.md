---
title: "Redis 核心面试题：缓存三剑客、双写一致性、分布式锁与集群架构"
published: 2026-08-22
description: "详解 Redis 缓存穿透/击穿/雪崩解决方案、双写一致性与延时双删、RDB 与 AOF 持久化、过期与内存淘汰策略、Redisson 分布式锁看门狗与可重入锁原理、主从复制/哨兵模式/分片集群与 IO 多路复用。"
tags: ["Redis","缓存","分布式锁","集群架构","面试"]
category: "面试"
draft: false
---

# Redis缓存问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784098866774-491eab72-e35b-44bf-a756-086374908ea6.png)
## 缓存穿透
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784098961373-3634fc5f-a440-48c5-ba22-c4c08f181f34.png)
比如有人得到你的请求路径，恶意伪造请求攻击你的数据库
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099048177-f7d3c94e-76f1-4cef-94c4-ed4886fe0e25.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099210658-1ba79a35-a943-4189-8a9e-c46719290128.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099519945-4b67e954-16b6-40ea-88c8-2ae9cc7b52ce.png)
布隆过滤器可能会存在**误判**
在布隆过滤器的具体实现中可以设置误判率，一般设置在百分之五以内
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099606926-ae48b6d8-33b4-410f-8278-582acdcce318.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099695907-933f54d1-2ed5-4799-a730-b842ad54f633.png)
## 缓存击穿
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784099823215-7e6adaba-f414-4eb4-b7cc-9df31d9847dc.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100027312-ca97cbff-c982-4c38-9a31-6816283fabac.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100059613-0b977fe8-a38b-4d63-af12-efe828e382e2.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100095319-e7044a04-0e1a-4bf8-9124-35f081d6bdd2.png)
## 缓存雪崩
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100178282-c0ec825f-eeac-4cbe-b11d-9b5f9bfeae86.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100237234-3dfb3645-ec2c-4d1c-ba20-3c4d804f994b.png)
## 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100299661-3430c971-4dc1-4e25-9c39-f01dd9038c1b.png)
# 双写一致性
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100351497-885466c8-227a-4a80-bef8-3e27f5be59d6.png)
## 强一致性
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100435586-bb17c008-82fd-4a9d-a663-02e63d7de52f.png)
## 先删除缓存再操作数据库
正常情况
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100511063-cbbde0d0-75a1-4c3e-b693-adb0131518f5.png)
非正常情况，可能会出现数据不一致问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100565180-1a1645a8-a8f6-4306-923e-97974accd64d.png)
## 先操作数据库再删除缓存
正常情况
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100640480-df5254bb-24e9-4cc5-8940-105e5f1725a9.png)
非正常情况
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100725556-548a8959-518b-44d1-a428-6ae516a0e9ef.png)
当线程一恰好查到过期的缓存，查数据库，此时线程二还未操作数据库，线程一拿到的是旧数据，也会把旧数据写入缓存，造成缓存和数据库数据不一致
## 因此两种操作都会导致数据不一致
## 延时双删
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784100965659-c4f77f1c-0c47-44eb-b57e-3c7c491f56b8.png)
## 读写锁对比分布式锁
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101055653-25e69ce9-84d1-4576-a2b3-91f3b5cb11b5.png)
左边为使用分布式锁，只有一个线程得到锁才可以继续操作
因此效率低下
而右边是读写锁，效率较高
**读锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101150255-91eae286-7af9-4fa8-bddd-7ffcb016903b.png)
**写锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101184798-c356b276-b9f1-4eb5-9940-6df5031d2aea.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101237706-d528b0b5-a8e1-452c-9a9a-270c32aa0a43.png)
**注意获取的是同一把锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101271859-6b3caa3e-2c65-4edc-9b56-4365409c8f8b.png)
因为会阻塞其他线程，因此性能比较低下，只推荐强一致性使用
## 对一致性要求不严格的场景
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101357702-31669885-7cec-4b2a-8163-eefcee784490.png)
由于使用的消息中间件，所以必定有延迟
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101467696-9dbf2bdb-f49e-4e7b-8255-04ec66b1b575.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101544533-516b96fa-0043-417e-b228-6816a3a151ce.png)
# Redis持久化问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101616234-5b5e05a4-fcac-4df6-8152-09f785376e21.png)
## RDB
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101710317-0c38d5b2-3564-4c90-8c9e-08334b5a43cd.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101873770-285e2ac1-4bb5-4f91-b4b4-10e94f10db17.png)
linux中，进程无法直接操作物理内存，只能操作虚拟内存，通过页表将对虚拟内存的操作映射到物理内存中
## AOF
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784101973475-c59862a3-6968-4a18-9b64-33318c70b271.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102032373-a7beeb49-6fc3-42cf-8c5e-5f5873bbd2b6.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102126933-4c245a60-61e2-49ef-8086-4da3dc24e528.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102210798-4f5308dc-427a-4d82-8976-6f19bed8942d.png)
# Redis过期策略
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102295378-08600256-f964-43c2-a651-d536fbaed43b.png)
## 惰性删除
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102351729-ada95b71-a50c-4280-8ada-cc274423cb93.png)
即使用的时候判断一次
## 定期删除
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102475150-dc026446-62ac-453d-b6c0-0fb9d27043b9.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784102518230-1dc6d206-5c5c-45ce-81a5-75a56ee3fccf.png)
# Redis淘汰策略
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104604312-d2afa8bc-9ca0-402a-9896-512232b05493.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104757715-c383d3c6-2af7-4405-9083-ec7f93185c07.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104816351-3b9257c9-a8e5-4a20-baf5-d9e7ec82bd58.png)
## 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104863765-4c3b46de-1891-4982-918e-d22ace5bd31b.png)
# 分布式锁
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104909007-a0554c6f-16aa-4ec7-9fec-5cc712182e9f.png)
## 抢券场景超卖问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784104989348-daf3e602-8b63-40f1-8e16-08694691076e.png)
**正常情况**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105031621-07f784fd-8d7d-4d7a-9586-d12e77244490.png)
**出现超卖**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105081183-7b36395b-d954-47fd-9551-d1f001840222.png)
**线程一和线程二查询的时候都是1，两个都减去一个，那么库存变成-1，即出现了超卖问题**
## 互斥锁
**使用互斥锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105180953-a9582448-b29c-44cf-bfd3-0837bd634930.png)
**但互斥锁只能在同一个jvm中生效，如果多台机器同时部署，那么无法获得同一把锁**
## 分布式锁
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105293915-fe044fb6-8157-4feb-9432-0cdafd149b10.png)
**获得的不是同一把锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105494480-fa8c18ae-258b-418b-b3a9-9b3b9ca5bed0.png)
**使用Redis作为分布式锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105567061-d19632c9-5da7-47d1-bf55-eca8f93c15d9.png)
# 分布式锁原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784105888163-6c81ca4e-e485-452d-8f12-9f022705020b.png)
**主要要给锁设置过期时间，否则会出现，持有锁的业务宕机了无法释放锁，那么其他线程只能等待，会造成死锁问题**
---
**同时还有锁的过期时间设置，根据业务时间预估是不可行的，比如网络波动都会影响，因此需要给锁续期，可以开启一个新的线程监视，redis已经帮我们实现了这样的锁**
---
## 分布式锁Redisson
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784106149799-6d5ad81f-8684-4b90-a1d3-f2ecd3168473.png)
redisson的重试机制并不是一直循环等待
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784106235109-10527161-5ba6-4a4d-8318-d9ce3a01d2fd.png)
如果自己设置了锁的存在时间，那么看门狗机制就不会生效
---
**lua脚本保证执行命令的原子性**
---
## Redisson可重入锁
**同一个线程可以重复获取同一把锁**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784106525411-ec9ea01b-0774-4a51-b10f-84867e7faded.png)
每次获取一次锁value+1,释放一次锁value-1
## Redisson主从一致性
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784106660668-6240032e-e0f6-466b-8f3c-79b34b8c9fff.png)
**当写入从主节点获取锁时，主节点宕机那么redis的哨兵机制会重新推举出一个主节点，此时一个新的线程来获取锁可以成功，那么锁的互斥性就失效了，可能会出现脏数据问题**
---
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784106873247-4b3eb43b-26e0-4ab6-8e9a-53099418144d.png)
**使用zookeeper可以实现分布式锁的强一致性**
---
## 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107002393-5ff8378f-1343-429c-a02a-12a2275908e2.png)
# Redis集群方案
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107296193-3c6ed106-82af-4298-8830-a35fb31d81b9.png)
## 主从复制
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107376322-311254af-8a64-494b-82f0-0a794ef87f17.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107560708-e15e6c7c-efcd-451f-acba-55f1ab2a9567.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107617239-7f54e4ca-fe65-40bc-8f3f-2e960bc2870a.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107665880-94b2af01-41ad-4e30-85cc-3e5ff09b2442.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107749333-93325d9f-6e8a-49ff-9b66-d4cc753650bc.png)
## 哨兵模式
### 原理
**主从模式一写多读，无法解决Redis的高可用问题**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107885906-797d848f-fc4c-4714-ab93-20db10e53812.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784107980758-7e2c0b83-dc2e-4066-b94b-62439e603e5e.png)
### 脑裂问题
**正常模式下，主从节点处于一个网络下，可以正常监控**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784108051825-70f5cb6b-fcf1-4cb9-8d1f-398fae1bbc66.png)
**假如网络出现问题，哨兵无法监测主节点，主节点也没有宕机，那么此时哨兵会从从节点中推举出一个主节点，此时有两个主节点，即脑裂**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784108133923-5c4c8cba-1ed0-4630-bf92-e74d7ea96121.png)
**网络恢复后，老的master节点会被降级成为slave几点，此时从新的master节点同步数据，老的master会清空自己的数据，但之前脑裂客户端也向老master节点写入数据，导致数据丢失**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784108387530-40595c34-3dd5-4261-bd81-09df0770d188.png)
**通过这两个配置，达不到这样的要求，则拒绝客户端的写入请求，防止大量数据丢失**
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784108569985-a4ca1ff2-91ec-4d7e-b37f-a9d4f39cb616.png)
## 分片集群
### 原理
**有很多个master节点，每个master节点又附带一些slave节点**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784109201996-5b4f3c9a-b825-4ba5-b5f7-6c9b2772f31d.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784109294515-91e73c2b-413e-4e00-a25a-fa167fd2ca61.png)
**如果想要一些业务相关的key放到一个master，那么就可以对key设置一样的有效部分，例如右侧的aaa**
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784109421935-e57232c9-0e5d-4975-bedb-c9137b3c9169.png)
**分片集群解决了海量数据存储，高并发写的问题，同时也有哨兵模式的自动推举master**
# Redis单线程为什么还快
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784110635302-702ecc69-d34a-4d05-81d0-49661addf072.png)
## 用户空间与内核空间
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784110780426-7677a8ad-1f97-44cb-ae55-2bfe6c9ebffa.png)
## 阻塞IO
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784110860865-ca09967c-f674-4b69-9f8f-174697b904de.png)
## 非阻塞IO
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784110938214-8b31c788-d9d6-4e3c-a2c5-5d523e284bd9.png)
这种效率也不高
## IO多路复用
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784111079420-ad2f9624-1873-4e01-a033-984f3e767a00.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784111298911-6d822027-398d-4112-8f58-0256bdc18487.png)
## Redis网络模型
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784111458089-67a3d702-ee11-40a0-aef2-57038d0fbfa4.png)
加入了两个多线程，降低了网络IO
## 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784111593449-2fdb733f-6522-493b-93bc-643928e85b0a.png)