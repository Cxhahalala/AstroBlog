---
title: "JUC并发编程"
published: 2025-07-11
description: "JUC 并发编程通俗教程：深入讲解线程与进程、线程池体系、锁机制与并发核心容器的底层原理与实战应用。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817163236565.png"
tags: ["java"]
category: "Java"
author: "Cxhahalala"
draft: false
---

Juc并发编程通俗易懂教程
<!--more-->


# JUC并发编程

## 1. 什么是JUC

 JUC就是java.util.concurrent下面的类包，专门用于多线程的开发。
 ![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250618183524057.png)

## 2. 线程和进程



>进程是操作系统中的应用程序、是资源分配的基本单位，线程是用来执行具体的任务和功能，是CPU调度和分派的最小单位
>
>一个进程往往可以包含多个线程，至少包含一个

### 1）进程

**一个程序，QQ.EXE Music.EXE；数据+代码+pcb**

一个进程可以包含多个线程，至少包含一个线程！

Java默认有几个线程？**2个线程！** main线程、GC线程

### 2）线程

**开了一个进程Typora，写字，等待几分钟会进行自动保存(线程负责的)**

对于Java而言：Thread、Runable、Callable进行开启线程的。

**提问？JAVA真的可以开启线程吗？ 开不了的！**

Java是没有权限去开启线程、操作硬件的，这是一个native的一个本地方法，它调用的底层的C++代码。

```java
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
	//这是一个C++底层，Java是没有权限操作底层硬件的
    private native void start0();

```

### 3）并发

多线程操作同一个资源。

**并发（Concurrency）：**
- 多个线程通过时间片轮转共享CPU资源
- 单核CPU：快速切换线程，营造"同时执行"的效果
- 多核CPU：线程数超过核心数时，仍需要时间片轮转

**并发编程的本质：最大化利用CPU资源，避免CPU空闲等待！**

### 4）并行

**并行：** 多个人一起行走

- CPU多核，多个线程可以同时执行。 我们可以使用线程池！

**获取cpu的核数**

```java
public class Test1 {
    public static void main(String[] args) {
        //获取cpu的逻辑核心(线程)数量
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

### 5）线程的状态

```java
public enum State {

    	//运行
        NEW,

    	//运行
        RUNNABLE,

    	//阻塞
        BLOCKED,

    	//等待
        WAITING,

    	//超时等待
        TIMED_WAITING,

    	//终止
        TERMINATED;
    }

```

### 6）wait/sleep

**1、来自不同的类**

wait => Object

sleep => Thread

一般情况企业中使用休眠是：

```java
// java.util.concurrent.TimeUnit;
TimeUnit.DAYS.sleep(1); //休眠1天
TimeUnit.SECONDS.sleep(1); //休眠1s
```

**2、关于锁的释放**

wait wait要先得到锁才能执行,会释放锁；

sleep睡觉了，不会释放锁；

**3、使用的范围是不同的**

wait 必须在同步代码块中；(在多线程环境下，同一时间只有一个线程可以执行特定的代码段)

sleep 可以在任何地方睡；

**4、是否需要捕获异常**
两者都需要捕获异常

```java
package com.example.juc.demo01;

import java.util.concurrent.TimeUnit;

public class Test1 {
 public static void main(String[] args) throws InterruptedException {
    //获取逻辑核心（线程）

    System.out.println(Runtime.getRuntime().availableProcessors());
    Thread.sleep(1000);
    TimeUnit.SECONDS.sleep(1);

    Test1 test = new Test1();
    test.wait();
 }
}

```

## 3.Lock
   
###  1）传统的 synchronized

```java
package com.example.juc.demo01;

public class SaleTicket {
    public static void main(String[] args) {
        final Ticket ticket = new Ticket();
        // lamda 表达式开启新的线程
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"C").start();
    }
}
// 资源类 OOP 属性、方法
class Ticket {
    private int number = 30;

    //卖票的方式
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "张票剩余" + number + "张票");
        }
    }
}


```

### 2）Lock

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250618194345757.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250618194144425.png)

**公平锁：** 十分公平，必须先来后到~；

**非公平锁：** 十分不公平，可以插队；**(默认为非公平锁)**

```java
package com.example.juc.demo01;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class SaleTicket1 {
    public static void main(String[] args) {
        final Ticket1 ticket = new Ticket1();

        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"C").start();
    }
}
// 1.获取锁
// 2.加锁
// 3.解锁
// 资源类 OOP 属性、方法
class Ticket1     {
    private int number = 30;
    Lock lock = new ReentrantLock();
    //卖票的方式
    public  void sale() {
        if (number > 0) {
            lock.lock(); // 加锁
            try {
                 System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "张票剩余" + number + "张票");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock(); // 解锁
            }
        }
    }
}


```

### 3. Synchronized 与Lock 的区别

1、Synchronized 内置的Java关键字，Lock是一个Java类

2、Synchronized 无法判断获取锁的状态，Lock可以判断

3、Synchronized 会自动释放锁，lock必须要手动加锁和手动释放锁！**可能会遇到死锁**

4、Synchronized 线程1(获得锁->阻塞)、线程2(等待)；lock就不一定会一直等待下去，**lock会有一个trylock去尝试获取锁**，不会造成长久的等待。

5、Synchronized 是可重入锁，不可以中断的，非公平的；Lock，可重入的，可以判断锁，可以自己设置公平锁和非公平锁；

6、Synchronized 适合锁少量的代码同步问题，Lock适合锁大量的同步代码；

## 4. 生产者和消费者的关系

### 1）Synchronzied 版本

```java
package com.example.juc.pc;

public class A {
    public static void main(String[] args) {
        B b = new B();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.increment();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.decrement();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "B").start();
    }
}

class B {
    Integer num = 0;

    public synchronized void increment() {
        if (num != 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        notifyAll();
    }

    public synchronized void decrement() {
        if (num == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        notifyAll();
    }
}

```

### 2）存在问题（虚假唤醒）

**问题，如果有四个线程**，会出现虚假唤醒
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250620193917634.png)

解决方式 ，**if 改为while即可，防止虚假唤醒**

>结论：就是用if判断的话，唤醒后线程会从wait之后的代码开始运行，但是不会重新判断if条件，直接继续运行if代码块之后的代码，而如果使用while的话，也会从wait之后的代码运行，但是唤醒后会重新判断循环条件，如果不成立再执行while代码块之后的代码块，成立的话继续wait。
>
>这也就是为什么用while而不用if的原因了，因为线程被唤醒后，执行开始的地方是wait之后

```java
package com.example.juc.pc;

public class A {
    public static void main(String[] args) {
        B b = new B();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.increment();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.decrement();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.decrement();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                b.increment();
                System.out.println(Thread.currentThread().getName() + "-" + b.num);
            }
        }, "D").start();
    }
}

class B {
    Integer num = 0;

    public synchronized void increment() {
        if (num != 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num++;
        notifyAll();
    }

    public synchronized void decrement() {
        while (num == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        num--;
        notifyAll();
    }
}

```
解释:
初始num = 0
P1进入
increment()
，检查num == 0，不等待，执行num++（num变为1）
P2进入
increment()
，检查num != 0，调用wait()并释放锁
消费者线程C进入
decrement()
，消费num变为0，调用notifyAll()
P2被唤醒，但使用if时它会直接继续执行num++，导致num变为1
但是，如果使用while，P2会再次检查num != 0，发现num == 0，所以会继续等待

为什么synchronized不足以保证线程安全？
synchronized保证的是互斥性：
确保同一时间只有一个线程可以进入同步方法
但是，当线程调用wait()时，它会释放锁并进入等待状态
问题出在wait()之后：
当线程A在wait()时释放了锁
其他线程（比如线程B）可以进入同步方法
当线程B调用notifyAll()时，所有等待的线程（包括线程A）都会被唤醒
线程A被唤醒后需要重新获取锁才能继续执行


### 3）Lock版

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250620194328099.png)


![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250620195305445.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250620194427109.png)

```java
package com.example.juc.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-" + data.num);
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-" + data.num);
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-" + data.num);
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "-" + data.num);
            }
        }, "D").start();
    }
}
// 判断等待，业务，通知
class Data2 {
    Integer num = 0;
    
    Lock lock = new ReentrantLock(); 

    Condition condition = lock.newCondition();
    
