---
title: "Java 多线程与并发编程核心面试题精讲"
published: 2026-08-14
description: "深入剖析线程与进程、线程状态机与生命周期、wait/sleep/notify、Synchronized 锁升级原理（偏向锁/轻量级锁/重量级锁与 MarkWord）、JMM 内存模型、CAS 与原子操作。"
tags: ["Java","多线程","并发编程","JMM","面试"]
category: "面试"
draft: false
---

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/359856b316be42d8.png)
# 线程基础知识
## 线程和进程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/100af7b0ff8825a4.png)
### 进程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/16399320a84208b8.png)
### 线程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5e7f0f4c010381a2.png)
### 区别
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7bbb44a36ade9c30.png)
## 并行与并发
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3d9a947b1e2077a7.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c1344f7017e32758.png)
##
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f9fa34bd0871dc29.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/cf9790d3b04681b0.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/30fdc3975088a485.png)
线程创建的方式
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7bfc7f079bbfaa0d.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9df6dbb1422a430f.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0646e626a41ad004.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/783dd56a7b8af196.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/af6908f8f491d8c4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1eb705b1d2f68b7e.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e896f8e90c6c54ca.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e47bda17585c1771.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/636578071a42f001.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/da6500aa2c8a5e62.png)
**直接调用run方法就是调用普通方法**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/64cd8332c4603691.png)
## 线程的状态
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f63e117d0bbf9f01.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2cdf227d0d00b918.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/07ebfc132c9b530e.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/350d083ddfc5a5ea.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3c5eefd2c5570ec6.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bfc89d626dcd46ff.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/70000c2ff2671e56.png)
## 线程的执行顺序
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ce800268ad614059.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/15ad4bfe3c8f5c46.png)
## notify和notifyall有什么区别
##
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/55e03cf453bd017b.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7e4f566544cb02ac.png)
wait和sleep方法的不同
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d807025ccb1f099a.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c199df386f19430b.png)
**sleep的时候不会释放锁，sleep完成后会释放锁**
## 如何停止正在运行的线程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f97fada6b051f47d.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f4445971fa7683ca.png)
# 线程并发安全
## Synchronized关键字的底层原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f975cc30761fd3b1.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d1ade1215a84bc43.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/4a2f6d94b910b339.png)

> 两次解锁，隐式的try catch结构，防止第一次解锁失败

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3cd5116e3565d343.png)

> lock对象关联Monitor结构

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f4ec30837b7d6b5d.png)
## Synchronized关键字的底层原理进阶
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/51506db605584eab.png)
### 重量级锁
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/72e7cd599bf61d94.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8898d14ddcd9ea04.png)
> 主要介绍MarkWord
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/57b19c78ad108e98.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b388698a995b6e5d.png)
### 轻量级锁
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/625e13a80d37384c.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0dcf6284efadda3b.png)

> lockrecord 交换 hashcode age 交换成功，对象头就记录了锁对象的地址
> cas失败，可能是有多个线程竞争，这个时候直接升级成重量级锁。或者是当前锁重入了，例如method1调用method2，他两个是同一个锁，此时会重新添加一条Lock Record,Lock Record有几个则代表此线程重入了锁几次

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/01f6a9755e21fb1a.png)

> 解锁再交换回来，不过第二个 lock record是null，就把这个记录删掉。然后交换，markword又变成无锁状态

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/49bcbe12f36fab0b.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bfd8dd1dcc6f2748.png)
### 轻量级锁与重量级锁的区别
这里的关键不是“有几个线程使用这把锁”，而是“是否有线程在另一个线程持锁期间同时来竞争”。
轻量级锁适用于低竞争场景。多个线程可以依次使用同一把锁，只要前一个线程已经释放，后一个线程才来加锁，持锁区间不重叠。此时主要通过 CAS 修改对象头中的 Mark Word，使其指向当前线程栈上的 Lock Record，不需要让线程阻塞。
```plain text
低竞争：
T1：|------持锁------|释放
T2：                       |----持锁----|
```
重量级锁适用于存在实际并发竞争的场景。当 T1 持锁时，T2/T3 也来尝试加锁，它们无法立即获得锁，需要等待。锁可能膨胀为重量级 Monitor，等待线程进入队列并可能发生阻塞、唤醒和线程调度，因此开销更大。
```plain text
高竞争：
T1：|------------持锁------------|释放
T2：       尝试加锁 → 等待
T3：          尝试加锁 → 等待
```
所以，“轻量级锁”不是只有一个线程使用，而是多个线程使用时竞争较少；“重量级锁”也不要求一定有很多线程，两个线程在持锁区间重叠并产生明显等待，就可能触发锁膨胀。
> 注意：CAS 失败不等于任何情况下都立刻升级为重量级锁。教材中的锁升级流程是经典 HotSpot 模型的简化描述，实际是否自旋、何时膨胀与 JDK/JVM 版本有关。
### 偏向锁
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/fe3de5f005a3572b.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/44bab970d3e88aff.png)

> 写入线程Id，并将偏向锁标志改为1

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8e3802bba4118f08.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/47b7fcce60652dcc.png)
## JMM（Java内存模型）
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2173219b94136679.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6960a3d24b2e649e.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a1d3ccbe7c7a7649.png)

> 例如主内存有int a =10,线程A和线程B都拿到了，A将a修改为9，则先同步到主内存中，线程B再从主内存更新到线程B

