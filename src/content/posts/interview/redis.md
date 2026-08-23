---
title: "Redis 核心面试题：缓存三剑客、双写一致性、分布式锁与集群架构"
published: 2026-08-22
description: "详解 Redis 缓存穿透/击穿/雪崩解决方案、双写一致性与延时双删、RDB 与 AOF 持久化、过期与内存淘汰策略、Redisson 分布式锁看门狗与可重入锁原理、主从复制/哨兵模式/分片集群与 IO 多路复用。"
tags: ["Redis","缓存","分布式锁","集群架构","面试"]
category: "面试"
draft: false
---

# Redis缓存问题
![](/images/posts/84e343378853fa96.png)
## 缓存穿透
### 原理
![](/images/posts/48368dad4cba1efb.png)
比如有人得到你的请求路径，恶意伪造请求攻击你的数据库
![](/images/posts/2da8a035f6bc2744.png)
![](/images/posts/83bfebfb10b5e3de.png)
![](/images/posts/96d8eadc15bd7cea.png)
布隆过滤器可能会存在**误判**
在布隆过滤器的具体实现中可以设置误判率，一般设置在百分之五以内
![](/images/posts/119e5a6bf68b8548.png)
### 回答
![](/images/posts/8900b77cd5e3449e.png)
## 缓存击穿
### 原理
![](/images/posts/52c19e270fc64492.png)
![](/images/posts/6231a99b8de55daf.png)
![](/images/posts/5a4b6f4ba5c69598.png)
### 回答
![](/images/posts/3cefc350d218bc05.png)
## 缓存雪崩
### 原理
![](/images/posts/775af8ce683a3a93.png)
### 回答
![](/images/posts/d984ced2b7d2db26.png)
## 总结
![](/images/posts/6f5f445cf4810674.png)
# 双写一致性
![](/images/posts/df17c35431d37b33.png)
## 强一致性
![](/images/posts/091b32a78c6ff0e5.png)
## 先删除缓存再操作数据库
正常情况
![](/images/posts/a54993e1bd50e005.png)
非正常情况，可能会出现数据不一致问题
![](/images/posts/f24fa3af706d93c4.png)
## 先操作数据库再删除缓存
正常情况
![](/images/posts/2aca7457103e0e93.png)
非正常情况
![](/images/posts/43799cc4a6b3b163.png)
当线程一恰好查到过期的缓存，查数据库，此时线程二还未操作数据库，线程一拿到的是旧数据，也会把旧数据写入缓存，造成缓存和数据库数据不一致
## 因此两种操作都会导致数据不一致
## 延时双删
![](/images/posts/b4dbc38808ead9c3.png)
## 读写锁对比分布式锁
![](/images/posts/10cc96e108c332c3.png)
左边为使用分布式锁，只有一个线程得到锁才可以继续操作
因此效率低下
而右边是读写锁，效率较高
**读锁**
![](/images/posts/5b2f4398433d1c6e.png)
**写锁**
![](/images/posts/89b05f35a031a97d.png)
![](/images/posts/0aa273ecdc65c033.png)
**注意获取的是同一把锁**
![](/images/posts/34a7d910154323b6.png)
因为会阻塞其他线程，因此性能比较低下，只推荐强一致性使用
## 对一致性要求不严格的场景
![](/images/posts/c85105cabb40c6c7.png)
由于使用的消息中间件，所以必定有延迟
![](/images/posts/a5f971dd8a6cc06b.png)
## 回答
![](/images/posts/6566c3683e348265.png)
# Redis持久化问题
![](/images/posts/835a9b2ad08601f1.png)
## RDB
![](/images/posts/86c67e68864ffa2e.png)
![](/images/posts/4c96eb94c4fb1594.png)
linux中，进程无法直接操作物理内存，只能操作虚拟内存，通过页表将对虚拟内存的操作映射到物理内存中
## AOF
![](/images/posts/76001cf2b47ce4e2.png)
![](/images/posts/d4c517c45d617440.png)
![](/images/posts/64f32c1bb40cee42.png)
![](/images/posts/c75b2e238372897e.png)
# Redis过期策略
![](/images/posts/ce9f4ed07459d502.png)
## 惰性删除
![](/images/posts/3ea142715d998280.png)
即使用的时候判断一次
## 定期删除
![](/images/posts/f0d85dbf4fdb2f85.png)
## 回答
![](/images/posts/88a3227d17e5862c.png)
# Redis淘汰策略
## 原理
![](/images/posts/c273650a05598813.png)
![](/images/posts/3260b499550f8e2c.png)
![](/images/posts/3c78156db471a79c.png)
## 总结
![](/images/posts/3e7a0bf4b128f267.png)
# 分布式锁
![](/images/posts/b876ee732edb1c28.png)
## 抢券场景超卖问题
![](/images/posts/36524b4ea3138a82.png)
**正常情况**
![](/images/posts/21b6a2cfb3348376.png)
**出现超卖**
![](/images/posts/8342db48816ef637.png)
**线程一和线程二查询的时候都是1，两个都减去一个，那么库存变成-1，即出现了超卖问题**
## 互斥锁
**使用互斥锁**
![](/images/posts/1de37d8bbe6db41e.png)
**但互斥锁只能在同一个jvm中生效，如果多台机器同时部署，那么无法获得同一把锁**
## 分布式锁
![](/images/posts/bd814c12523fd145.png)
**获得的不是同一把锁**
![](/images/posts/eaf87fbfd18c5786.png)
**使用Redis作为分布式锁**
![](/images/posts/7d4bf92ec7120d16.png)
# 分布式锁原理
![](/images/posts/eb65aae2af3bb5b7.png)
**主要要给锁设置过期时间，否则会出现，持有锁的业务宕机了无法释放锁，那么其他线程只能等待，会造成死锁问题**
---
**同时还有锁的过期时间设置，根据业务时间预估是不可行的，比如网络波动都会影响，因此需要给锁续期，可以开启一个新的线程监视，redis已经帮我们实现了这样的锁**
---
## 分布式锁Redisson
![](/images/posts/4f7019d9eacd9132.png)
redisson的重试机制并不是一直循环等待
![](/images/posts/dff7a3afbefc36ec.png)
如果自己设置了锁的存在时间，那么看门狗机制就不会生效
---
**lua脚本保证执行命令的原子性**
---
## Redisson可重入锁
**同一个线程可以重复获取同一把锁**
![](/images/posts/66b11d362b659a52.png)
每次获取一次锁value+1,释放一次锁value-1
## Redisson主从一致性
![](/images/posts/e93209b9419aee28.png)
**当写入从主节点获取锁时，主节点宕机那么redis的哨兵机制会重新推举出一个主节点，此时一个新的线程来获取锁可以成功，那么锁的互斥性就失效了，可能会出现脏数据问题**
---
![](/images/posts/41aaf221b5728997.png)
**使用zookeeper可以实现分布式锁的强一致性**
---
## 总结
![](/images/posts/12fffe49de7f8aeb.png)
# Redis集群方案
![](/images/posts/7acf3fa287b14d5a.png)
## 主从复制
### 原理
![](/images/posts/d13f26cd9f4e040c.png)
![](/images/posts/991bd8e2e595aeaa.png)
![](/images/posts/5131029328aeb9a3.png)
![](/images/posts/b334fcaf20ccfe44.png)
### 回答
![](/images/posts/2377c8db8d087d20.png)
## 哨兵模式
### 原理
**主从模式一写多读，无法解决Redis的高可用问题**
![](/images/posts/c857055d733b9cf9.png)
![](/images/posts/56cd380c8ae3ee63.png)
### 脑裂问题
**正常模式下，主从节点处于一个网络下，可以正常监控**
![](/images/posts/23a47fcacfdfff8e.png)
**假如网络出现问题，哨兵无法监测主节点，主节点也没有宕机，那么此时哨兵会从从节点中推举出一个主节点，此时有两个主节点，即脑裂**
![](/images/posts/aa892e5c05061e82.png)
**网络恢复后，老的master节点会被降级成为slave几点，此时从新的master节点同步数据，老的master会清空自己的数据，但之前脑裂客户端也向老master节点写入数据，导致数据丢失**
![](/images/posts/f6e92c60c9531725.png)
**通过这两个配置，达不到这样的要求，则拒绝客户端的写入请求，防止大量数据丢失**
### 回答
![](/images/posts/f0abe6396fc2858b.png)
## 分片集群
### 原理
**有很多个master节点，每个master节点又附带一些slave节点**
![](/images/posts/1a73692fb118cf0b.png)
![](/images/posts/494535663ee02f56.png)
**如果想要一些业务相关的key放到一个master，那么就可以对key设置一样的有效部分，例如右侧的aaa**
### 回答
![](/images/posts/04efa826f36f710b.png)
**分片集群解决了海量数据存储，高并发写的问题，同时也有哨兵模式的自动推举master**
# Redis单线程为什么还快
![](/images/posts/e1786dd434e67c0a.png)
## 用户空间与内核空间
![](/images/posts/67b6716c5a3d588e.png)
## 阻塞IO
![](/images/posts/9e234cf0d1802221.png)
## 非阻塞IO
![](/images/posts/a8b39b3b13097be8.png)
这种效率也不高
## IO多路复用
![](/images/posts/eb7e33036c84a8ba.png)
![](/images/posts/aeeca513e4f2e31b.png)
## Redis网络模型
![](/images/posts/8321521a12ae4ead.png)
加入了两个多线程，降低了网络IO
## 总结
![](/images/posts/cb3945b5c1dc55cb.png)