    public  void increment() throws InterruptedException {
        lock.lock();
        try {
            while (num != 0) {
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num++;
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException{
        lock.lock();
        try {
            while (num == 0) {
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num--;
            condition.signalAll();
        } catch (Exception e) {
           e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

```

### 4）Condition的优势

精准的通知和唤醒的线程！

**如果我们要指定通知的下一个进行顺序怎么办呢？ 我们可以使用Condition来指定通知进程~**

```java
// A->B
// B->C
// C->A
// ...
package com.example.juc.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.printA();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.printB();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.printC();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
    }
}
// 判断等待，业务，通知
class Data3 {
    Integer num = 1;
    
    Lock lock = new ReentrantLock(); 
    // 创建多个condition
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();
    Condition condition3 = lock.newCondition();
    
    public  void printA() throws InterruptedException {
        lock.lock();
        try {
            while (num != 1) {
                try {
                    condition1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num = 2;
            // 唤醒B
            condition2.signal();
            System.out.println(Thread.currentThread().getName());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() throws InterruptedException{
        lock.lock();
        try {
            while (num != 2) {
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num = 3;
            // 唤醒C
            condition3.signal();
            System.out.println(Thread.currentThread().getName());
        } catch (Exception e) {
           e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() throws InterruptedException{
        lock.lock();
        try {
            while (num != 3) {
                try {
                    condition3.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num = 1;
            // 唤醒A
            condition1.signal();
            System.out.println(Thread.currentThread().getName());
        } catch (Exception e) {
           e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

/*
A==> AAAA
B==> BBBB
C==> CCCC
A==> AAAA
B==> BBBB
C==> CCCC
...
*/
```

## 5. 8锁现象

如何判断锁的是谁！锁到底锁的是谁？

锁会锁住：对象、Class

深刻理解我们的锁

**问题1**

两个同步方法，先执行发短信还是打电话

```java
public class dome01 {
    public static void main(String[] args) {
        Phone phone = new Phone();

        new Thread(() -> { phone.sendMs(); }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> { phone.call(); }).start();
    }
}

class Phone {
    public synchronized void sendMs() {
        System.out.println("发短信");
    }
    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

输出结果为 

发短信

打电话

**为什么？ 如果你认为是顺序在前？ 这个答案是错误的！**

**问题2：**

**我们再来看：我们让发短信 延迟4s**

```java
public class dome01 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();

        new Thread(() -> {
            try {
                phone.sendMs();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> { phone.call(); }).start();
    }
}

class Phone {
    public synchronized void sendMs() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("发短信");
    }
    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

现在结果是什么呢？

结果：**还是先发短信，然后再打电话！**

**why？**

>原因：并不是顺序执行，而是synchronized 锁住的对象是方法的调用！对于两个方法用的是同一个锁，谁先拿到谁先执行，另外一个等待

**问题三**

加一个普通方法

```java
public class dome01 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();

        new Thread(() -> {
            try {
                phone.sendMs();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> { phone.hello(); }).start();
    }
}

class Phone {
    public synchronized void sendMs() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("发短信");
    }
    public synchronized void call() {
        System.out.println("打电话");
    }
    public void hello() {
        System.out.println("hello");
    }
}
```

输出结果为

hello

发短信

> 原因：hello是一个普通方法，不受synchronized锁的影响，不用等待锁的释放

**问题四**

**如果我们使用的是两个对象，一个调用发短信，一个调用打电话，那么整个顺序是怎么样的呢？**

```java
public class dome01 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();

        new Thread(() -> {
            try {
                phone1.sendMs();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> { phone2.call(); }).start();
    }
}

class Phone {
    public synchronized void sendMs() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("发短信");
    }
    public synchronized void call() {
        System.out.println("打电话");
    }
    public void hello() {
        System.out.println("hello");
    }
}
```

输出结果

打电话

发短信

> 原因：两个对象两把锁，不会出现等待的情况，发短信睡了4s,所以先执行打电话

**问题五、六**

**如果我们把synchronized的方法加上static变成静态方法！那么顺序又是怎么样的呢？**
```java
package com.example.juc.lockQuestions;

import java.util.concurrent.TimeUnit;
/**
 * 场景：两个对象实例，但调用的是它们的静态同步方法。
 * 预期打印顺序：phone1, phone2。
 * 解释：当使用 `synchronized` 锁定静态方法时，锁对象是类的 `Class` 对象。由于每个类只有一个 `Class` 对象，
 * 因此所有静态同步方法都共享同一个锁。这意味着即使存在多个对象实例，对静态同步方法的访问也是互斥的，
 * 从而保证了线程安全和预期的执行顺序。
 */
public class Test3 {
    public static void main(String[] args) throws InterruptedException {
       // 两个对象的Class类模板只有一个，static, 锁的是类的class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();   
        new Thread(() -> {
            phone1.phone1();
        }, "A").start();

        TimeUnit.SECONDS.sleep(3);
        new Thread(() -> {
            phone2.phone2();
        }, "B").start();
    }
}

class Phone3 {
    // 对于synchronized实例方法，锁对象是当前实例(this)
    // 对于synchronized静态方法，锁对象是类的Class对象
    // 由于静态方法在类加载时初始化，所有静态同步方法共享同一个Class对象锁
    public static synchronized void phone1() {
        try {
            // 休眠5秒
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("phone1");
    }


     public static synchronized void phone2() {
        System.out.println("phone2");
    }
}


```

（1）我们先来使用一个对象调用两个方法！

答案是：**先打印phone1,后打印phone2**

（2）如果我们使用两个对象调用两个方法！

答案是：**还是打印phone1,后打印phone2**

原因是什么呢？ **为什么加了static就始终前面一个对象先执行呢！为什么后面会等待呢？**

原因是：**对于static静态方法来说，对于整个类Class来说只有一份，对于不同的对象使用的是同一份方法，相当于这个方法是属于这个类的，如果静态static方法使用synchronized锁定，那么这个synchronized锁会锁住整个对象！不管多少个对象，对于静态的锁都只有一把锁，谁先拿到这个锁就先执行，其他的进程都需要等待！**

---

**问题七**

**如果我们使用一个静态同步方法、一个同步方法、一个对象调用顺序是什么？**
```java
package com.example.juc.lockQuestions;

import java.util.concurrent.TimeUnit;
/**
 * 同步代码块和静态同步方法
 * 先打印phone2再打印phone1
 * 因为同步代码快和静态同步方法使用的不是同一把锁
 * 静态代码块锁的是Class
 * 而同步代码块锁的是具体的对象
 */
public class Test4 {
    public static void main(String[] args) throws InterruptedException {
       // 两个对象的Class类模板只有一个，static, 锁的是类的class
        Phone4 phone1 = new Phone4();
        new Thread(() -> {
            phone1.phone1();
        }, "A").start();

        TimeUnit.SECONDS.sleep(3);
        new Thread(() -> {
            phone1.phone2();
        }, "B").start();
    }
}

class Phone4 {
    // 静态同步方法
    public static synchronized void phone1() {
        try {
            // 休眠5秒
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("phone1");
    }

    // 同步代码块
     public  synchronized void phone2() {
        System.out.println("phone2");
    }
}
```

先打印phone2 ，再打印phone1

> 原因：因为一个锁的是Class类的模板，一个锁的是对象的调用者。所以不存在等待，直接运行。

**小解**

**new** 出来的 this 是具体的一个对象

**static Class** 是唯一的一个模板

### 补充
synchronized 是 Java 中用于实现线程同步的关键字，它通过**监视器锁（Monitor Lock）**来确保在同一时间只有一个线程可以执行被保护的代码，从而避免多线程环境下的数据竞争和不一致性。

synchronized 可以用于以下三种情况：

1. 同步实例方法（Synchronized Instance Method） ：
   
   - 当您将 synchronized 关键字应用于一个非静态方法时，它锁定的对象是 当前实例对象（ this ） 。这意味着，如果一个类的两个不同实例，它们各自的同步方法可以同时被不同的线程调用，因为它们锁的是不同的对象。但同一个实例的同步方法，在同一时间只能被一个线程访问。
   - 示例： public synchronized void myMethod() { ... }
2. 同步静态方法（Synchronized Static Method） ：
   
   - 当您将 synchronized 关键字应用于一个静态方法时，它锁定的对象是 当前类的 Class 对象 。由于一个类只有一个 Class 对象，因此所有对该类的静态同步方法的访问都将是互斥的，无论有多少个实例。
   - 示例： public static synchronized void myStaticMethod() { ... }
3. 同步代码块（Synchronized Block） ：
   
   - 这是最灵活的使用方式。您需要明确指定一个 对象作为锁 。只有获取到这个对象的监视器锁的线程才能进入同步代码块。
   - 如果多个线程尝试进入同一个对象的同步代码块，它们将竞争同一个锁。
   - 示例： synchronized (lockObject) { ... }
     - lockObject 可以是任何 Java 对象。通常，我们会选择一个私有的、不可变的、专门用于锁的对象，以避免外部代码意外地获取到这个锁。
     - 在您提供的代码中， synchronized ("xxx") 就是一个同步代码块，它锁定的就是字符串常量池中唯一的那个 "xxx" 字符串对象。
核心要点：

- 锁的是对象，而不是代码或线程。 synchronized 保护的是共享资源，通过锁定一个对象来控制对这些资源的访问。
- 互斥性。 任何时候，只有一个线程能够持有某个对象的监视器锁，从而保证了被 synchronized 保护的代码块的原子性。
- 可见性。 当一个线程释放锁时，它对共享变量的修改会立即刷新到主内存，确保其他线程在获取锁后能看到最新的值。
- 可重入性。 一个线程如果已经持有了某个对象的锁，那么它可以再次进入该对象的任何其他 synchronized 代码块或方法，而不会被自己阻塞。
理解 synchronized 的关键在于识别它到底锁定了哪个对象，因为所有竞争同一个对象的锁的线程都会被同步。

## 6. 集合不安全

### 1）List 不安全

```java
//java.util.ConcurrentModificationException 并发修改异常！
public class ListTest {
    public static void main(String[] args) {

        List<Object> arrayList = new ArrayList<>();

        for(int i=1;i<=10;i++){
            new Thread(()->{
                arrayList.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(arrayList);
            },String.valueOf(i)).start();
        }

    }
}
```

会导致 java.util.ConcurrentModificationException 并发修改异常！

**ArrayList 在并发情况下是不安全的**

解决方案：

```java
public class ListTest {
    public static void main(String[] args) {
        /**
         * 解决方案
         * 1. List<String> list = new Vector<>();
         * 2. List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3. List<String> list = new CopyOnWriteArrayList<>();
         */
        List<String> list = new CopyOnWriteArrayList<>();
        

        for (int i = 1; i <=10; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

**CopyOnWriteArrayList**：写入时复制！ **COW 计算机程序设计领域的一种优化策略** 

核心思想是，如果有多个调用者（Callers）同时要求相同的资源（如内存或者是磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者视图修改资源内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的（transparently）。此做法主要的优点是如果调用者没有修改资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。

读的时候不需要加锁，如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的CopyOnWriteArrayList。

多个线程调用的时候，list，读取的时候，固定的，写入（存在覆盖操作）；在写入的时候避免覆盖，造成数据错乱的问题；

> **CopyOnWriteArrayList**比**Vector**厉害在哪里？
JDK17中两者使用的都是Synchronized关键字来实现的：效率特别低下。

**Vector**底层是使用**synchronized**关键字来实现的：效率特别低下。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624164706749.png)
**CopyOnWriteArrayList** 

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624165704307.png)

但
Vector：
* 在读写操作时都加锁
* 适合读写都频繁且对数据一致性要求高的场景
* 高并发下性能较差
CopyOnWriteArrayList：
* 采用写时复制（Copy-On-Write）策略
* 读操作完全无锁，性能极高
* 写操作加锁，但会创建底层数组的新副本
* 适合读多写少的场景
简单来说，CopyOnWriteArrayList 通过空间换时间的方式，在读多写少的场景下提供了更好的并发性能，而 Vector 的锁粒度更粗，适合写操作较多的场景。


### 2）set 不安全

**Set和List同理可得:** 多线程情况下，普通的Set集合是线程不安全的；

解决方案还是两种：

- 使用Collections工具类的**synchronized**包装的Set类
- 使用CopyOnWriteArraySet 写入复制的**JUC**解决方案

CopyOnWriteArraySet底层维护的是CopyOnWriteArrayList
所有操作都委托给这个 CopyOnWriteArrayList 来完成
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624172108620.png)

```java
public class SetTest {
    public static void main(String[] args) {
        /**
         * 1. Set<String> set = Collections.synchronizedSet(new HashSet<>());
         * 2. Set<String> set = new CopyOnWriteArraySet<>();
         */
//        Set<String> set = new HashSet<>();
        Set<String> set = new CopyOnWriteArraySet<>();

        for (int i = 1; i <= 30; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
}
```

**HashSet底层是什么？**

hashSet底层就是一个**HashMap**；

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624170927459.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624171155112.png)
HashSet的add方法实际就是**put**方法，所有的元素都是**key**，**value**是**PRESENT**，**PRESENT**是一个**static**的**Object**对象；

### 3）Map不安全

```java
//map 是这样用的吗？  不是，工作中不使用这个
//默认等价什么？ new HashMap<>(16,0.75);
Map<String, String> map = new HashMap<>();
//加载因子、初始化容量
```

默认**加载因子是0.75**,默认的**初始容量是16**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250624173442171.png)


同样的HashMap基础类也存在**并发修改异常**！

```java
public class MapTest {
    public static void main(String[] args) {
        //map 是这样用的吗？  不是，工作中不使用这个
        //默认等价什么？ new HashMap<>(16,0.75);
        /**
         * 解决方案
         * 1. Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
         *  Map<String, String> map = new ConcurrentHashMap<>();
         */
        Map<String, String> map = new ConcurrentHashMap<>();
        //加载因子、初始化容量
        for (int i = 1; i < 100; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```


## 7. Callable
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701151858225.png)
**1、可以有返回值；
2、可以抛出异常；
3、方法不同，run()/call()**
Thread只能接受Runablle, FutureTask实现了Runable接口，FutureTask可以包装Callable
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701152957595.png)


![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701153710385.png)
Callable接受的泛型是返回值的类型
```java
package com.example.juc.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread myThread = new MyThread();
        FutureTask<String> futureTask = new FutureTask<>(myThread);
        // new Thread(new Runnacable())
        // new Thread(new FutureTask<>(Callable))
        new Thread(futureTask,"t1").start();
        new Thread(futureTask,"t2").start();
        String s = futureTask.get(); // 可能会导致阻塞，一般放在代码最后或者异步通信
        System.out.println(s);  
    }
    
}

// 实现Callable接口，Callable接受泛型是返回值类型
class MyThread implements Callable<String>{
    @Override
    public String call() throws Exception {
        System.out.println("Callable");
        return "测试成功";
    }
}

```
实际上只会打印一次Callable, FutureTask在被启动后，会缓存Callable的返回值，所以即使有多个线程，也不会打印多次Callable。call方法只会执行一次。
如果需要打印多次Callable，可以创建多个FutureTask实例。

## 8. 常用的辅助类

### 1）CountDownLatch
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701155924134.png)
简单理解为一个减法计时器，只有当计数器归零时，才会继续向下执行。

```java
package com.example.juc.CountDownLatch;

import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName());
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        countDownLatch.await();

        System.out.println("以上线程执行完毕");
    }
}

```

主要方法：

- countDown 减一操作；
- await 等待计数器归零

await 等待计数器归零，就唤醒，再继续向下运行

### 2）CyclickBarrier

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701162332223.png)

从概念上讲，CyclicBarrier可以理解为一种“加法计数器”或者“集合点”：它等待参与者数量达到预设值，而不是像CountDownLatch那样等待计数器减到零。

```java
package com.example.juc.add;

import java.util.concurrent.CyclicBarrier;

public class CyClickBarrierDemo {
    public static void main(String[] args) {
        // 公共屏障点为7，当7个线程都到达屏障点时，执行召唤神龙
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7 , () -> {
            System.out.println("召唤神龙");
        });
        for (int i = 1; i <= 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "集齐7颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
        System.out.println("上面线程已经全部执行");
    }
}
```

### 3）Semaphore
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250701163230155.png)


```java
package com.example.juc.add;

import java.util.concurrent.Semaphore;

public class SemaPhoreDemo {
    public static void main(String[] args) {
        // 信号量，许可数为3  资源//限流使用
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    // 获取信号量
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "获取许可");
                    Thread.sleep(2000);
                    System.out.println(Thread.currentThread().getName() + "释放许可"); 
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                      // 释放信号量
                      semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }

}

```

```java
2获取许可
1获取许可
3获取许可
1释放许可
2释放许可
4获取许可
5获取许可
3释放许可
6获取许可
4释放许可
5释放许可
6释放许可
Process finished with exit code 0
```

原理：

**semaphore.acquire()获得资源，如果资源已经使用完了，就等待资源释放后再进行使用！**

**semaphore.release()释放，会将当前的信号量释放+1，然后唤醒等待的线程！**

作用： 多个共享资源互斥的使用！ 并发限流，控制最大的线程数！

## 9. 读写锁

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705095608336.png)

读写锁即允许多个线程读取，但是写入的时候，其他线程必须等待！
线程的启动顺序不等于执行顺序


/**
 * 独占锁(写锁)：一次只能被一个线程占有
 * 共享锁(读锁)：可以被多个线程占有
 * ReadWriteLock:
 * 读-读：可以共存
 * 读-写：不能共存
 * 写-读：不能共存
 * 写-写：不能共存
 */

```java
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        int num = 6;
        for (int i = 1; i <= num; i++) {
            final int finalI = i;
            new Thread(() -> {

                myCache.write(String.valueOf(finalI), String.valueOf(finalI));

            },String.valueOf(finalI)).start();
        }

        for (int i = 1; i <= num; i++) {
            final int finalI = i;
            new Thread(() -> {

                myCache.read(String.valueOf(finalI));

            },String.valueOf(finalI)).start();
        }
    }
}


/**
 *  方法未加锁，导致写的时候被插队
 */
class MyCache {
    private volatile Map<String, String> map = new HashMap<>();

    public void write(String key, String value) {
        System.out.println(Thread.currentThread().getName() + "线程开始写入");
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "线程写入ok");
    }

    public void read(String key) {
        System.out.println(Thread.currentThread().getName() + "线程开始读取");
        map.get(key);
        System.out.println(Thread.currentThread().getName() + "线程写读取ok");
    }
}

```

```java
6线程开始写入
5线程开始读取
2线程开始写入 #  线程6尚未写入完成，线程2插队写入
4线程开始读取
3线程开始读取
3线程开始写入
3线程写入ok
2线程开始读取
1线程开始读取
1线程写读取ok
6线程开始读取
1线程开始写入
1线程写入ok
6线程写读取ok
2线程写读取ok
4线程开始写入
5线程开始写入
5线程写入ok
3线程写读取ok
4线程写读取ok
2线程写入ok
5线程写读取ok
6线程写入ok
4线程写入ok
Process finished with exit code 0
```

所以如果我们不加锁的情况，多线程的读写会造成数据不可靠的问题。

我们也可以采用**synchronized**这种重量锁和轻量锁 **lock**去保证数据的可靠。

但是这次我们采用更细粒度的锁：**ReadWriteLock** 读写锁来保证


读写锁和Lock锁的区别:
1. **锁的粒度**
   - [Lock]：只有一种锁，完全互斥
   - [ReadWriteLock]：细分为读锁和写锁

2. **并发性能**
   - [Lock]：任何时候都只允许一个线程访问（无论是读还是写）
   - [ReadWriteLock]：
     - 读-读：可以并发执行
     - 读-写：互斥
     - 写-写：互斥

3. **使用场景**
   - [Lock](cci:2://file:///c:/JavaProjects/juc/src/main/java/com/example/juc/rw/ReadWriteLockDemo.java:6:0-30:1)：
     - 适用于读写操作都很少
     - 或者读写操作时间很短的场景
   - [ReadWriteLock](cci:2://file:///c:/JavaProjects/juc/src/main/java/com/example/juc/rw/ReadWriteLockDemo.java:6:0-30:1)：
     - 适用于读多写少的场景
     - 可以显著提高并发性能



```java
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache2 myCache = new MyCache2();
        int num = 6;
        for (int i = 1; i <= num; i++) {
            int finalI = i;
            new Thread(() -> {

                myCache.write(String.valueOf(finalI), String.valueOf(finalI));

            },String.valueOf(i)).start();
        }

        for (int i = 1; i <= num; i++) {
            int finalI = i;
            new Thread(() -> {

                myCache.read(String.valueOf(finalI));

            },String.valueOf(i)).start();
        }
    }

}
class MyCache2 {
    private volatile Map<String, String> map = new HashMap<>();
    private ReadWriteLock lock = new ReentrantReadWriteLock();

    public void write(String key, String value) {
        lock.writeLock().lock(); // 写锁
        try {
            System.out.println(Thread.currentThread().getName() + "线程开始写入");
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "线程写入ok");

        }finally {
            lock.writeLock().unlock(); // 释放写锁
        }
    }

    public void read(String key) {
        lock.readLock().lock(); // 读锁
        try {
            System.out.println(Thread.currentThread().getName() + "线程开始读取");
            map.get(key);
            System.out.println(Thread.currentThread().getName() + "线程写读取ok");
        }finally {
            lock.readLock().unlock(); // 释放读锁
        }
    }
}
```

```java
1线程开始写入
1线程写入ok
2线程开始写入
2线程写入ok
6线程开始写入
6线程写入ok
5线程开始写入
5线程写入ok
4线程开始写入
4线程写入ok
3线程开始写入
3线程写入ok
1线程开始读取
2线程开始读取
5线程开始读取
5线程写读取ok
4线程开始读取
1线程写读取ok
3线程开始读取
4线程写读取ok
6线程开始读取
2线程写读取ok
6线程写读取ok
3线程写读取ok

Process finished with exit code 0
```
可以看到写锁是独占的，读锁是共享的。

##  10. 阻塞队列
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705110800326.png)
Deque即双端队列，两边都可以取出元素


![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705104440688.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705104636073.png)

### 1）BlockQueue概述


`BlockingQueue` 是 `Queue` 的子接口，而 `Queue` 是 `Collection` 的子接口，`Collection` 又继承自 `Iterable`。

什么情况下我们会使用阻塞队列

> 多线程并发处理、线程池

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705105326589.png)



BlockingQueue 有四组api

|     方式     | 抛出异常 | 不会抛出异常，有返回值 | 阻塞，等待 |        超时等待         |
| :----------: | :------: | :--------------------: | :--------: | :---------------------: |
|     添加     |   add    |         offer          |    put     | offer(timenum,timeUnit) |
|     移出     |  remove  |          poll          |    take    | poll(timenum,timeUnit)  |
| 获取队首元素 | element  |          peek          |     -      |            -            |

### ArrayBlockingQueue
```java
/**
     * 抛出异常
     */
    public static void test1(){
        //需要初始化队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);

        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        //抛出异常：java.lang.IllegalStateException: Queue full
//        System.out.println(blockingQueue.add("d"));
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        //如果多移除一个
        //这也会造成 java.util.NoSuchElementException 抛出异常
        System.out.println(blockingQueue.remove());
    }
=======================================================================================
/**
     * 不抛出异常，有返回值
     */
    public static void test2(){
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        //添加 一个不能添加的元素 使用offer只会返回false 不会抛出异常
        System.out.println(blockingQueue.offer("d"));

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        //弹出 如果没有元素 只会返回null 不会抛出异常
        System.out.println(blockingQueue.poll());
    }
=======================================================================================
/**
     * 等待 一直阻塞
     */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);

        //一直阻塞 不会返回
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");

        //如果队列已经满了， 再进去一个元素  这种情况会一直等待这个队列 什么时候有了位置再进去，程序不会停止
//        blockingQueue.put("d");

        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        //如果我们再来一个  这种情况也会等待，程序会一直运行 阻塞
        System.out.println(blockingQueue.take());
    }
=======================================================================================
/**
     * 等待 超时阻塞
     *  这种情况也会等待队列有位置 或者有产品 但是会超时结束
     */
    public static void test4() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);
        blockingQueue.offer("a");
        blockingQueue.offer("b");
        blockingQueue.offer("c");
        System.out.println("开始等待");
        blockingQueue.offer("d",2, TimeUnit.SECONDS);  //超时时间2s 等待如果超过2s就结束等待
        System.out.println("结束等待");
        System.out.println("===========取值==================");
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println("开始等待");
        blockingQueue.poll(2,TimeUnit.SECONDS); //超过两秒 我们就不要等待了
        System.out.println("结束等待");
    }

```

### 2）同步队列SynchronousQueue

同步队列 没有容量，也可以视为**容量为1的队列**；

进去一个元素，必须等待取出来之后，才能再往里面放入一个元素；

**put**方法 和 **take**方法；

**Synchronized** 和 其他的**BlockingQueue** 不一样 它不存储元素；

put了一个元素，就必须从里面先take出来，否则不能再put进去值！

并且SynchronousQueue 的take是使用了**lock锁保证线程安全**的。

```java

import java.util.concurrent.BlockingDeque;
import java.util.concurrent.BlockingQueue;


public class SynchronousQueue {
    public static void main(String[] args) {
        BlockingQueue<String> synchronousQueue = new java.util.concurrent.SynchronousQueue<>();
        // 向queue中添加元素
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + "put 01");
                synchronousQueue.put("1");
                System.out.println(Thread.currentThread().getName() + "put 02");
                synchronousQueue.put("2");
                System.out.println(Thread.currentThread().getName() + "put 03");
                synchronousQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        // 取出元素
        new Thread(()-> {
            try {
                System.out.println(Thread.currentThread().getName() + "take" + synchronousQueue.take());
                System.out.println(Thread.currentThread().getName() + "take" + synchronousQueue.take());
                System.out.println(Thread.currentThread().getName() + "take" + synchronousQueue.take());
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}

```

```java
Thread-0put 01
Thread-1take1
Thread-0put 02
Thread-1take2
Thread-0put 03
Thread-1take3

Process finished with exit code 0
```

## 11. 线程池

CPU核心数决定同一时刻能真正并行执行的线程数量（如8核=8线程），但通过时间片轮转，系统可以"同时"运行更多线程，实际能创建的线程数主要受内存限制。


线程池：三大方式、七大参数、四种拒绝策略

> 池化技术

程序的运行，本质：占用系统的资源！我们需要去优化资源的使用 ===> 池化技术

线程池、JDBC的连接池、内存池、对象池 等等。。。。

资源的 **创建、销毁** 十分消耗资源

**池化技术**：事先准备好一些资源，如果有人要用，就来我这里拿，用完之后还给我，以此来提高效率。

### 1）线程池的好处：

1、降低资源的消耗；

2、提高响应的速度；

3、方便管理；

**线程复用、可以控制最大并发数、管理线程；**


### 2）线程池：三大方法

- **ExecutorService threadPool = Executors.newSingleThreadExecutor();//单个线程**
- **ExecutorService threadPool2 = Executors.newFixedThreadPool(5); //创建一个固定的线程池的大小**
- **ExecutorService threadPool3 = Executors.newCachedThreadPool(); //可伸缩的**
可伸缩线程池，当任务提交速度>线程处理速度的时候，会创建新的线程，当任务提交速度<线程处理速度的时候，会销毁多余的线程


使用execute()或submit()方法提交任务
使用完线程池必须要关闭线程池 shutdown();

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705150759498.png)
```java
//工具类 Executors 三大方法；
package com.example.juc.Excutor;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Demo01 {
    public static void main(String[] args) {
        //单个线程
        ExecutorService threadPool1 = Executors.newSingleThreadExecutor();
        // 固定大小为5个线程的线程池
        ExecutorService threadPool2 = Executors.newFixedThreadPool(5);
        //可伸缩的线程池
        ExecutorService threadPool3 = Executors.newCachedThreadPool();

        try {
            for(int i = 1;i<=100;i++){
                threadPool3.execute(() -> {
                    System.out.println("线程" + Thread.currentThread().getName() + "执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭线程池
            threadPool1.shutdown();
        }
    }
}

```

### 3）七大参数
查看源码
```java
  public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE, //21亿
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```
可以看出创建线程池的三种方法底层都是创建的ThreadPoolExecutor



```java
public ThreadPoolExecutor(int corePoolSize,  //核心线程池大小
                          int maximumPoolSize, //最大的线程池大小
                          long keepAliveTime,  //空闲线程存活时间
                          TimeUnit unit, //超时单位
                          BlockingQueue<Runnable> workQueue, //阻塞队列
                          ThreadFactory threadFactory, //线程工厂 创建线程的 一般不用动
                          RejectedExecutionHandler handler //拒绝策略
                         ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705151908769.png)
一图看懂线程池七大参数
corePoolSize 即，核心线程数，一直存活的线程数量
workQueue，阻塞队列，核心线程都在被使用，那么新的任务将被放入阻塞队列中等待
maximumPoolSize，最大线程数，当阻塞队列也满了的时候，就会创建新的线程，直到达到最大线程数




![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705145125279.png)

阿里巴巴的Java操作手册中明确说明：对于Integer.MAX_VALUE初始值较大，所以一般情况我们要使用底层的**ThreadPoolExecutor**来创建线程池。

```java
package com.example.juc.Excutor;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.LinkedBlockingDeque;
public class Demo01 {
    public static void main(String[] args) {
        //单个线程
        ExecutorService threadPool1 = Executors.newSingleThreadExecutor();
        // 固定大小为5个线程的线程池
        ExecutorService threadPool2 = Executors.newFixedThreadPool(5);
        //可伸缩的线程池
        ExecutorService threadPool3 = Executors.newCachedThreadPool();


        // 使用ThreadPoolExecutor自定义线程池
        ExecutorService threadPool4 = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                // new ThreadPoolExecutor.AbortPolicy() // 阻塞队列和线程池都满了，抛出异常
                // new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的回哪里
                // new ThreadPoolExecutor.DiscardPolicy() // 阻塞队列和线程池都满了，丢弃任务，不抛出异常
                new ThreadPoolExecutor.DiscardOldestPolicy() // 阻塞队列和线程池都满了，尝试和最早的线程竞争，竞争成功则执行，竞争失败则丢弃
        );

        try {
            // 最大承载，Deque+max
            for(int i = 1;i<=9;i++){
                threadPool4.execute(() -> {
                    System.out.println("线程" + Thread.currentThread().getName() + "执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭线程池
            threadPool4.shutdown();
        }
    }
}

```

### 4）拒绝策略

**1. new ThreadPoolExecutor.AbortPolicy()：** //该拒绝策略为：银行满了，还有人进来，不处理这个人的，并抛出异常

超出最大承载，就会抛出异常：队列容量大小+maxPoolSize

**2. new ThreadPoolExecutor.CallerRunsPolicy()：** //该拒绝策略为：哪来的去哪里

**3. new ThreadPoolExecutor.DiscardPolicy():** //该拒绝策略为：队列满了,丢掉异常，不会抛出异常。

**4. new ThreadPoolExecutor.DiscardOldestPolicy()：** //该拒绝策略为： 阻塞队列和线程池都满了，丢弃队列中最老的任务，尝试将新任务加入队列，若队列还是满的拒绝，不会丢出异常

### 5）如何设置线程池的大小

**1、CPU密集型：电脑的核数是几核就选择几；选择maximunPoolSize的大小**

```java
// 获取cpu 的核数
        int max = Runtime.getRuntime().availableProcessors();
        ExecutorService service =new ThreadPoolExecutor(
                2,
                max,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
```

**2、I/O密集型：**

在程序中有15个大型任务，io十分占用资源；I/O密集型就是判断我们程序中十分耗I/O的线程数量，大约是最大I/O数的一倍到两倍之间。

## 12. 四大函数式接口

新时代的程序员：**lambda表达式、链式编程、函数式接口、Stream流式计算**

>简化编程模型

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705165743356.png)

### 1）Function 函数型接口

> 函数式接口：只包含 一个且仅一个 抽象方法 (abstract method) 的接口。
> 抽象方法是只有方法签名，没有方法体的方法
> 抽象类里必须显式写 abstract，接口里可以省略。
>

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705170105185.png)
传入参数T,返回类型R

```java
package com.example.juc.function;

import java.util.function.Function;

public class Demo01 {
    public static void main(String[] args) {
        //匿名内部类
        // Function<Integer, Integer> function =  new Function<Integer, Integer>() {
        //     @Override
        //     public Integer apply(Integer integer) {
        //         return integer * integer;
        //     }
        // };
        //可以使用lambda表达式实现函数式接口
        Function<Integer, Integer> function = (x) -> {return x * x;};
        System.out.println(function.apply(2));
    }
}

```

### 2）Predicate 断定型接口

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705172734549.png)
有一个输入参数，返回值只能是boolean类型

```java
package com.example.juc.function;

import java.util.function.Predicate;

public class Demo02 {
    public static void main(String[] args) {
        Predicate<String> predicate = (s) -> {return s.length() > 5;};
        System.out.println(predicate.test("hello"));
    }
}

```

### 3）Suppier 供给型接口

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705173404065.png)
没有输入只有返回

```java
/**
 * 供给型接口，只返回，不输入
 */
public class Demo4 {
    public static void main(String[] args) {
        Supplier<String> supplier = ()->{return "1024";};
        System.out.println(supplier.get());
    }
}

```

### 4）Consummer 消费型接口

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705173136801.png)

```java
/**
 * 消费型接口 没有返回值！只有输入！
 */
public class Demo3 {
    public static void main(String[] args) {
        Consumer<String> consumer = (str)->{
            System.out.println(str);
        };
        consumer.accept("abc");
    }
}

```

## 13. Stream 流式计算

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705174150099.png)


```java
package com.example.juc.stream;
import java.util.Arrays;
import java.util.List;
/**
 * * 题目要求： 用一行代码实现
 * 1. Id 必须是偶数
 * 2.年龄必须大于23
 * 3. 用户名转为大写
 * 4. 用户名倒序
 * 5. 只能输出一个用户
 */
public class Stream {
    public static void main(String[] args) {
        User u1 = new User(1, "a", 23);
        User u2 = new User(2, "b", 23);
        User u3 = new User(3, "c", 23);
        User u4 = new User(6, "d", 24);
        User u5 = new User(4, "e", 25);
        // 集合就是存储数据的
        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);

        // 计算交给Stream
        // 链式编程
        list.stream().filter((u) -> {
            return u.getId() % 2 == 0; // 判断Id是否为偶数
        }).filter((u) -> {
            return u.getAge() > 23; // 判断年龄是否大于23
        }).map((u) -> {
            return u.getName().toUpperCase(); // 用户名转为大写
        }).sorted((s1, s2) -> {
            return -s1.compareTo(s2); // 用户名倒序
        }).limit(1).forEach(System.out::println); // 只能输出一个用户
    }
}
class User {
    private String name;
    private int age;
    private int id;

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public User(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}

```

流式计算的结果也可以进行收集

```java
List<String> result = list.stream()
    .filter(u -> u.getId() % 2 == 0)
    .filter(u -> u.getAge() > 23)
    .map(u -> u.getName().toUpperCase())
    .sorted(Comparator.reverseOrder())
    .limit(1)
    .collect(Collectors.toList());

// 使用结果
result.forEach(System.out::println);
```

## 14. ForkJoin

ForkJoin 在JDK1.7，并行执行任务！提高效率~。在大数据量速率会更快！

大数据中：**MapReduce 核心思想->把大任务拆分为小任务！**

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705185412596.png)

### 1）ForkJoin 特点： 工作窃取！

实现原理是：**双端队列**！从上面和下面都可以去拿到任务进行执行！

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705185531344.png)

线程B执行完后，会从线程A的队列中拿任务执行，这就是工作窃取！

### 2）如何使用ForkJoin?

- 1、通过**ForkJoinPool**来执行

- 2、计算任务 **execute(ForkJoinTask<?> task)**

- 3、计算类要去继承ForkJoinTask；

  **ForkJoin 的计算类**

```java

import java.util.concurrent.RecursiveTask;


public class ForkJoinDemo extends RecursiveTask<Long> {
    private long star;
    private long end;
    /** 临界值 */
    private long temp = 1000000L;

    public ForkJoinDemo(long star, long end) {
        this.star = star;
        this.end = end;
    }

    /**
     * 计算方法
     * @return
     */
    @Override
    protected Long compute() {
        if ((end - star) < temp) {
            Long sum = 0L;
            for (Long i = star; i < end; i++) {
                sum += i;
            }
            return sum;
        }else {
            // 使用ForkJoin 分而治之 计算
            //1 . 计算平均值
            long middle = (star + end) / 2;
            ForkJoinDemo forkJoinDemo1 = new ForkJoinDemo(star, middle);
            // 拆分任务，把线程压入线程队列
            forkJoinDemo1.fork();
            ForkJoinDemo forkJoinDemo2 = new ForkJoinDemo(middle, end);
            forkJoinDemo2.fork();

            long taskSum = forkJoinDemo1.join() + forkJoinDemo2.join();
            return taskSum;
        }
    }
}
```

**测试类**

```java

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;


public class ForkJoinTest {
    private static final long SUM = 20_0000_0000;

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();
        test2();
        test3();
    }

    /**
     * 使用普通方法
     */
    public static void test1() {
        long star = System.currentTimeMillis();
        long sum = 0L;
        for (long i = 1; i < SUM ; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println(sum);
        System.out.println("时间：" + (end - star));
        System.out.println("----------------------");
    }
    /**
     * 使用ForkJoin 方法
     */
    public static void test2() throws ExecutionException, InterruptedException {
        long star = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, SUM);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);
        Long along = submit.get();

        System.out.println(along);
        long end = System.currentTimeMillis();
        System.out.println("时间：" + (end - star));
        System.out.println("-----------");
    }
    /**
     * 使用 Stream 流计算
     */
    public static void test3() {
        long star = System.currentTimeMillis();

        long sum = LongStream.range(0L, 20_0000_0000L).parallel().reduce(0, Long::sum);
        System.out.println(sum);
        long end = System.currentTimeMillis();
        System.out.println("时间：" + (end - star));
        System.out.println("-----------");
    }
}
```



**.parallel().reduce(0, Long::sum)使用一个并行流去计算整个计算，提高效率。**

## 异步编程与回调：从理念到 `CompletableFuture` 实战

### 1. 核心理念：什么是异步？什么是回调？

在深入 `CompletableFuture` 之前，我们必须理解两个基本概念。

#### **什么是回调 (Callback)？**

回调是一种编程模式：你定义一个函数（A），然后将它作为参数传递给另一个函数（B）。函数B在执行到某个特定时机时，会“回过头来调用”你传给它的函数A。

**生活比喻：** 你去餐厅点餐，服务员给了你一个**震动取餐器**。
*   **任务发起**：你点完餐，付了钱。
*   **回调注册**：服务员把取餐器（回调的“凭证”）给你。
*   **主线程解放**：你不用站在柜台干等，可以回到座位玩手机、聊天（主线程可以做其他事）。
*   **回调触发**：厨房把餐做好了（异步任务完成），服务员按下了按钮，你的取餐器震动了（回调函数被执行）。
*   **结果处理**：你去取餐。

这里的“取餐器震动”这个动作，就是**回调**。

#### **什么是异步 (Asynchronous)？**

异步的核心是 **“立即返回，不阻塞”**。

当你调用一个异步方法时，它会立即返回，让你的主线程可以继续执行后续代码，而那个耗时的任务则在后台（通常是另一个线程）悄悄进行。

**前端AJAX的例子非常贴切：**
*   **同步（synchronous）**：浏览器发送请求 → **必须等服务器响应完毕** → 才能继续执行后续JS代码／渲染UI。在这段等待过程中，页面会“卡住”。
*   **异步（asynchronous）**：浏览器发送请求后**不阻塞主线程**；JS继续往下执行。当服务器返回数据时，会通过一个**回调函数**（如 `Promise.then`）来处理响应。这种非阻塞模型让页面依旧可交互，体验更流畅。

**异步回调**，就是将这两者结合：发起一个不会阻塞当前线程的任务，并预先指定一个回调函数，以便在任务完成后自动处理其结果或异常。`CompletableFuture` 正是 Java 中实现这一模式的利器。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705194740585.png)

---

### 2. `CompletableFuture` 入门：创建异步任务

`CompletableFuture` 提供了两种主要方式来创建异步任务。

| 方法 | 描述 | 对应的函数式接口 |
| :--- | :--- | :--- |
| `runAsync` | 执行一个没有返回值的异步任务。 | `Runnable` |
| `supplyAsync` | 执行一个带有返回值的异步任务。 | `Supplier<T>` |

**注意**：若不指定线程池（`Executor`），默认使用 `ForkJoinPool.commonPool()`。对于I/O密集型任务，强烈建议自定义线程池，以避免阻塞`commonPool`中宝贵的计算线程。

```java
// 1) 没有返回值的异步任务 (runAsync)
CompletableFuture<Void> futureRun = CompletableFuture.runAsync(() -> {
    System.out.println("这是一个没有返回值的异步任务...");
    // 适合执行日志记录、消息发送等操作
});

// 2) 带有返回值的异步任务 (supplyAsync)
CompletableFuture<String> futureSupply = CompletableFuture.supplyAsync(() -> {
    // 适合执行数据库查询、远程API调用等
    return "Hello, Async!";
});

// 3) 使用自定义线程池执行
ExecutorService myThreadPool = Executors.newFixedThreadPool(4);
CompletableFuture<String> futureWithPool = CompletableFuture.supplyAsync(() -> {
    // 模拟调用远程服务
    return callRemoteService();
}, myThreadPool);
```

#### **重要：`get()` 是阻塞的！**

`get()` 方法会**阻塞当前线程**，直到异步任务完成。它通常用于程序的最后，或者当你确实需要同步等待结果时。在异步流的中间过程滥用 `get()` 会使其失去异步的意义。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    System.out.println("主线程开始...");

    CompletableFuture<Void> future = CompletableFuture.runAsync(()->{
        try {
            TimeUnit.SECONDS.sleep(2); // 模拟耗时任务
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("异步任务在线程: " + Thread.currentThread().getName() + " 执行完毕");
    });

    System.out.println("主线程不阻塞，继续执行其他代码...");

    // 调用 get() 会让主线程在这里停下，直到异步任务完成
    future.get();
    System.out.println("主线程在 get() 后继续执行，程序结束。");
}
```

---

### 3. 核心能力：构建异步回调流水线

`CompletableFuture` 的真正威力在于其**链式调用**能力，可以优雅地构建处理流水线。

#### 示例：一个完整的异步处理流程

下面示例演示了这些典型场景：获取一个数字 -> 记录日志 -> 处理异常 -> 转换结果 -> 再发起另一个异步任务。

```java
// 创建一个自定义线程池
ExecutorService executor = Executors.newFixedThreadPool(4);

CompletableFuture<String> finalResultFuture = CompletableFuture
    // ① 异步计算，返回 1024。可以解开注释模拟异常
    .supplyAsync(() -> {
        System.out.println("任务1（supplyAsync）- 线程: " + Thread.currentThread().getName());
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
        // int i = 1/0; // 模拟异常
        return 1024;
    }, executor)

    // ② 注册回调：无论成功/失败都会执行，但【不能】改变结果。常用于打日志。
    .whenComplete((result, exception) -> {
        System.out.println("任务2（whenComplete）- 线程: " + Thread.currentThread().getName());
        if (exception == null) {
            System.out.println("上一步执行成功，结果: " + result);
        } else {
            System.out.println("上一步执行失败，异常: " + exception.getMessage());
        }
    })

    // ③ 注册异常回调：仅在出现异常时触发，用于提供“兜底”数据，让流水线继续。
    .exceptionally(exception -> {
        System.err.println("任务3（exceptionally）- 捕获到异常: " + exception.getMessage());
        return 404; // 返回一个替代结果，future 会转为正常完成状态
    })

    // ④ 注册成功回调：对上一步的结果进行转换 (Integer -> String)
    .thenApply(result -> {
        System.out.println("任务4（thenApply）- 线程: " + Thread.currentThread().getName());
        return "最终结果: " + result;
    })

    // ⑤ 注册成功回调：继续执行另一个异步任务，并将两个任务“连接”起来
    // thenCompose 用于解决 CompletableFuture<CompletableFuture<T>> 嵌套问题
    .thenCompose(str -> {
        System.out.println("任务5（thenCompose）- 线程: " + Thread.currentThread().getName());
        return CompletableFuture.supplyAsync(() -> str + " -> 再加上用户信息", executor);
    });

// 在主线程最后，阻塞等待最终结果
System.out.println("主线程已提交所有任务，等待最终结果...");
System.out.println("最终拿到的结果是: " + finalResultFuture.join());
executor.shutdown();
```

> **执行顺序说明**
> 1. `supplyAsync` 是流水线的起点，它产生原始结果或异常。
> 2. `whenComplete` **总是**会被执行，它像一个观察者，只能“看”结果/异常，但不能修改。
> 3. 如果 `supplyAsync` 抛出异常，`exceptionally` 会捕获它并返回一个备用值，从而让流水线从异常状态恢复为正常状态，后续的 `thenApply` 才能继续执行。若无异常，`exceptionally` 会被跳过。
> 4. `thenApply` 对**成功的结果**进行转换。`thenAccept` 则只消费结果，不返回值。
> 5. `thenCompose` 用于连接两个有依赖关系的异步任务。它会将 `CompletableFuture<CompletableFuture<T>>` 这种嵌套结构“压平”成 `CompletableFuture<T>`，使流水线保持清爽。

---

### 4. 常用 API 小结与对比

| 方法 | 何时触发 | 是否可改变结果 | 返回类型 | 核心用途 |
| :--- | :--- | :--- | :--- | :--- |
| `whenComplete` | 成功 / 失败 | **否** | 原类型 `CompletableFuture<T>` | 日志、监控等**副作用**操作 |
| `exceptionally` | **仅失败** | **是**（提供兜底值） | `CompletableFuture<T>` | 异常恢复，提供默认值 |
| `handle` | 成功 / 失败 | **是** | `CompletableFuture<U>` | 无论成功失败都要处理并**转换结果** |
| `thenApply` | **仅成功** | **是**（映射） | `CompletableFuture<U>` | 同步地转换结果 |
| `thenAccept` | **仅成功** | **否** (消费) | `CompletableFuture<Void>` | 同步地消费结果，无后续 |
| `thenCompose` | **仅成功** | **是**（扁平化） | `CompletableFuture<U>` | **连接**两个**异步**任务 |

#### `get()` vs `join()` 的区别
两者都用于阻塞等待结果，但异常处理方式不同。

| 方法 | 抛出异常类型 | 中断处理 | 典型场景 |
| :--- | :--- | :--- | :--- |
| `get()` | **受检异常** (`InterruptedException`, `ExecutionException`) | 调用者必须 `try-catch` | 在需要精细化处理特定异常（如中断）的业务层使用 |
| `join()` | **非受检异常** (`CompletionException`) | 内部处理中断，不向外抛 `InterruptedException` | 在 Lambda 表达式、链式调用末端或 `main` 方法中，用于简化代码 |

> **小技巧**：在业务代码的底层（如DAO/Service）可以使用 `get()` 进行细致的异常处理。而在上层（如Controller）或测试代码中，可以直接使用 `join()` 来简化代码，让全局异常处理器来捕获 `CompletionException`。

---

### 5. 组合多个异步任务

| 组合方法 | 作用 | 示例 |
| :--- | :--- | :--- |
| `allOf(f1,f2,...)` | 等待 **所有** 任务都完成（返回 `Void`） | `CompletableFuture.allOf(f1,f2).join();` |
| `anyOf(f1,f2,...)` | 等待 **任意一个** 任务完成（返回 `Object`） | `anyOf(f1,f2).thenAccept(System.out::println);` |
| `thenCombine(f1,f2,fn)` | 当**两个**任务都完成后，合并它们的结果 | `f1.thenCombine(f2, (r1, r2) -> r1 + r2);` |
| `applyToEither(f1,f2,fn)`| 当**两个**任务中**任意一个**完成后，使用它的结果 | `f1.applyToEither(f2, result -> "Fastest: " + result);` |

### 6. 超时控制 (JDK 9+)

处理外部依赖时，超时控制至关重要。

```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> slowApiCall());

// 方式一：超时后返回默认值
String result1 = cf.completeOnTimeout("默认值", 3, TimeUnit.SECONDS).join();

// 方式二：超时后抛出异常
try {
    String result2 = cf.orTimeout(3, TimeUnit.SECONDS).join();
} catch (CompletionException e) {
    if (e.getCause() instanceof TimeoutException) {
        System.err.println("任务超时了！");
    }
}
```

*   `completeOnTimeout`：超时后让 Future **正常完成**并返回一个默认值。
*   `orTimeout`：超时后让 Future **异常完成**，抛出 `TimeoutException`。

---

### 异步回调对比同步任务的优势

核心区别在于：**主线程在“等待”期间，是在“忙碌地做别的事”还是在“被动地阻塞”？**


#### 1. 传统同步模式：被动的“阻塞”

在同步模式下，当主线程调用一个耗时方法（比如数据库查询）时，它会**完全卡住**。

```java
public static void main(String[] args) {
    System.out.println("主线程：开始查询数据库...");
    // 这个方法会执行5秒钟
    String result = queryDatabase(); // <--- 主线程卡在这里，动弹不得，CPU时间片被浪费
    System.out.println("主线程：终于拿到结果：" + result);
    System.out.println("主线程：结束。");
}
```

*   **主线程的状态**：**阻塞（Blocked）**。它的执行指针停在了 `queryDatabase()` 这一行，无法前进。它什么也做不了，只能干等。
*   **资源浪费**：这个线程虽然没在做计算，但它仍然占用了内存（线程栈）和操作系统资源。如果这是一个Web服务器的请求处理线程，那么这个线程在这5秒内无法处理任何其他新的请求。

**生活比喻**：你去银行办业务，柜员告诉你需要等5分钟。于是你**只能站在柜台前，盯着他，什么都不能做**，直到他办完。你的时间被完全占用了。

---

#### 2. 异步回调模式：主动的“等待”或“继续工作”

在异步模式下，主线程发起一个异步调用后，它**立即就返回了**，可以继续执行后面的代码。

```java
public static void main(String[] args) {
    System.out.println("主线程：已提交异步数据库查询...");

    CompletableFuture.supplyAsync(() -> queryDatabase()) // 任务被甩给后台线程池
        .thenAccept(result -> {
            System.out.println("回调线程：拿到结果：" + result);
        });

    System.out.println("主线程：不等待，我去做别的事情！比如响应UI事件、处理其他请求...");
    // ... 在这里，主线程可以执行成千上万行其他代码 ...
    // ... 比如更新UI、计算另一个数据、响应用户点击 ...

    // 在演示代码中，我们用 sleep 模拟主线程有其他工作要做，并防止程序提前退出
    // 在真实应用中，这里根本不会是 sleep()
    // Thread.sleep(6000); 
    System.out.println("主线程：我的其他工作都做完了，现在可以结束了。");
}
```

主线程完成后就就被释放，而异步任务会被其它线程执行，异步任务结束后会触发回调，将由另外一个线程执行回调。

*   **主线程的状态**：**非阻塞（Non-Blocked）/运行（Running）**。它提交任务后，立即继续执行 `System.out.println("主线程：不等待...")` 以及后续所有代码。它完全是自由的。
*   **资源高效利用**：主线程（或请求处理线程）可以立即去服务下一个任务。这极大地提高了应用的**吞吐量（Throughput）**。

**生活比喻**：你去餐厅点餐，服务员给了你一个**震动取餐器**（`CompletableFuture`）。你**不需要站在前台干等**。你可以回到座位上玩手机、和朋友聊天（做其他工作）。当饭菜好了，取餐器震动（回调被触发），你再去取餐。你的时间被解放了。

---

#### 关键区别总结：以 Web 服务器为例

这在 Web 服务器等高并发场景下，区别是**天壤之别**。

假设服务器有10个工作线程，一个请求需要5秒的数据库I/O。

*   **同步模式**：
    1.  第1个请求进来，占用线程1。线程1被**阻塞5秒**。
    2.  在这5秒内，如果又来了9个请求，它们会分别占用线程2到线程10，并全部被**阻塞**。
    3.  如果第11个请求进来，线程池已满，该请求只能排队等待，用户会感到明显的卡顿甚至超时。
    4.  **结论**：10个线程最多只能**同时处理10个**这种慢请求。

*   **异步模式**：
    1.  第1个请求进来，占用线程1。线程1提交一个异步DB查询后，**立即被释放**，回到线程池。
    2.  线程1可以**立刻去处理第2个请求**，同样提交异步查询后又被释放。
    3.  ...以此类推，这10个线程可以非常快速地接收并提交**成百上千个**请求的初始部分。真正的等待发生在后台的I/O操作上，而不是在宝贵的工作线程上。
    4.  当某个数据库查询完成后，线程池中的任何一个空闲线程都可以执行后续的回调（`thenAccept`），然后将结果返回给用户。
    5.  **结论**：10个线程可以轻松应对**远超10个**的并发请求，极大地提升了服务的吞吐能力和响应性。

### 结论

| 特性 | 同步模式 | 异步模式 (`CompletableFuture`) |
| :--- | :--- | :--- |
| **主线程（或调用线程）** | **被阻塞**，在原地等待任务完成。 | **不被阻塞**，提交任务后立即返回，可继续做其他事。 |
| **资源利用率** | **低**。线程在等待I/O时被浪费。 | **高**。线程被解放出来去处理更多任务。 |
| **程序吞吐量** | **低**。受限于阻塞任务的数量。 | **高**。能同时处理大量并发I/O密集型任务。 |
|

所以，虽然最终整个**程序（JVM进程）**都需要存活到异步任务结束，但**发起任务的那个线程**却早已被解放。这就是异步回调与传统同步的根本区别和巨大优势。

## 16. JMM

### 1）对Volatile 的理解

**Volatile** 是 Java 虚拟机提供 **轻量级的同步机制**


保证可见性 - 确保变量修改对所有线程立即可见
不保证原子性 - 不能保证复合操作的原子性
禁止指令重排 - 防止编译器和处理器对代码进行重排序优化

**如何实现可见性**

volatile变量修饰的共享变量在进行写操作的时候回多出一行汇编：

0x01a3de1d:movb $0×0，0×1104800（%esi）;0x01a3de24**:lock** addl $0×0,(%esp);

Lock前缀的指令在多核处理器下会引发两件事情。

1）将当前处理器缓存行的数据写回到系统内存。

2）这个写回内存的操作会使其他cpu里缓存了该内存地址的数据无效。

**多处理器总线嗅探：**

​    为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存后再进行操作，但操作不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是在**多处理器下**，为了保证各个处理器的缓存是一致的，就会实现缓存缓存一致性协议，**每个处理器通过嗅探在总线上传播的数据来检查自己的缓存值是不是过期了，如果处理器发现自己缓存行对应的内存地址呗修改，就会将当前处理器的缓存行设置无效状态**，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据库读到处理器缓存中。

### 2）什么是JMM？

JMM：JAVA内存模型，不存在的东西，是一个概念，也是一个约定！

**关于JMM的一些同步的约定：**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250705210039990.png)
1、线程解锁前，必须把共享变量**立刻**刷回主存；

2、线程加锁前，必须**读取主存**中的最新值到工作内存中；

3、加锁和解锁是同一把锁；

线程中分为 **工作内存、主内存**   

**8种操作**:

- **Read（读取）**：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用；

- **load（载入）**：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中；

- **Use（使用）**：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令；

- **assign（赋值）**：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中；

- **store（存储）**：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用；

- **write（写入）**：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中；

- **lock（锁定）**：作用于主内存的变量，把一个变量标识为线程独占状态；

- **unlock（解锁）**：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定；

  

  **JMM对这8种操作给了相应的规定**：

  - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
  - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
  - 不允许一个线程将没有assign的数据从工作内存同步回主内存
  - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是对变量实施use、store操作之前，必须经过assign和load操作
  - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
  - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
  - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  - 对一个变量进行unlock操作之前，必须把此变量同步回主内存


```java
package com.example.juc.testVolatile;
import java.util.concurrent.TimeUnit;

public class Demo {
    public static int num = 0;
    public static void main(String[] args) throws InterruptedException { // 主线程
        new Thread(()->{ // 子线程
            while(num == 0){
                // 线程1一直在循环，直到num不等于0
            }
        }).start();
        
        TimeUnit.SECONDS.sleep(1); // 主线程休眠1秒

        num = 1; // 主线程将num设置为1
    }
}

```
主线程修改了num的值，但是子线程并没有感知到这个变化，导致子线程一直在死循环。
需要让子线程知道主存中的num值已经被修改过了。

**程序不知道主存中的值已经被修改过了！**

## 17. volatile

### 1）保证可见性

```java
package com.example.juc.testVolatile;
import java.util.concurrent.TimeUnit;

public class Demo {
    // 不加volatile修饰符的变量, 导致线程1无法感知到主线程对num的修改
    public static int num = 0;
    public static void main(String[] args) throws InterruptedException { // 主线程
        new Thread(()->{ // 线程1
            while(num == 0){ // 线程1对主存的变化不知道
                
            }
        }).start();
        
        TimeUnit.SECONDS.sleep(1); // 主线程休眠1秒

        num = 1; // 主线程将num设置为1
        System.out.println(num);
    }
}

```

### 2）不保证原子性

原子性：不可分割；

线程A在执行任务的时候，不能被打扰的，也不能被分割的，要么同时成功，要么同时失败。

```java
package com.example.juc.testVolatile;

public class Demo02 {
    private  static volatile  int num = 0; // 使用volatile修饰符

    public static synchronized void add(){
        num++; // num++不是原子操作,读取变量 num 的当前值
        // 对值加 1
        // 将结果写回 num
    }

    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[2]; // 创建两个线程
        for(int i=0;i<2;i++){
            threads[i] = new Thread(()->{
                for(int j=0;j<1000;j++){
                    add(); // 每个线程对num进行1000次自增操作
                }
            });
            threads[i].start(); // 启动两个线程
        }
    // while(Thread.activeCount() > 2){ // 默认有main和gc两个线程
    //     Thread.yield(); // yield()方法让当前线程让出CPU执行权，但当前线程仍然会被调度，不会等待其他线程结束
        for(Thread thread : threads){
            thread.join(); // 等待两个线程执行完毕
        }
        // 输出最终的num值
    System.out.println("最终的num值: " + num);
    }
}

```
使用volatile而不在add方法上加入synchronized修饰符，num++操作就不是原子操作，可能会出现线程安全问题，导致最终的num值不等于2000。

* join和yield的区别：
- `join()`：等待线程执行完毕，当前线程会阻塞，直到被调用的线程执行完毕。
- `yield()`：让出CPU执行权，当前线程仍然会被调度，不会等待其他线程结束。

**如果不加lock或synchronized ，怎么样保证原子性？**



**使用原子类**

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250706095804993.png)
```java
public class VDemo02 {

    private static volatile AtomicInteger number = new AtomicInteger();

    public static void add(){
//        number++;
        number.getAndIncrement();  //底层是CAS保证的原子性
    }

    public static void main(String[] args) {
        //理论上number  === 20000

        for (int i = 1; i <= 20; i++) {
            new Thread(()->{
                for (int j = 1; j <= 1000 ; j++) {
                    add();
                }
            }).start();
        }

        while (Thread.activeCount()>2){
            //main  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+",num="+number);
    }
}
```

这些类的底层都直接和操作系统挂钩！是在内存中修改值。

Unsafe类是一个很特殊的存在；

> 原子类为什么这么高级？

### 3）禁止指令重排

**什么是指令重排？**

我们写的程序，计算机并不是按照我们自己写的那样去执行的

源代码–>编译器优化重排–>指令并行也可能会重排–>内存系统也会重排–>执行

**处理器在进行指令重排的时候，会考虑数据之间的依赖性！**

```java
int x=1; //1
int y=2; //2
x=x+5;   //3
y=x*x;   //4

//我们期望的执行顺序是 1_2_3_4  可能执行的顺序会变成2134 1324
//可不可能是 4123？ 不可能的
1234567
```

可能造成的影响结果：前提：a b x y这四个值 默认都是0

| 线程A | 线程B |
| ----- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

正常的结果： x = 0; y =0;

但是如果指令重排，可能会出现以下情况：
因为在线程A中，x和b没有数据依赖关系
线程B中，y和a没有数据依赖关系

| 线程A | 线程B |
| ----- | ----- |
| b=1   | a=2   |
| x=a   | y=b   |

可能在线程A中会出现，先执行b=1,然后再执行x=a；

在B线程中可能会出现，先执行a=2，然后执行y=b；

那么就有可能结果如下：x=2; y=1.

**volatile可以避免指令重排：**

**volatile中会加一道内存的屏障，这个内存屏障可以保证在这个屏障中的指令顺序。**

内存屏障：CPU指令。作用：

1、保证特定的操作的执行顺序；

2、可以保证某些变量的内存可见性（利用这些特性，就可以保证volatile实现的可见性）

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250706101747051.png)

当你在变量前加上 volatile 关键字时，JVM 会在对该变量的读写操作前后插入内存屏障（Memory Barrier）。内存屏障是一种 CPU 指令，用于限制指令的执行顺序。



好的，我们来对本次对话进行一个清晰的总结。

---

#### 本次对话总结

我们通过一张经典的并发问题图片，深入探讨了`volatile`关键字如何通过内存屏障解决指令重排问题。

##### 1. 问题根源：指令重排 (Instruction Reordering)

*   **场景**: 在一个多线程程序中（如图片所示的线程A和线程B），变量 `a, b, x, y` 初始值为0。
    *   **线程A**: `x = a; b = 1;`
    *   **线程B**: `y = b; a = 2;`
*   **问题**: 为了提高性能，编译器和CPU在不影响**单线程**结果的情况下，可能会对没有数据依赖的指令进行重排。
    *   线程A的指令可能被重排为: `b = 1;` -> `x = a;`
    *   线程B的指令可能被重排为: `a = 2;` -> `y = b;`
*   **后果**: 这种重排可能导致出现一个在逻辑上看似不可能的结果：`x = 2` 且 `y = 1`。

##### 2. `volatile` 的解决方案

*   将共享变量 `a` 和 `b` 声明为 `volatile` (`volatile int a = 0; volatile int b = 0;`)。
*   `volatile` 关键字能解决这个问题，主要依靠它的两大作用：
    1.  **保证可见性**: 一个线程对`volatile`变量的修改，对其他线程是立即可见的。
    2.  **禁止指令重排**: 这是解决本问题的核心。

##### 3. 核心机制：内存屏障 (Memory Barrier)

*   `volatile` 通过在底层操作中插入**内存屏障**来实现禁止指令重排。
*   内存屏障就像一道“栅栏”，它规定了：
    *   栅栏前的所有读写操作必须全部完成。
    *   才能执行栅栏后的读写操作。
    *   指令不能被重排到栅栏的另一边。
*   **应用到本例中**:
    *   当 `b` 是 `volatile` 时，`b = 1;` (volatile写) 会受到内存屏障的保护，它不能被重排到 `x = a;` 之前。
    *   同理，`a = 2;` (volatile写) 也不能被重排到 `y = b;` 之前。
    *   因此，两个线程的执行顺序都得到了保证，从而避免了异常结果的发生。

##### 4. 关键点澄清 (针对 `x=a` 和 `y=b` 的提问)

*   `volatile` 的内存屏障规则是围绕**被`volatile`修饰的变量**的访问来建立的，而不是赋值符号`=`本身。
*   在 `x = a;` 这个操作中，虽然最终是**写**局部变量 `x`，但它首先需要**读取`volatile`变量`a`**。
*   正是这个对`volatile`变量`a`的**读操作**，触发了内存屏障机制，使得该语句的执行顺序被固定，不能随意移动。
*   同理，`y = b;` 中的对`volatile`变量`b`的**读操作**也受到了保护。

**最终结论**: `volatile`关键字是解决并发编程中变量可见性和指令重排问题的重要工具。它通过插入内存屏障，强制规定了内存操作的顺序，确保了多线程程序的执行结果与程序员的预期一致。


![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250706103638282.png)
### 4）总结

- **volatile可以保证可见性；**
- **不能保证原子性**
- **由于内存屏障，可以保证避免指令重排的现象产生**

面试官：那么你知道在哪里用这个内存屏障用得最多呢？**单例模式**



## 18. 玩转单例模式

### 什么是单例模式
单例模式是一种非常常见的设计模式，它的核心思想很简单：确保一个类在任何情况下都绝对只有一个实例，并提供一个全局的访问点来获取这个实例。
你可以把它想象成一个国家只有一个总统，或者一个公司只有一个 CEO。无论你在公司的哪个部门，当你需要向 CEO 汇报时，你找到的都是同一个人，而不是每次都新“创建”一个 CEO。

### 单例模式的好处
- 节省资源：对于一些需要频繁创建和销毁的对象，比如数据库连接池、线程池等，创建一个实例的开销很大。单例模式可以确保只有一个实例在内存中，从而节省系统资源。
- 保证结果正确性：当一个类的功能需要依赖某些共享的状态或数据时，单例可以确保所有操作都作用于同一个实例，避免了数据不一致的问题。例如，一个网站的计数器，如果每次都 new 一个新对象，计数就永远无法累加。
- 提供全局访问点：可以方便地在程序的任何地方访问这个唯一的实例，就像一个全局变量，但比全局变量更安全、更易于管理。

### 如何实现单例模式
- **私有化构造函数** (Private Constructor)：
目的是为了防止外部通过 new 关键字随意创建对象。既然要保证唯一，就不能让别人随便创建。

- **在类内部创建私有的静态实例** (Private Static Instance)：
在类的内部自己创建一个实例。因为是 static 的，所以它与类绑定，是唯一的。

- **提供一个公有的静态方法** (Public Static Method)：
这个方法是外界获取这个唯一实例的入口，通常命名为 getInstance()。

**常见的实现方式**

### 1）饿汉式
迫不及待，类加载的时候就把实例常见好，不管用不用
- **优点**：实现简单，代码清晰。由于实例是在类加载时创建的，由 JVM 保证了线程安全，不存在多线程同步问题。
- **缺点**：如果这个实例从未使用过，会造成内存浪费。它不是懒加载（Lazy Loading）。

```java
// 饿汉式：线程安全，但可能造成资源浪费
public class Singleton {

    // 1. 在类加载时就立即创建实例
    private static final Singleton INSTANCE = new Singleton();

    // 2. 私有化构造函数
    private Singleton() {}

    // 3. 提供公有的获取实例的方法
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### 2）DCL懒汉式
"懒汉" 就是非常懒，只有在第一次被用到的时候才去创建实例。
```java
//懒汉式单例模式
package com.example.juc.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class LazyDoubleCheck {
    // 静态变量 instance 用于存储单例实例，statci与类绑定
    private static LazyDoubleCheck instance;

    private LazyDoubleCheck() {
        // 私有构造函数，防止外部实例化

        synchronized (LazyDoubleCheck.class) {
    
        }
    }
    // 双重检查锁定（Double-Checked Locking）实现单例模式，避免每次都要同步锁，提高性能
    public static LazyDoubleCheck getInstance() {
        // 第一次检查 instance 是否为 null
        if (instance == null) {
           synchronized (LazyDoubleCheck.class) {
                // 第二次检查 instance 是否为 null
                if (instance == null) {
                    // 如果仍然为 null，则创建新的实例
                    instance = new LazyDoubleCheck();
                    /**
                     *  instance = new LazyDoubleCheck();并非原子操作
                     *  大概分为三步：：1. 分配内存；2. 初始化对象；3. 将 instance 引用指向分配的内存。
                     *  如果没有volatile修饰，可能会发生指令重排，导致第三步在第二步之前执行，此时另外一个线程发现instance不为null
                     *  但实际上对象还没有初始化完成，使用就会出错i。volatile可以防止这种重排
                     */
                }
            }
        } 
        return instance;
    }

}

```

反射可以获取类的构造方法，并改变构造方法的权限从而创建实例。这个时候就需要在构造方法中判断当前实例是否已经存在，如果存在则抛出异常。

```java
package com.example.juc.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class LazyDoubleCheck {
    // 静态变量 instance 用于存储单例实例，statci与类绑定
    private static LazyDoubleCheck instance;

    private LazyDoubleCheck() {
        // 私有构造函数，防止外部实例化

        synchronized (LazyDoubleCheck.class) {
            // 为了防止使用反射破坏单例模式，检查 instance 是否为 null
            if (instance != null) {
                throw new RuntimeException("不要试图利用反射破坏单例模式");
            }
        }
    }
    // 双重检查锁定（Double-Checked Locking）实现单例模式，避免每次都要同步锁，提高性能
    public static LazyDoubleCheck getInstance() {
        // 第一次检查 instance 是否为 null
        if (instance == null) {
           synchronized (LazyDoubleCheck.class) {
                // 第二次检查 instance 是否为 null
                if (instance == null) {
                    // 如果仍然为 null，则创建新的实例
                    instance = new LazyDoubleCheck();
                    /**
                     *  instance = new LazyDoubleCheck();并非原子操作
                     *  大概分为三步：：1. 分配内存；2. 初始化对象；3. 将 instance 引用指向分配的内存。
                     *  如果没有volatile修饰，可能会发生指令重排，导致第三步在第二步之前执行，此时另外一个线程发现instance不为null
                     *  但实际上对象还没有初始化完成，使用就会出错i。volatile可以防止这种重排
                     */
                }
            }
        } 
        return instance;
    }

    // 反射可以破解单例模式
    public static void main(String[] args) throws Exception {
        LazyDoubleCheck instance1 = LazyDoubleCheck.getInstance();
        // 获取空参构造器
        Constructor<LazyDoubleCheck> declaredConstructor = LazyDoubleCheck.class.getDeclaredConstructor();

        // 设置可访问性
        declaredConstructor.setAccessible(true);

        // 通过反射创建实例
        LazyDoubleCheck instance2 = declaredConstructor.newInstance();
        System.out.println(instance1 == instance2); // 输出 false，说明是不同的实例
    }

}


```
现在从双重检测升级了三重检测
但如果从始至终都使用getInstance()方法获取实例，只用反射创建实例，三重检测就会失效。

```java
public static void main(String[] args) throws Exception {
        // LazyDoubleCheck instance1 = LazyDoubleCheck.getInstance();
        // 获取空参构造器
        Constructor<LazyDoubleCheck> declaredConstructor = LazyDoubleCheck.class.getDeclaredConstructor();

        // 设置可访问性
        declaredConstructor.setAccessible(true);

        // 通过反射创建实例
        LazyDoubleCheck instance2 = declaredConstructor.newInstance();

        LazyDoubleCheck instance3 = declaredConstructor.newInstance();
        System.out.println(instance2 == instance3); // 输出 false，说明是不同的实例
    }
```
**两个实例都是利用反射创建的对象**


**引入变量确保构造方法只能构造一次**
```java
package com.example.juc.single;

import java.lang.reflect.Constructor;


public class LazyDoubleCheck {
    // 静态变量 instance 用于存储单例实例，statci与类绑定
    private static LazyDoubleCheck instance;
    // 引入一个变量，确保构造方法只能被调用一次
    private static boolean flag = false;

    private LazyDoubleCheck() {
        // 私有构造函数，防止外部实例化
        // 为了防止使用反射破坏单例模式，检查 instance 是否为 null
        synchronized (LazyDoubleCheck.class) {
            // 为了防止使用反射破坏单例模式，检查 instance 是否为 null
            if(flag == false) {
                flag = true;
            } else {
                // 如果已经被实例化，则抛出异常
                throw new RuntimeException("不要试图利用反射破坏单例模式");
            }
            
        }
    }
    // 双重检查锁定（Double-Checked Locking）实现单例模式，避免每次都要同步锁，提高性能
    public static LazyDoubleCheck getInstance() {
        // 第一次检查 instance 是否为 null
        if (instance == null) {
           synchronized (LazyDoubleCheck.class) {
                // 第二次检查 instance 是否为 null
                if (instance == null) {
                    // 如果仍然为 null，则创建新的实例
                    instance = new LazyDoubleCheck();
                    /**
                     *  instance = new LazyDoubleCheck();并非原子操作
                     *  大概分为三步：：1. 分配内存；2. 初始化对象；3. 将 instance 引用指向分配的内存。
                     *  如果没有volatile修饰，可能会发生指令重排，导致第三步在第二步之前执行，此时另外一个线程发现instance不为null
                     *  但实际上对象还没有初始化完成，使用就会出错i。volatile可以防止这种重排
                     */
                }
            }
        } 
        return instance;
    }

    // 反射可以破解单例模式
    public static void main(String[] args) throws Exception {
        // LazyDoubleCheck instance1 = LazyDoubleCheck.getInstance();
        // 获取空参构造器
        Constructor<LazyDoubleCheck> declaredConstructor = LazyDoubleCheck.class.getDeclaredConstructor();

        // 设置可访问性
        declaredConstructor.setAccessible(true);

        // 通过反射创建实例
        LazyDoubleCheck instance2 = declaredConstructor.newInstance();

        LazyDoubleCheck instance3 = declaredConstructor.newInstance();
        System.out.println(instance2 == instance3); // 输出 false，说明是不同的实例
    }

}

```
加密就会有解密，如果别人知道我们的变量，依旧可以利用反射破坏单例模式

```java
public static void main(String[] args) throws Exception {
        // LazyDoubleCheck instance1 = LazyDoubleCheck.getInstance();
        // 获取空参构造器
        Constructor<LazyDoubleCheck> declaredConstructor = LazyDoubleCheck.class.getDeclaredConstructor();


        // 设置可访问性
        declaredConstructor.setAccessible(true);

        // 获取字段
        Field declaredField = LazyDoubleCheck.class.getDeclaredField("flag");
        // 设置可访问性
        declaredField.setAccessible(true);

        // 通过反射创建实例
        LazyDoubleCheck instance2 = declaredConstructor.newInstance();

        declaredField.set(instance2,false); // 设置 flag 为 false

        LazyDoubleCheck instance3 = declaredConstructor.newInstance();
        System.out.println(instance2 == instance3); // 输出 false，说明是不同的实例
    }
```

道高一尺，魔高一丈
        
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250706161641757.png)      
java的反射不能破坏枚举的单例模式  

### 3）静态内部类

> 静态内部类实现单例模式仍然会被反射破坏
```java
//静态内部类
public class Holder {
    private Holder(){

    }
    public static Holder getInstance(){
        return InnerClass.holder;
    }
    public static class InnerClass{
        private static final Holder holder = new Holder();
    }
}
```
同样可以使用反射得到无参构造方法并修改权限获取实例
> 单例不安全, 因为反射

### 4）枚举

> Java 的枚举类型（enum）天生就是单例模式。
每个枚举常量（如 INSTANCE）在 JVM 中只会被实例化一次，并且由 Java 保证线程安全和反序列化时的单例性。因此，使用枚举实现单例模式是最简单且最安全的方式之一。

```java
//enum 是什么？ enum本身就是一个Class 类
package com.example.juc.single;
// 枚举本身也是一个Class类
// 枚举类默认就是单例模式
// 每个枚举常量在JVM中只会被实例化一次
public enum EnumSingle {
    INSTANCE;
}
class Test{
    public static void main(String[] args) {
        EnumSingle instance1 = EnumSingle.INSTANCE;

        EnumSingle instance2 = EnumSingle.INSTANCE;
        System.out.println(instance1 == instance2); // 输出 true，说明是同一个实例
    }
}

```

查看枚举类的编译后的字节码文件
```java
// Source code is decompiled from a .class file using FernFlower decompiler.
package com.example.juc.single;

public enum EnumSingle {
   INSTANCE;

   private EnumSingle() {
   }
}

```
可以看到里面有一个私有的无参构造方法
尝试用反射获取无参构造方法

```java
package com.example.juc.single;

import java.lang.reflect.Constructor;

public enum EnumSingle {
    INSTANCE;
}
class Test{
    public static void main(String[] args)  throws Exception {
        EnumSingle instance1 = EnumSingle.INSTANCE;

        // EnumSingle instance2 = EnumSingle.INSTANCE;
        // System.out.println(instance1 == instance2); // 输出 true，说明是同一个实例

        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        // 通过反射创建实例
        EnumSingle instance2 = declaredConstructor.newInstance();

        System.out.println(instance1);
        System.out.println(instance2);

    }
}

```
报错如下
`Exception in thread "main" java.lang.NoSuchMethodException: com.example.juc.single.EnumSingle.<init>()
        at java.base/java.lang.Class.getConstructor0(Class.java:3585)
        at java.base/java.lang.Class.getDeclaredConstructor(Class.java:2754)
        at com.example.juc.single.Test.main(EnumSingle.java:18)`

意思是枚举类型没有这个空参构造方法

只要有空参的构造方法的都是骗了我们

使用专业工具jad将class文件反编译为java文件



枚举类型的最终反编译源码：

```java
public final class EnumSingle extends Enum
{

    public static EnumSingle[] values()
    {
        return (EnumSingle[])$VALUES.clone();
    }

    public static EnumSingle valueOf(String name)
    {
        return (EnumSingle)Enum.valueOf(com/ogj/single/EnumSingle, name);
    }

    private EnumSingle(String s, int i)
    {
        super(s, i);
    }

    public EnumSingle getInstance()
    {
        return INSTANCE;
    }

    public static final EnumSingle INSTANCE;
    private static final EnumSingle $VALUES[];

    static 
    {
        INSTANCE = new EnumSingle("INSTANCE", 0);
        $VALUES = (new EnumSingle[] {
            INSTANCE
        });
    }
}
```
可以看到里面的构造器是有参构造器

那么使用有参构造器获取实例化会怎么样呢
```java
package com.example.juc.single;

import java.lang.reflect.Constructor;

// 枚举本身也是一个Class类
// 枚举类默认就是单例模式
// 每个枚举常量在JVM中只会被实例化一次
public enum EnumSingle {
    INSTANCE;
}
class Test{
    public static void main(String[] args)  throws Exception {
        EnumSingle instance1 = EnumSingle.INSTANCE;

        // EnumSingle instance2 = EnumSingle.INSTANCE;
        // System.out.println(instance1 == instance2); // 输出 true，说明是同一个实例

        // Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(null);
        // 获取有参构造器
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class, int.class);
        declaredConstructor.setAccessible(true);
        // 通过反射创建实例
        EnumSingle instance2 = declaredConstructor.newInstance();

        System.out.println(instance1);
        System.out.println(instance2);

    }
}

```
结果如下
```plaintext
Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects
        at java.base/java.lang.reflect.Constructor.newInstanceWithCaller(Constructor.java:493)
        at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:481)
        at com.example.juc.single.Test.main(EnumSingle.java:23)
```
意思是不能反射创建枚举对象,正是我我们之前在newInstance方法中见到的，不能使用反射破坏枚举的单例模式。


#### 枚举类单例模式实用场景

1. **配置管理**：如前面提到的 `AppConfig`，用于加载和提供应用的配置信息。
2. **日志管理**：一个全局的日志管理器，负责记录应用的日志信息。
3. **线程池管理**：一个全局的线程池，供整个应用共享使用。
4. **数据库连接池**：一个全局的数据库连接池，管理数据库连接的创建和释放。


在实际开发中，我们通常需要单例来管理一些共享的资源或状态。下面通过一个具体的例子来说明。

好的，完全没有问题。

这是一个使用**枚举类**创建**线程池单例**的伪代码，它清晰地展示了其核心结构和思想。

#### 场景：全局异步任务处理器

在应用中，我们需要一个全局唯一的线程池来处理各种异步任务，如发送邮件、记录日志、处理文件等，以避免重复创建线程带来的资源浪费。

---

##### 1. 线程池单例的伪代码

```
// 使用枚举定义线程池管理器，这本身就是单例
ENUM ThreadPoolManager:
    
    // 这是唯一的实例。JVM会保证它只被创建一次。
    INSTANCE

    // 私有成员变量，用于持有真正的线程池对象
    PRIVATE FINAL thread_pool

    // 构造函数：在INSTANCE被创建时自动调用，且仅调用一次
    CONSTRUCTOR ThreadPoolManager():
        // 1. 定义线程池的配置参数
        core_threads = 4
        max_threads = 10
        task_queue = new Queue(capacity: 100)

        // 2. 创建线程池实例
        this.thread_pool = CREATE_THREAD_POOL(
            core_size: core_threads,
            max_size: max_threads,
            queue: task_queue
        )
        
        // 3. (可选但推荐) 注册一个程序关闭时的钩子，用于优雅地关闭线程池
        REGISTER_SHUTDOWN_HOOK(FUNCTION() {
            PRINT "Application shutting down, closing thread pool..."
            this.thread_pool.SHUTDOWN_GRACEFULLY()
        })

    // 公共方法：提供给外部提交任务的接口
    PUBLIC FUNCTION execute(task):
        IF task IS NOT NULL:
            this.thread_pool.SUBMIT(task)
        ELSE:
            PRINT_ERROR "Submitted task is null."

END ENUM
```

##### 2. 在应用不同模块中使用该单例的伪代码

**用户服务模块 (UserService):**

```
CLASS UserService:
    
    FUNCTION register_user(email):
        // 同步执行数据库插入
        SAVE_USER_TO_DB(email)
        
        // 创建一个异步任务：发送欢迎邮件
        send_email_task = FUNCTION():
            SLEEP(2 seconds) // 模拟耗时
            SEND_EMAIL(to: email, subject: "Welcome!")
        
        // 将任务提交给全局唯一的线程池
        ThreadPoolManager.INSTANCE.execute(send_email_task)
        
        PRINT "User registration complete. Welcome email will be sent in background."

END CLASS
```

**文件处理模块 (FileProcessor):**

```
CLASS FileProcessor:

    FUNCTION process_file(file_path):
        // 创建一个异步任务：处理文件
        process_file_task = FUNCTION():
            SLEEP(5 seconds) // 模拟耗时
            COMPRESS_FILE(file_path)
            GENERATE_THUMBNAIL(file_path)

        // 同样将任务提交给那个全局唯一的线程池
        ThreadPoolManager.INSTANCE.execute(process_file_task)
        
        PRINT "File processing started in background for: " + file_path

END CLASS
```

#####  3. 主程序流程伪代码

```
FUNCTION main():
    user_service = new UserService()
    file_processor = new FileProcessor()
    
    // 触发用户注册，这将向线程池提交一个任务
    // 在这里，ThreadPoolManager的构造函数会被第一次调用，并创建线程池
    user_service.register_user("test@example.com")
    
    // 触发文件处理，向同一个线程池提交另一个任务
    // 此时，直接复用已存在的线程池，构造函数不会再次执行
    file_processor.process_file("/path/to/my_report.pdf")
    
    PRINT "Main thread is free to do other work."
    // ...主线程继续执行...

END FUNCTION
```

---

##### 伪代码核心思想

1.  **`ThreadPoolManager.INSTANCE`** 是全局唯一的访问点。
2.  **构造函数只执行一次**，在其中完成线程池的复杂初始化和配置。
3.  任何模块（如 `UserService`、`FileProcessor`）都通过 `ThreadPoolManager.INSTANCE.execute()` 来提交任务，它们共享着同一个线程池资源。
4.  资源的管理（如创建和关闭）被封装在单例内部，对使用者透明，降低了使用复杂度。
#### 总结

*   `INSTANCE` 是你为这个**唯一的、全局可访问的枚举实例**起的名字。
*   在实际开发中，你将**状态和行为**（即成员变量和方法）添加到这个枚举类中。
*   通过 `你的枚举类名.INSTANCE` 这种方式在程序的任何地方安全地、方便地访问这个唯一的实例及其方法。
*   这种方式是实现单例的**最推荐、最安全、最简洁**的方式。

## 19. 深入理解CAS

### 1）什么是CAS？

大厂必须深入研究底层！！！！**修内功！操作系统、计算机网络原理、组成原理、数据结构**

```java
public class casDemo {
    //// CAS（Compare-And-Set）操作：比较当前工作内存的值和主内存的值，若这个值是期望则执行，如果不是就一直循环
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        //boolean compareAndSet(int expect, int update)
        //期望值、更新值
        //如果实际值 和 我的期望值相同，那么就更新
        //如果实际值 和 我的期望值不同，那么就不更新
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        //因为期望值是2020  实际值却变成了2021  所以会修改失败
        //CAS 是CPU的并发原语
        atomicInteger.getAndIncrement(); //++操作
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
    }
}
```
Unsafe 类
进入compareAndSet方法
查看U发现是Unsafe类
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708102609039.png)

查看Unsafe类中的compareAndSetInt方法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708103103123.png)
    
compareAndSetInt方法是内存操作，效率很高

getAndIncrement()
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708104145218.png)

可以看到不能更新成功就一直循环，这就是自旋锁


### 2）总结

CAS：比较当前工作内存中的值 和 主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就一直循环，使用的是自旋锁。

**缺点：**

- 循环会耗时；
- 一次性只能保证一个共享变量的原子性；
- 它会存在ABA问题

> CAS：ABA问题？(狸猫换太子)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708143943997.png)
线程1：期望值是1，要变成2；

线程2：两个操作：

- 1、期望值是1，变成3
- 2、期望是3，变成1

线程2先把两个操作完成，线程1期望是1，现在也是1，但不是原来的1，而是线程2操作过的1；
即线程1被骗过了




```java
public class casDemo {
    //CAS : compareAndSet 比较并交换
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());

        //boolean compareAndSet(int expect, int update)
        //期望值、更新值
        //如果实际值 和 我的期望值相同，那么就更新
        //如果实际值 和 我的期望值不同，那么就不更新
        System.out.println(atomicInteger.compareAndSet(2021, 2020));
        System.out.println(atomicInteger.get());

        //因为期望值是2020  实际值却变成了2021  所以会修改失败
        //CAS 是CPU的并发原语
//        atomicInteger.getAndIncrement(); //++操作
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
    }
}
```



## 20. 原子引用

> 解决ABA问题，对应的思想：就是使用了**乐观锁~**

带版本号的 原子操作！

其实就是乐观锁的思路，判断修改的时候判断版本号即可


**Integer 使用了对象缓存机制，默认范围是-128~127，推荐使用静态工厂方法valueOf获取对象实例，而不是new，因为valueOf使用缓存，而new一定会创建新的对象分配新的内存空间。**

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708152324295.png)

**带版本号的原子操作**

```java
package com.example.juc.cas;

import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.TimeUnit;
public class CasDemo02 {
    public static void main(String[] args) throws Exception {
        AtomicStampedReference <Integer> atomicReference = new AtomicStampedReference<>(1, 1);

        new Thread(()->{
            int stamp = atomicReference.getStamp(); // a1 = 1
            System.out.println("a1=>" + stamp);
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟线程1等待
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result1 = atomicReference.compareAndSet(1, 3, atomicReference.getStamp(), atomicReference.getStamp()+1);
            System.out.println("a2=>"+ atomicReference.getStamp()); // a2 = 2
            System.out.println("result1"+result1); // true

            boolean result2=atomicReference.compareAndSet(3, 1, atomicReference.getStamp(), atomicReference.getStamp()+1);
            System.out.println("result2"+result2); // true
            System.out.println("a3=>"+atomicReference.getStamp());//a3 = 3
            
        },"a").start();

        new Thread(()->{
            int stamp = atomicReference.getStamp();
            System.out.println("b1=>"+stamp); // b1 = 1
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (Exception e) {
                e.printStackTrace();
            }
            // 最初的版本号stamp是1，但当线程a修改了值后，版本号变成了3，导致与期望版本号1不一致则修改失败
            boolean result = atomicReference.compareAndSet(1, 5, stamp, stamp+1);
            System.out.println("result"+result); // false
            System.out.println(atomicReference.getStamp());// 3
        },"b").start();
    }   
}
    
```
**坑点** 

如果我把值修初始值修改为很大，那么修改结果将都为false
```java
package com.example.juc.cas;

import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.TimeUnit;
public class CasDemo02 {
    public static void main(String[] args) throws Exception {
        AtomicStampedReference <Integer> atomicReference = new AtomicStampedReference<>(2021, 1);

        new Thread(()->{
            int stamp = atomicReference.getStamp(); // a1 = 1
            System.out.println("a1=>" + stamp);
            try {
                TimeUnit.SECONDS.sleep(1); // 模拟线程1等待
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result1 = atomicReference.compareAndSet(2021, 2022, atomicReference.getStamp(), atomicReference.getStamp()+1);
            System.out.println("a2=>"+ atomicReference.getStamp()); // a2 = 2
            System.out.println("result1"+result1); // true

            boolean result2=atomicReference.compareAndSet(2022, 2021, atomicReference.getStamp(), atomicReference.getStamp()+1);
            System.out.println("result2"+result2); // true
            System.out.println("a3=>"+atomicReference.getStamp());//a3 = 3
            
        },"a").start();

        new Thread(()->{
            int stamp = atomicReference.getStamp();
            System.out.println("b1=>"+stamp); // b1 = 1
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (Exception e) {
                e.printStackTrace();
            }
            boolean result = atomicReference.compareAndSet(2021, 2023, stamp, stamp+1);
            System.out.println("result"+result); // false
            System.out.println(atomicReference.getStamp());// 3
        },"b").start();
    }   
}
    
```
**原因分析**
首先我们当前的引用的对象是Integer
查看源码
compareAndSet
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708160257018.png)

使用的是 **==** 进行比较，对于基本数据类型，比较的是值，对于引用类型，比较的是地址值

Integer是引用数据类型，并且存在缓存机制。

当Integer的范围处于-128~127之间时，会触发缓存机制，并不会创建新的对象。

而我们的值是2021和2022，每次都是新的对象，所以会比较失败。

因此实际中，通常不使用直接的包装类型作为泛型，而是自定义包装类型，并在原对象中操作。

## 21. 各种锁的理解

### 1）公平锁，非公平锁

1. 公平锁：非常公平，不能插队，必须先来后到
```java
/**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```


2. 非公平锁：非常不公平，允许插队，可以改变顺序

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}
```

synchronized 和 ReentrantLock 默认都是非公平锁。

### 2）可重入锁(递归锁)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708162134105.png)

可重入锁是指同一个线程可以多次获取同一把锁而不会发生死锁的情况。
上锁与解锁成对出现


1. Synchonized 锁

```java
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();
        },"A").start();
        new Thread(()->{
            phone.sms();
        },"B").start();
    }

}

class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName()+"=> sms");
        call();//这里也有一把锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName()+"=> call");
    }
}
```
运行结果
```
A=> sms
A=> call
B=> sms
B=> call
```

2. Lock 锁

```java
//lock
public class Demo02 {

    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(()->{
            phone.sms();
        },"A").start();
        new Thread(()->{
            phone.sms();
        },"B").start();
    }

}
class Phone2{

    Lock lock=new ReentrantLock();

    public void sms(){
        lock.lock(); //细节：这个是两把锁，两个钥匙
        //lock锁必须配对，否则就会死锁在里面
        try {
            System.out.println(Thread.currentThread().getName()+"=> sms");
            call();//这里也有一把锁
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public void call(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "=> call");
        }catch (Exception e){
            e.printStackTrace();
        }
        finally {
            lock.unlock();
        }
    }
}
```

- lock锁必须配对，相当于lock和 unlock 必须数量相同；
- 在外面加的锁，也可以在里面解锁；在里面加的锁，在外面也可以解锁；

### 3）自旋锁

> 一直尝试直到成功

1. spinlock

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

2. 自我设计自旋锁

```java
public class SpinlockDemo {

    // 默认
    // int 0
    //thread null
    AtomicReference<Thread> atomicReference=new AtomicReference<>();

    //加锁 ,使用线程作为锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"===> mylock");

        //AtomicReference的compareAndSet方法是比较值
        boolean lockAcquired = atomicReference.compareAndSet(null, thread);
        while(!lockAcquired){
            //如果当前线程没有获取到锁，就一直自旋等待
            System.out.println(thread.getName()+"===> 自旋等待");
            lockAcquired = atomicReference.compareAndSet(null, thread);
        }
        System.out.println(thread.getName()+"===> 获取锁成功");
    }


    //解锁，将当前线程从atomicReference中移除
    public void myUnlock(){
        Thread thread=Thread.currentThread();
        System.out.println(thread.getName()+"===> myUnlock");
        atomicReference.compareAndSet(thread,null);
    }

}
```

```java
public class TestSpinLock {
    public static void main(String[] args) throws InterruptedException {
        ReentrantLock reentrantLock = new ReentrantLock();
        reentrantLock.lock();
        reentrantLock.unlock();


        //使用CAS实现自旋锁
        SpinlockDemo spinlockDemo=new SpinlockDemo();
        new Thread(()->{
            spinlockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        },"t1").start();

        TimeUnit.SECONDS.sleep(1);


        new Thread(()->{
            spinlockDemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinlockDemo.myUnlock();
            }
        },"t2").start();
    }
}
```

### 4）死锁
在多线程编程中，我们为了防止多线程竞争共享资源而导致数据错乱，都会在操作共享资源之前加上互斥锁，只有成功获得到锁的线程，才能操作共享资源，获取不到锁的线程就只能等待，直到锁被释放
当两个线程为了保护两个不同的共享资源而使用了两个互斥锁，那么这两个互斥锁应用不当的时候，可能会造成两个线程都在等待对方释放锁，在没有外力的作用下，这些线程会一直相互等待，就没办法继续运行，这种情况就是发生了死锁。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708170802743.png)

```java
package com.example.juc.lock;

public class DeadLock {
    public static void main(String[] args) {
        Object lock1 = new Object();
        Object lock2 = new Object();

        MyThread t1 = new MyThread("Thread-1", lock1, lock2);
        MyThread t2 = new MyThread("Thread-2", lock2, lock1);

        t1.start();
        t2.start();
    }

}
class MyThread extends Thread {
    private String name;
    private Object lock1;
    private Object lock2;

    public MyThread(String name, Object lock1, Object lock2) {
        this.name = name;
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    @Override
    public void run() {
        synchronized (lock1) {
            System.out.println(name + " acquired lock on " + lock1);
            try {
                Thread.sleep(100); // 模拟一些工作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lock2) {
                System.out.println(name + " acquired lock on " + lock2);
            }
        }
    }
}
```

如何解开死锁

**1、使用jps定位进程号，jdk的bin目录下： 有一个jps**

命令：`jps -l`

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708172924688.png)

**2、使用`jstack` 进程进程号 找到死锁信息**


**一般情况信息在最后：**

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250708172950308.png)


**日志**


**堆栈信息**