## 什么是CAS
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0436b7d518e0d661.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ecf21b0059f2a880.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3dea62f24fce8ba4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/714a26456f272f01.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/945c711d36a37f0c.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a80754e4d36309f5.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c9c1d0cd155fcaf4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/60d2a8aa01a38976.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/141dd6c31997a5e4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6442df27db80f22f.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c35a0e0ad6858dfe.png)
## volatile的理解
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/24c7cd83908a764f.png)
### 保证线程之间的可见性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e1b23eea55bcddca.png)
> `stop` 最初为 `false`，随后启动三个线程。三个线程之间没有固定的执行顺序：如果 t2 在线程 t1 休眠期间读取 `stop`，会读到 `false`；如果 t1 醒来并将 `stop` 改为 `true` 后，t2 才读取，才可能读到 `true`。没有使用 `volatile` 时，即使 t2 读到了 `true`，也不代表 t3 一定能看到更新，t3 仍可能一直使用旧值而陷入死循环。加上 `volatile` 后，t3 才能可靠地读取到 `stop` 的最新值并退出循环。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/192c6b7c3f8c4fbf.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0be0470a6ab39d90.png)
> 使用valotile后，t3正常获取stop的值，但要注意的是，线程执行顺序没有先后之分，若想指定线程执行顺序需要加入相关参数,t1.start,t2.start,t3.start只是依次发出了启动请求，实际顺序由jvm和操作系统调度
### 禁止指令的重排序
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b0c0169eccaaf075.png)
> 情况四中，actor1先执行第二行代码，代码是跳着执行的，发生了指令重排序
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/decf53d57107ecd4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7f34f0c68ca5f784.png)
> 在x上加入volatile是没有用的，写操作只能防止上方的其他写越过屏障，但无法阻止下方写越过屏障，依然有可能存在先写y再些x，得到1，0的结果
> 那在x,y上都加入volatile，也是可以解决问题的，但是指令重排序对cpu来说是一种优化，可以和减低cpu的运行压力，所以都加上会增加性能
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/baef77a62c3b8568.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ab4fd03df6dbf018.png)

可以通过**同步工具**指定某些执行阶段的先后顺序，不能靠 `start()` 的调用顺序或 `sleep()` 来保证。
### 1. `join()`：最简单
如果要求严格按照“线程 1 执行完，再执行线程 2，再执行线程 3”：
```java
public static void main(String[] args) throws InterruptedException {
    t1.start();
    t1.join(); // 等待 t1 执行完

    t2.start();
    t2.join(); // 等待 t2 执行完

    t3.start();
}
```
执行顺序就是：
```plain text
t1 完成 → t2 完成 → t3 开始
```
不过，这样实际上变成了**串行执行**，失去了并发的意义。
### 2. `CountDownLatch`：线程可以同时启动，但按阶段执行
例如：
```java
CountDownLatch t1Done = new CountDownLatch(1);

Thread t1 = new Thread(() -> {
    // 线程1的工作
    stop = true;

    t1Done.countDown(); // 通知线程2：我完成了
});

Thread t2 = new Thread(() -> {
    try {
        t1Done.await(); // 等待线程1完成
        System.out.println(stop);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

t1.start();
t2.start();
```
这里 t1 和 t2 可以同时启动，但 t2 必须等 t1 调用 `countDown()` 后才能继续。
需要注意：
- `volatile`：保证变量的可见性，**不负责指定线程顺序**
- `synchronized`：保证互斥和可见性，但也**不保证线程获得锁的顺序**
- `sleep()`：只能让当前线程暂时休眠，**不能用来可靠地控制执行顺序**
所以，想控制顺序，就使用 `join()`、`CountDownLatch`、`CyclicBarrier` 或 `CompletableFuture` 等同步机制。你笔记中说三个线程没有固定执行顺序，是正确的。[\[1\]](https://app.notion.com/p/3bc841c1a5f08032a2b5c745a5e2a000)
## 什么是AQS
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/80e6b5d86d3a950d.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1e3ac927d093703f.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/15fc61c3a8d9be45.png)
> 释放锁后，则给队列中的head元素持有锁
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d7334f1ab28cd7fd.png)
> 可以把它理解成：**线程 0 和线程 4 同时争抢一个停车位**，但只有一个线程能成功把“空车位”改成“自己的车位”。
假设锁对象当前是空闲状态：
```plain text
对象头：UNLOCKED
```
线程 0 和线程 4 同时尝试加锁：
```plain text
线程0：CAS(UNLOCKED, LockRecord0)
线程4：CAS(UNLOCKED, LockRecord4)
```
CAS 的意思是：如果当前值还是 `UNLOCKED`，就把它改成我的 `LockRecord`；否则失败。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/da8c8fbc79583340.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8172ee865f890747.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ab930557c815ebce.png)
## ReentrantLocak实现原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3143ddf1ae85b6c6.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/4946c3754b11807f.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/914b94eb9a6f659a.png)
> Sync是非公平锁和公平锁的父类
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/66b0c52b573b7c29.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a5b534729bab1237.png)
## synchronized和Lock锁的区别

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e521ea4bfc8abd91.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/674b067316bd13b8.png)
```java
package com.itheima.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest {

    //创建锁对象
    static ReentrantLock lock = new ReentrantLock();
    //条件1
    static Condition c1 = lock.newCondition();
    //条件2
    static Condition c2 = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {

        //可打断
//        lockInterrupt();

        //可超时
//        timeOutLock();

        //多条件变量
        conditionTest();

    }

    /**
     * 多条件变量
     */
    public static void conditionTest(){
        new Thread(() -> {
            lock.lock();
            try {
                //进入c1条件的等待
                c1.await();
                System.out.println(Thread.currentThread().getName()+",acquire lock...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }, "t1").start();
        new Thread(() -> {
            lock.lock();
            try {
                //进入c2条件的等待
                c1.await();
                System.out.println(Thread.currentThread().getName()+",acquire lock...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }, "t2").start();

        new Thread(() -> {
            lock.lock();
            try {
                //唤醒c1条件的线程
                c1.signalAll();
                //唤醒c2条件的线程
//                c2.signal();
                System.out.println(Thread.currentThread().getName()+",acquire lock...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }, "t3").start();


    }

    /**
     * 锁超时
     * @throws InterruptedException
     */
    public static void timeOutLock() throws InterruptedException {

        Thread t1 = new Thread(() -> {
            //尝试获取锁，如果获取锁成功，返回true，否则返回false
            try {
                if (!lock.tryLock(2, TimeUnit.SECONDS)) {
                    System.out.println("t1-获取锁失败");
                    return;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                System.out.println("t1线程-获得了锁");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        System.out.println("主线程获得了锁");
        t1.start();
        try {
            Thread.sleep(3000);
        } finally {
            lock.unlock();
        }
    }

    /**
     * 可打断
     * @throws InterruptedException
     */
    public static void lockInterrupt() throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                //开启可中断的锁

                lock.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("等待的过程中被打断");
                return;
            }
            try {
                System.out.println(Thread.currentThread().getName() + ",获得了锁");
            } finally {
                lock.unlock();
            }
        }, "t1");
        lock.lock();
        System.out.println("主线程获得了锁");
        t1.start();

        try {
            Thread.sleep(1000);
            t1.interrupt();
            System.out.println("执行打断");
        } finally {
            lock.unlock();
        }
    }

}
```

### 代码说明
这段代码围绕同一个 `ReentrantLock` 展示三种并发控制方式：可中断获取锁、限时获取锁，以及通过 `Condition` 进行等待和通知。
#### 1. 可打断获取锁：`lockInterruptibly()`
\*\*作用：\*\*线程等待锁时可以响应中断，不会无限等待。
**执行流程：**
1. 主线程先执行 `lock.lock()`，持有锁。
2. t1 调用 `lockInterruptibly()`，由于锁已被主线程占用，进入等待。
3. 主线程等待 1 秒后调用 `t1.interrupt()`。
4. t1 的等待被打断，抛出 `InterruptedException`，直接退出，不会执行临界区。
5. 主线程最后执行 `unlock()`，释放锁。
#### 2. 超时获取锁：`tryLock(2, TimeUnit.SECONDS)`
\*\*作用：\*\*线程最多等待指定时间，超时后放弃获取锁。
**执行流程：**
1. 主线程先获得锁，并持有 3 秒。
2. t1 调用 `tryLock(2, TimeUnit.SECONDS)`，开始等待。
3. 2 秒内主线程仍未释放锁，t1 返回 `false`，输出获取失败并结束。
4. 第 3 秒主线程释放锁，但 t1 已经超时，不会再获取这把锁。
如果主线程在 2 秒内释放锁，t1 会返回 `true`，进入 `try` 执行临界区，最后在 `finally` 中调用 `unlock()`。等待期间如果 t1 被中断，`tryLock` 会抛出 `InterruptedException`；未成功获取锁时不能调用 `unlock()`。
#### 3. 多条件变量：`Condition`
`Condition` 不是一把新锁，而是同一把 `ReentrantLock` 下的不同等待队列。`c1`、`c2` 可以分别代表不同的等待条件。
**执行流程：**
1. t1 先获得锁，调用 `c1.await()`；它会释放锁并进入 c1 等待队列。
2. t2 随后获得锁，也可以进入某个条件队列等待。
3. t3 获得锁后调用 `c1.signalAll()`，唤醒所有等待 c1 的线程。
4. 被唤醒的线程不会立即执行，而是要等 t3 释放锁后重新竞争并获得锁。
5. 重新获得锁后，`await()` 返回，线程继续执行临界区，最后调用 `unlock()`。
当前代码中 t2 的注释写的是等待 `c2`，但实际调用的是 `c1.await()`；如果要使用两个条件，应改为 `c2.await()`，并使用 `c2.signal()` 或 `c2.signalAll()` 通知。`signal` 不会记住提前发出的通知，实际开发中通常要配合共享状态和 `while` 条件判断。
## 死锁的产生条件以及排查方案
### 死锁的产生条件
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/eda204a6c6708e88.png)
### 如何排查死锁
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5eca9f75d28ef715.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2cfc8a972e60b7de.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/edfc3860d03cc39c.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/65276aec13f17368.png)
## 聊一下ConcurrentHashMap
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/17b681d927e98ed6.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b32ffeb5f21e492c.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e77f9b490cd4f6f8.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/29f6a71e32982c6b.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8a1fe42f307e5a6b.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/78f5bb2ed67746ce.png)
### ConcurrentHashMap 中 CAS 与 synchronized 的配合
JDK 8 及以后，`ConcurrentHashMap` 不使用一把全局锁保护整张表，而是把 **CAS** 和 **synchronized** 结合起来，尽量缩小线程竞争的范围。
#### 1. CAS：抢占空桶
线程先根据 key 的 hash 计算桶下标 `i`。如果 `table[i]` 为空，就尝试把新节点放进去：
```java
if (tabAt(table, i) == null) {
    if (casTabAt(table, i, null, newNode)) {
        // CAS 成功，当前线程完成空桶占位
        break;
    }
    // CAS 失败，重新读取 table[i] 后重试
}
```
这里的关键不是前面的 `table[i] == null` 判断，而是 CAS 会在修改共享内存的瞬间再次检查当前值。
```plain text
初始：table[i] == null

线程 A：提前读到 null
线程 B：提前读到 null

线程 A：CAS(null, 节点 A) —— 成功，table[i] 变成节点 A
线程 B：CAS(null, 节点 B) —— 失败，发现 table[i] 已经是节点 A
```
CAS 是“比较和交换”一体化的原子操作。线程 B 即使之前也读到了 `null`，执行 CAS 时如果发现桶已经被 A 修改，就不会写入节点 B，因此不会覆盖节点 A。
#### 2. synchronized：保护非空桶的复杂修改
如果桶中已经存在节点，说明可能发生了哈希冲突。此时需要遍历链表或操作红黑树，这不是一次简单赋值，而是多个步骤的组合。`ConcurrentHashMap` 会对当前桶的头节点加锁：
```java
synchronized (firstNode) {
    // 查找 key
    // key 已存在：更新 value
    // key 不存在：追加节点或操作红黑树
}
```
这里的 `synchronized` 不是锁住整张 `ConcurrentHashMap`，而是逻辑上保护当前桶的链表或红黑树：
```plain text
线程 A 修改 bucket[1] ─┐
                      ├─ 可以并发
线程 B 修改 bucket[5] ─┘

线程 A 修改 bucket[1]
线程 B 也修改 bucket[1] —— 竞争同一个桶，需要排队
```
`synchronized` 主要解决两个问题。这里的锁对象可以理解为当前桶的头节点；后续线程如果要修改同一个桶，必须先获取这个监视器：
- **互斥**：同一个桶在同一时刻只允许一个线程执行查找、插入、更新或删除。
- **可见性**：一个线程释放监视器后，另一个随后获得同一把锁的线程可以看到前一个线程完成的修改。
#### 3. 为什么要把 CAS 和 synchronized 结合起来
- 只有空桶时，操作很简单，使用 CAS，避免不必要的阻塞。
- 桶不为空时，链表或红黑树的修改步骤较多，使用 `synchronized` 更容易保证整个临界区的正确性。
- 锁只作用于当前桶，不同桶可以同时修改，从而比“一把全局锁”拥有更高的并发度。
一个简化的 `put` 流程是：
```plain text
计算 key 的 hash 和桶下标
        ↓
桶为空？
  ├─ 是：CAS(null, 新节点)
  │       ├─ 成功：插入完成
  │       └─ 失败：重新读取并重试
  └─ 否：synchronized(桶头节点)
          ├─ 找到相同 key：更新 value
          └─ 找不到：追加节点或操作红黑树
```
#### 4. 线程安全的边界
`ConcurrentHashMap` 的单次 `put`、`get`、`remove`，以及 `putIfAbsent`、`compute`、`computeIfAbsent`、`merge` 等原子方法是并发安全的。但下面的多步操作仍然不是原子的：
```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```
应改用：
```java
map.putIfAbsent(key, value);
```
可以把核心思想记成：
```plain text
CAS：负责抢占空桶，失败就重试，不会覆盖成功者
synchronized：负责保护非空桶的复杂修改
桶级锁：不同桶并发，同一个桶排队
```

> JDK 7 主要采用 Segment + ReentrantLock 的分段锁；JDK 8 及以后主要采用 CAS + synchronized 的桶级并发设计。

## 导致并发程序出现问题的原因是什么
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/781e9bb80bc8abf2.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f515f19a61429c72.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/64b8d8116980034d.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2a8817d66fc2c24d.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/22951258d7691aa1.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c6ce6466969c6016.png)
# 线程池
## 线程池核心参数（线程池执行原理）
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2ab2889afd5a21d8.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/22c2a2788d8b5f53.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5dc820b6149035dc.png)
```java
package com.itheima.threadpool;

import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class TestThreadPoolExecutor {

    static class MyTask implements Runnable {
        private final String name;
        private final long duration;

        public MyTask(String name) {
            this(name, 0);
        }

        public MyTask(String name, long duration) {
            this.name = name;
            this.duration = duration;
        }

        @Override
        public void run() {
            try {
                LoggerUtils.get("myThread").debug("running..." + this);
                Thread.sleep(duration);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public String toString() {
            return "MyTask(" + name + ")";
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger c = new AtomicInteger(1);
        ArrayBlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(2);

        LinkedBlockingQueue linkedBlockingQueue = new LinkedBlockingQueue();
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                2, //核心线程数
                3, //最大线程数
                0, //应急线程存活时间
                TimeUnit.MILLISECONDS, // 应急线程存活单位
                queue, // 阻塞队列
                r -> new Thread(r, "myThread" + c.getAndIncrement()), // 线程工厂
                new ThreadPoolExecutor.AbortPolicy()); //拒绝策略
        showState(queue, threadPool);
        threadPool.submit(new MyTask("1", 3600000));
        showState(queue, threadPool);
        threadPool.submit(new MyTask("2", 3600000));
        showState(queue, threadPool);
        threadPool.submit(new MyTask("3"));
        showState(queue, threadPool);
        threadPool.submit(new MyTask("4"));
        showState(queue, threadPool);
        threadPool.submit(new MyTask("5",3600000));
        showState(queue, threadPool);
        threadPool.submit(new MyTask("6"));
        showState(queue, threadPool);
    }

    private static void showState(ArrayBlockingQueue<Runnable> queue, ThreadPoolExecutor threadPool) {
        try {
            Thread.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        List<Object> tasks = new ArrayList<>();
        for (Runnable runnable : queue) {
            try {
                Field callable = FutureTask.class.getDeclaredField("callable");
                callable.setAccessible(true);
                Object adapter = callable.get(runnable);
                Class<?> clazz = Class.forName("java.util.concurrent.Executors$RunnableAdapter");
                Field task = clazz.getDeclaredField("task");
                task.setAccessible(true);
                Object o = task.get(adapter);
                tasks.add(o);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        LoggerUtils.main.debug("pool size: {}, queue: {}", threadPool.getPoolSize(), tasks);
    }


}
```
<table header-row="true">
<tr>
<td>**提交**</td>
<td>**当时状态**</td>
<td>**结果**</td>
</tr>
<tr>
<td>任务1</td>
<td>线程0 \< 核心2</td>
<td>创建线程1 执行（占1小时）</td>
</tr>
<tr>
<td>任务2</td>
<td>线程1 \< 核心2</td>
<td>创建线程2 执行（占1小时）</td>
</tr>
<tr>
<td>任务3</td>
<td>线程2 = 核心2</td>
<td>入队（队列:\[3\]）</td>
</tr>
<tr>
<td>任务4</td>
<td>线程2 = 核心2</td>
<td>入队（队列:\[3,4\]）</td>
</tr>
<tr>
<td>任务5</td>
<td>队列已满(2)</td>
<td>线程2 \< 最大3 → 创建线程3 执行（占1小时）</td>
</tr>
<tr>
<td>**任务6**</td>
<td>**线程3 = 最大3，队列满**</td>
<td>**拒绝 → 抛异常**</td>
</tr>
</table>
## 线程池中常见的阻塞队列
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3b95fbf0a2107197.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/81c800c8a332eb50.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0c740ee12994ce96.png)
> 有界无界即队列的容量大小，LinkedBlockingQueue地产是链表，如果创建时不指定容量，那么默认为Integer.MaxValue，一般使用LinkedBlockingQueue也要指定容量<br>LinkedBlockingQueue首尾两个锁，操作效率高，二者都是先进先出
## 如何确定核心线程数
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f07d4aa382de65c1.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/99a68f2ca0510bbf.png)
> N为计算机核心数，IO密集型任务对cpu占用不高，可以多设置点线程。<br>而Cpu密集型会高度占用Cpu，因此要避免线程切换带来的性能损耗。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f782d33516e23ad4.png)
## 线程池的种类
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3a026229c05142b3.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d7de4f37c8c231d7.png)
```java
package com.itheima.threadpool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FixedThreadPoolCase {

    static class FixedThreadDemo implements Runnable{
        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            for (int i = 0; i < 2; i++) {
                System.out.println(name + ":" + i);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //创建一个固定大小的线程池，核心线程数和最大线程数都是3
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        for (int i = 0; i < 5; i++) {
            executorService.submit(new FixedThreadDemo());
            Thread.sleep(10);
        }

        executorService.shutdown();
    }

}

```
```java
PS C:\JavaProjects\面试题目\juc-project>  & 'C:\Software\Jdks\bin\java.exe' '@C:\Users\Chenxin\AppData\Local\Temp\cp_bwbiqm81wdlttphwvn4t7m08j.argfile' 'com.itheima.threadpool.FixedThreadPoolCase'
pool-1-thread-1:0
pool-1-thread-1:1
pool-1-thread-2:0
pool-1-thread-2:1
pool-1-thread-3:0
pool-1-thread-3:1
pool-1-thread-1:0
pool-1-thread-1:1
pool-1-thread-2:0
pool-1-thread-2:1
```
> 可以看到，只有三个核心线程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e774d7eb42045152.png)
```java
package com.itheima.threadpool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NewSingleThreadCase {

    static int count = 0;

    static class Demo implements Runnable {
        @Override
        public void run() {
            count++;
            System.out.println(Thread.currentThread().getName() + ":" + count);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //单个线程池，核心线程数和最大线程数都是1
        ExecutorService exec = Executors.newSingleThreadExecutor();

        for (int i = 0; i < 10; i++) {
            exec.execute(new Demo());
            Thread.sleep(5);
        }
        exec.shutdown();
    }

}

```
```java
PS C:\JavaProjects\面试题目\juc-project>  & 'C:\Software\Jdks\bin\java.exe' '@C:\Users\Chenxin\AppData\Local\Temp\cp_bwbiqm81wdlttphwvn4t7m08j.argfile' 'com.itheima.threadpool.NewSingleThreadCase' 
pool-1-thread-1:1
pool-1-thread-1:2
pool-1-thread-1:3
pool-1-thread-1:4
pool-1-thread-1:5
pool-1-thread-1:6
pool-1-thread-1:7
pool-1-thread-1:8
pool-1-thread-1:9
pool-1-thread-1:10
```
> 只有一个线程，且按照提交线程的顺寻执行
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b3a7680d5aa104aa.png)
```java
package com.itheima.threadpool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CachedThreadPoolCase {

    static class Demo implements Runnable {
        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            try {
                //修改睡眠时间，模拟线程执行需要花费的时间
                Thread.sleep(100);

                System.out.println(name + "执行完了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //创建一个缓存的线程，没有核心线程数，最大线程数为Integer.MAX_VALUE
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 10; i++) {
            exec.execute(new Demo());
            Thread.sleep(1);
        }
        exec.shutdown();
    }

}

```
```java

PS C:\JavaProjects\面试题目\juc-project>  & 'C:\Software\Jdks\bin\java.exe' '@C:\Users\Chenxin\AppData\Local\Temp\cp_bwbiqm81wdlttphwvn4t7m08j.argfile' 'com.itheima.threadpool.CachedThreadPoolCase' 
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
pool-1-thread-1执行完了
pool-1-thread-2执行完了
pool-1-thread-3执行完了
pool-1-thread-4执行完了
pool-1-thread-5执行完了
pool-1-thread-6执行完了
pool-1-thread-7执行完了
pool-1-thread-8执行完了
pool-1-thread-9执行完了
pool-1-thread-10执行完了
```

> 新建线程中，每个线程要睡眠100ms，而我们每隔1ms提交线程，所以没有任何一个线程完成，即创建了10个线程，这样的缺点是，如果线程执行任务久，那么久会创建很多很多的线程，例如执行一个线程1000ms，那么我们每隔1ms开启一个线程，都可能会开启数百个或者1000个线程。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/cdc1dc1850812371.png)
```java
package com.itheima.threadpool;

import java.util.Date;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledThreadPoolCase {

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                String name = Thread.currentThread().getName();

                System.out.println(name + ", 开始：" + new Date());
                Thread.sleep(1000);
                System.out.println(name + ", 结束：" + new Date());

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //按照周期执行的线程池，核心线程数为2，最大线程数为Integer.MAX_VALUE
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(2);
        System.out.println("程序开始：" + new Date());

        /**
         * schedule 提交任务到线程池中
         * 第一个参数：提交的任务
         * 第二个参数：任务执行的延迟时间
         * 第三个参数：时间单位
         */
        scheduledThreadPool.schedule(new Task(), 0, TimeUnit.SECONDS);
        scheduledThreadPool.schedule(new Task(), 1, TimeUnit.SECONDS);
        scheduledThreadPool.schedule(new Task(), 5, TimeUnit.SECONDS);

        Thread.sleep(5000);

        // 关闭线程池
        scheduledThreadPool.shutdown();

    }

}

```
```java
PS C:\JavaProjects\面试题目\juc-project>  & 'C:\Software\Jdks\bin\java.exe' '@C:\Users\Chenxin\AppData\Local\Temp\cp_bwbiqm81wdlttphwvn4t7m08j.argfile' 'com.itheima.threadpool.ScheduledThreadPoolCase' 
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
程序开始：Sat Aug 22 14:25:47 CST 2026
pool-1-thread-1, 开始：Sat Aug 22 14:25:47 CST 2026
pool-1-thread-1, 结束：Sat Aug 22 14:25:48 CST 2026
pool-1-thread-2, 开始：Sat Aug 22 14:25:48 CST 2026
pool-1-thread-2, 结束：Sat Aug 22 14:25:49 CST 2026
pool-1-thread-1, 开始：Sat Aug 22 14:25:52 CST 2026
pool-1-thread-1, 结束：Sat Aug 22 14:25:53 CST 2026
```

> 即延迟任务，分别设置了0秒，1秒，和5秒后

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/077bbe66db887f88.png)
## 为什么不推荐使用Excutors创建线程池
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f05738d75acb79c2.png)
> 推荐使用ThreadPoolExecutor
# 多线程使用场景
## es数据批量导入
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bffc0b76ca82afb5.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b1e22a0f5ba8437d.png)
```java
package com.itheima.application;

import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {
        //初始化了一个倒计时锁 参数为 3
        CountDownLatch latch = new CountDownLatch(3);

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"-begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            //count--
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+"-end..." +latch.getCount());
        }).start();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"-begin...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            //count--
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+"-end..." +latch.getCount());
        }).start();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"-begin...");
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            //count--
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+"-end..." +latch.getCount());
        }).start();
        String name = Thread.currentThread().getName();
        System.out.println(name + "-waiting...");
        //等待其他线程完成
        latch.await();
        System.out.println(name + "-wait end...");
    }
    
}

```
>
	PS C:\\JavaProjects\\面试题目\\juc-project\>  & 'C:\\Software\\Jdks\\bin\\java.exe' '@C:\\Users\\Chenxin\\AppData\\Local\\Temp\\cp_bwbiqm81wdlttphwvn4t7m08j.argfile' 'com.itheima.application.CountDownLatchDemo'<br>Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8<br>Thread-0-begin...<br>Thread-1-begin...<br>main-waiting...<br>Thread-2-begin...<br>Thread-0-end...2<br>Thread-2-end...1<br>Thread-1-end...0<br>main-wait end...<br>每个线程给countdown-1,主线程要等countdonw=0的时候才会继续执行
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c31d9bd04f9025ca.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ff9e9ece41f7b94a.png)

> 将countdownlatch的值设置为总页数，每次导入一页后将countdown-1，提交到线程池中执行任务

```java
package com.itheima.cdl.service.impl;

import com.alibaba.fastjson.JSON;
import com.itheima.cdl.mapper.ApArticleMapper;
import com.itheima.cdl.pojo.SearchArticleVo;
import com.itheima.cdl.service.ApArticleService;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;

@Service
@Transactional
@Slf4j
public class ApArticleServiceImpl implements ApArticleService {

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private RestHighLevelClient client;

    @Autowired
    private ExecutorService executorService;

    private static final String ARTICLE_ES_INDEX = "app_info_article";

    private static final int PAGE_SIZE = 2000;

    /**
     * 批量导入
     */
    @SneakyThrows
    @Override
    public void importAll() {

        //总条数
        int count = apArticleMapper.selectCount();
        //总页数
        int totalPageSize = count % PAGE_SIZE == 0 ? count / PAGE_SIZE : count / PAGE_SIZE + 1;
        //开始执行时间
        long startTime = System.currentTimeMillis();
        //一共有多少页，就创建多少个CountDownLatch的计数
        CountDownLatch countDownLatch = new CountDownLatch(totalPageSize);

        int fromIndex;
        List<SearchArticleVo> articleList = null;

        for (int i = 0; i < totalPageSize; i++) {
            //起始分页条数
            fromIndex = i * PAGE_SIZE;
            //查询文章
            articleList = apArticleMapper.loadArticleList(fromIndex, PAGE_SIZE);
            //创建线程，做批量插入es数据操作
            TaskThread taskThread = new TaskThread(articleList, countDownLatch);
            //执行线程
            executorService.execute(taskThread);
        }

        //调用await()方法,用来等待计数归零
        countDownLatch.await();

        long endTime = System.currentTimeMillis();
        log.info("es索引数据批量导入共:{}条,共消耗时间:{}秒", count, (endTime - startTime) / 1000);
    }

    class TaskThread implements Runnable {

        List<SearchArticleVo> articleList;
        CountDownLatch cdl;

        public TaskThread(List<SearchArticleVo> articleList, CountDownLatch cdl) {
            this.articleList = articleList;
            this.cdl = cdl;
        }

        @SneakyThrows
        @Override
        public void run() {
            //批量导入
            BulkRequest bulkRequest = new BulkRequest(ARTICLE_ES_INDEX);

            for (SearchArticleVo searchArticleVo : articleList) {
                bulkRequest.add(new IndexRequest().id(searchArticleVo.getId().toString())
                        .source(JSON.toJSONString(searchArticleVo), XContentType.JSON));
            }
            //发送请求，批量添加数据到es索引库中
            client.bulk(bulkRequest, RequestOptions.DEFAULT);

            //让计数减一
            cdl.countDown();
        }
    }
}

```

> 原代码实现了“分批写入 ES”，但没有实现“有界的内存分批”。主线程会提前查询多页数据，并将携带 articleList 的任务提交到线程池；当生产速度快于消费速度时，这些数据会被阻塞队列持有，内存占用会随待处理批次增加。因此，分页本身不等于内存受控，还需要让任务只携带分页参数，或增加背压机制。同时，DiscardPolicy 和未在 finally 中调用 countDown() 都可能导致 CountDownLatch 永久等待。

### 优化方案：参数化轻量任务与内存受控（即用即查）
> **核心思想**：主线程只向线程池提交分页元数据（`fromIndex` 和 `pageSize`），不携带数据实体。只有当工作线程被真正调度执行时才去查数据库加载数据，写入 ES 后局部变量随栈帧销毁，GC 可以立即回收，保证 JVM 内存占用恒定可控（最大常驻数据量 = `核心线程数 × PAGE_SIZE`）。
```java
package com.itheima.cdl.service.impl;

import com.alibaba.fastjson.JSON;
import com.itheima.cdl.mapper.ApArticleMapper;
import com.itheima.cdl.pojo.SearchArticleVo;
import com.itheima.cdl.service.ApArticleService;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.TimeUnit;

@Service
@Slf4j
public class ApArticleServiceImpl implements ApArticleService {

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private RestHighLevelClient client;

    @Autowired
    private ExecutorService executorService;

    private static final String ARTICLE_ES_INDEX = "app_info_article";
    private static final int PAGE_SIZE = 2000;

    @SneakyThrows
    @Override
    public void importAll() {
        int count = apArticleMapper.selectCount();
        if (count == 0) {
            log.info("无数据需要导入");
            return;
        }

        int totalPageSize = (count + PAGE_SIZE - 1) / PAGE_SIZE;
        long startTime = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(totalPageSize);

        log.info("开始导入 ES，数据总量: {}，总批次数: {}", count, totalPageSize);

        for (int i = 0; i < totalPageSize; i++) {
            int fromIndex = i * PAGE_SIZE;
            // 仅传递分页参数，任务对象极轻量，主线程几乎不占内存
            executorService.execute(new ImportTask(fromIndex, PAGE_SIZE, countDownLatch));
        }

        // 等待所有批次执行完成（带超时时间，防止异常卡死）
        boolean completed = countDownLatch.await(30, TimeUnit.MINUTES);
        long endTime = System.currentTimeMillis();

        if (completed) {
            log.info("ES 索引数据批量导入完成，共导入: {} 条，耗时: {} 秒", count, (endTime - startTime) / 1000);
        } else {
            log.warn("ES 索引导入超时！可能存在部分任务未完成");
        }
    }

    class ImportTask implements Runnable {
        private final int fromIndex;
        private final int pageSize;
        private final CountDownLatch cdl;

        public ImportTask(int fromIndex, int pageSize, CountDownLatch cdl) {
            this.fromIndex = fromIndex;
            this.pageSize = pageSize;
            this.cdl = cdl;
        }

        @Override
        public void run() {
            try {
                // 1. 延迟加载：工作线程执行时才从 DB 查询数据
                List<SearchArticleVo> articleList = apArticleMapper.loadArticleList(fromIndex, pageSize);

                if (articleList != null && !articleList.isEmpty()) {
                    // 2. 批量写入 ES
                    BulkRequest bulkRequest = new BulkRequest(ARTICLE_ES_INDEX);
                    for (SearchArticleVo vo : articleList) {
                        bulkRequest.add(new IndexRequest().id(vo.getId().toString())
                                .source(JSON.toJSONString(vo), XContentType.JSON));
                    }
                    BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);
                    if (bulkResponse.hasFailures()) {
                        log.error("批次 offset [{}-{}] 写入部分失败: {}", fromIndex, fromIndex + pageSize, bulkResponse.buildFailureMessage());
                    }
                }
            } catch (Exception e) {
                log.error("批次 offset [{}-{}] 导入异常", fromIndex, fromIndex + pageSize, e);
            } finally {
                // 3. 必须在 finally 保证 countDown 执行
                cdl.countDown();
                // 4. 方法结束局部变量 articleList 失去引用，GC 即时回收
            }
        }
    }
}
```

> 优化总结<br>
> 1. 内存可控：内存峰值从 O(总数据量) 降为 O(工作线程数 × PAGE_SIZE)，避免了阻塞队列堆积强引用对象导致的内存溢出。<br>
> 2. 健壮性保障\<：countDown() 移入 finally 块，且主线程 await() 增加最大超时保护，防止单批次异常导致系统永久挂起。

## 数据汇总
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b6a3763b5a5f3cf3.png)
```java
package com.itheima.cdl.controller;

import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Future;


@RestController
@RequestMapping("/order_detail")
@Slf4j
public class OrderDetailController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private ExecutorService executorService;


    @SneakyThrows
    @GetMapping("/get/detail_new/{id}")
    public Map<String, Object> getOrderDetailNew() {

        long startTime = System.currentTimeMillis();

        Future<Map<String, Object>> f1 = executorService.submit(() -> {
            Map<String, Object> r =
                    restTemplate.getForObject("http://localhost:9991/order/get/{id}", Map.class, 1);
            return r;
        });
        Future<Map<String, Object>> f2 = executorService.submit(() -> {
            Map<String, Object> r =
                    restTemplate.getForObject("http://localhost:9991/product/get/{id}", Map.class, 1);
            return r;
        });

        Future<Map<String, Object>> f3 = executorService.submit(() -> {
            Map<String, Object> r =
                    restTemplate.getForObject("http://localhost:9991/logistics/get/{id}", Map.class, 1);
            return r;
        });


        Map<String, Object> resultMap = new HashMap<>();
        resultMap.put("order", f1.get());
        resultMap.put("product", f2.get());
        resultMap.put("logistics", f3.get());

        long endTime = System.currentTimeMillis();

        log.info("接口调用共耗时:{}毫秒",endTime-startTime);
        return resultMap;
    }

    @SneakyThrows
    @GetMapping("/get/detail/{id}")
    public Map<String, Object> getOrderDetail() {

        long startTime = System.currentTimeMillis();

        Map<String, Object> order = restTemplate.getForObject("http://localhost:9991/order/get/{id}", Map.class, 1);

        Map<String, Object> product = restTemplate.getForObject("http://localhost:9991/product/get/{id}", Map.class, 1);

        Map<String, Object> logistics = restTemplate.getForObject("http://localhost:9991/logistics/get/{id}", Map.class, 1);

        long endTime = System.currentTimeMillis();



        Map<String, Object> resultMap = new HashMap<>();
        resultMap.put("order", order);
        resultMap.put("product", product);
        resultMap.put("logistics", logistics);

        log.info("接口调用共耗时:{}毫秒",endTime-startTime);
        return resultMap;
    }


}
```

> getOrderDetail是串行执行，所以效率低。而getOrderDetailNew是使用多线程并行执行，执行时间会比串行高效很多。

> Future接口用来获取多线程的返回值的情况，由于获取的数据是Map\<String, Object\>，所以future接口里面的泛型也是这个，用get方法获取future接口中的数据
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c1d0c2687724d44f.png)
## 异步调用
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f00e87eba153a20d.png)
```java
SearchRequest request = new SearchRequest(ARTICLE_ES_INDEX);

            //设置查询条件
            BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
            //第一个条件
            if(null == keyword || "".equals(keyword)){
                request.source().query(QueryBuilders.matchAllQuery());
            }else {
                request.source().query(QueryBuilders.queryStringQuery(keyword).field("title").defaultOperator(Operator.OR));
                //保存搜索历史
                apUserSearchService.insert(userId,keyword);
            }
```

> insert方法即开心新线程的方法，insert方法是使用注解开启的新线程

```java
@Async("taskExecutor")
    @Override
    public void insert(Integer userId, String keyword) {

        //保存用户记录  mongodb或mysql
        //执行业务

        log.info("用户搜索记录保存成功,用户id:{},关键字:{}",userId,keyword);
}
```

> 通过Async注解开始新线程，taskExecutor即我们自己的线程池

```java
package com.itheima.cdl.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

@Configuration
public class ThreadPoolConfig {

    /**
     * 核心线程池大小
     */
    private static final int CORE_POOL_SIZE = 17;

    /**
     * 最大可创建的线程数
     */
    private static final int MAX_POOL_SIZE = 50;

    /**
     * 队列最大长度
     */
    private static final int QUEUE_CAPACITY = 1000;

    /**
     * 线程池维护线程所允许的空闲时间
     */
    private static final int KEEP_ALIVE_SECONDS = 500;

    @Bean("taskExecutor")
    public ExecutorService executorService(){
        AtomicInteger c = new AtomicInteger(1);
        LinkedBlockingQueue<Runnable> queue = new LinkedBlockingQueue<Runnable>(QUEUE_CAPACITY);
        return new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_SECONDS,
                TimeUnit.MILLISECONDS,
                queue,
                r -> new Thread(r, "itheima-pool-" + c.getAndIncrement()),
                new ThreadPoolExecutor.DiscardPolicy()
        );
    }
}

```

> 通过在应用上使用@EnableAsync  *//开启异步调用*

```java
@MapperScan("com.itheima.cdl.mapper")
@SpringBootApplication
@EnableAsync  //开启异步调用
public class CDLApplication {

    public static void main(String[] args) {
        SpringApplication.run(CDLApplication.class,args);
    }

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
## 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ab2a98cbfdc63dbb.png)
## 如何控制某个方法允许并发访问线程的数量
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1a60d8a3ed86b0cf.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ef67d83c1396d4c3.png)

> 信号量类似一个计数器

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2c991d05bae355c4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6e2ac2e8ee600088.png)

> 没有信号量会堵塞住等待

## ThreadLocal的理解
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3ce8c131a2835d81.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/608a1a509595c4a4.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bec6f308314614d9.png)
```java
package com.itheima.other;

public class ThreadLocalTest {
    static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        new Thread(() -> {
            String name = Thread.currentThread().getName();
            threadLocal.set("itcast");
            print(name);
            System.out.println(name + "-after remove : " + threadLocal.get());
        }, "t1").start();
        new Thread(() -> {
            String name = Thread.currentThread().getName();
            threadLocal.set("itheima");
            print(name);
            System.out.println(name + "-after remove : " + threadLocal.get());
        }, "t2").start();
    }

    static void print(String str) {
        //打印当前线程中本地内存中本地变量的值
        System.out.println(str + " :" + threadLocal.get());
        //清除本地内存中的本地变量
        threadLocal.remove();
    }

}
```
```java
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
t1 :itcast
t1-after remove : null
t2 :itheima
t2-after remove : null
```

> 每个都是使用自己的ThreadLocal

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0c328b6a32d897ff.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d9979dc719f642e1.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bff6b65076609379.png)
### 为什么实际开发中需要 ThreadLocal？（核心价值）
> `ThreadLocal` 最核心的实际价值不是为了并发计算，而是为了**「线程级上下文隐式传递（解耦参数）」**与**「非线程安全对象的无锁复用」**。可以把它理解为给当前线程绑一个**“随身小背包”**，在当前线程的任何方法层级中随时存取，避免方法参数层层污染。
### 实战典型场景：用户身份验证与上下文隐式传递
#### 1. 痛点：参数污染（参数传染）
在标准的 Spring Boot Web 应用中，一个 HTTP 请求通常由一个独立的 Tomcat Worker 线程处理：
```plain text
Filter/拦截器 → Controller → Service A → Service B → DAO
```
- **不使用 ThreadLocal**：如果 DAO 或 Service B 需要用到当前登录用户的 `userId`，就必须在 Controller、Service A、Service B 的**每一个方法签名中都加上 ****`Long userId`**** 参数**，造成严重的参数传染和代码冗余。
- **使用 ThreadLocal**：在拦截器鉴权解析 Token 后存入 `ThreadLocal`，后续任何业务层直接通过工具类获取，方法签名极其清爽。
#### 2. 代码实现
**① 定义用户上下文工具类（线程随身小背包）**
```java
package com.itheima.context;

import lombok.Data;

public class UserHolder {
    // 创建线程私有的 ThreadLocal 容器
    private static final ThreadLocal<UserDTO> TL = new ThreadLocal<>();

    public static void set(UserDTO user) {
        TL.set(user);
    }

    public static UserDTO get() {
        return TL.get();
    }

    public static void remove() {
        TL.remove();
    }

    @Data
    public static class UserDTO {
        private Long id;
        private String username;
        private String role;
    }
}
```
**② 在 Spring 拦截器中解析 Token 并存入 / 释放**
```java
package com.itheima.interceptor;

import com.itheima.context.UserHolder;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 1. 获取请求头中的 Token
        String token = request.getHeader("Authorization");
        if (token == null || token.isEmpty()) {
            response.setStatus(401);
            return false;
        }

        // 2. 解析 Token 获取用户信息（模拟）
        UserHolder.UserDTO user = parseToken(token);

        // 3. 将用户信息存入当前线程的 ThreadLocal
        UserHolder.set(user);
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // 4. 关键：请求结束时必须清理 ThreadLocal！
        UserHolder.remove();
    }

    private UserHolder.UserDTO parseToken(String token) {
        UserHolder.UserDTO dto = new UserHolder.UserDTO();
        dto.setId(1001L);
        dto.setUsername("zhangsan");
        return dto;
    }
}
```
**③ 业务层（Controller / Service / DAO）随用随取**
```java
package com.itheima.service.impl;

import com.itheima.context.UserHolder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class OrderServiceImpl {

    public void createOrder() {
        // 无需通过方法参数层层传递，直接从当前线程背包中获取登录用户信息
        UserHolder.UserDTO currentUser = UserHolder.get();
        log.info("当前操作用户ID: {}, 用户名: {}, 正在创建订单...", 
                currentUser.getId(), currentUser.getUsername());
    }
}
```

> \<b\>生产避坑铁律\</b\>：<br>
> 1. \<b\>防数据污染（串号）\</b\>：Tomcat 线程是线程池复用的，若不 remove()，下一个请求复用该线程会读到上一个用户的残留数据。<br>
> 2. \<b\>防内存泄漏\</b\>：ThreadLocalMap 的 Key 是弱引用（会被 GC 回收为 null），但 Value 是强引用；只要线程不销毁，Value 永远无法被 GC 回收，必须在 afterCompletion 或 finally 中显式调用 remove()。

### ThreadLocal 内存泄漏与底层结构
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bcf0b3c0c111aa8c.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6781f6a7008c066e.png)
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5dfc21c35c66355d.png)