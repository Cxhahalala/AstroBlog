---
title: "JVM 面试题核心汇总与实战解析"
published: 2026-08-23
description: "全面整理 JVM 核心面试题：JVM 内存组成（堆、栈、方法区/元空间）、类加载机制与双亲委派、垃圾回收算法与 GC 收集器、JVM 调优参数与 OOM/CPU 飙高排查思路。"
tags: ["Java", "JVM", "面试"]
category: "Java"
draft: false
---

# 导学
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822192312681.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822192506147.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822192636671.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822194059258.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822194128811.png)
# Jvm组成
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822194403986.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822195320673.png)
如图所示，线程1从0行开始运行，当运行到20行后，cpu时间片分给线程二，那么线程二也从0行开始执行，当时间片再次分给1的时候，线程一会从20行继续执行。程序计数器就是让线程记住当前执行的行号。

**什么是程序计数器？** 

> 线程私有的，每个线程一份，内部保存的`字节码的行号`。用于记录正在执行的字节码指令的地址。

**什么是字节码行号？**

> 编译后的 `.class` 字节码文件中，每条字节码指令在方法中的**偏移量地址（Offset）**，在 JVM 中通常被称为字节码行号或指令地址。
>
> 💡 **注意区分：源码行号 vs 字节码行号**：
> - **源码行号**：我们在 `.java` 源文件中看到的行号（如第 10 行 `int a = 1;`）。
> - **字节码行号（指令偏移量）**：一行 Java 源码通常会被编译成多条字节码指令。例如通过 `javap -c` 反编译后的指令：
>   ```
>   0: iconst_1        // 0 就是字节码指令偏移量（行号）
>   1: istore_1        // 1 就是下一条指令偏移量
>   2: return          // 2 代表 return 指令
>   ```
> - **程序计数器记录的正是这里的 `0, 1, 2` 等字节码指令偏移量**，用于指示下一条要执行的 JVM 字节码指令。

## 介绍Java的堆
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822200758615.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822200927323.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201052441.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201411464.png)

### 1. 什么是 Java 堆（Heap）？
> 💡 **核心特征**：
> - **线程共享**：所有线程共享同一个堆内存空间，在 JVM 启动时创建。
> - **主要作用**：主要用于保存 **对象实例**、**数组** 等。几乎所有的对象实例和数组都在这里分配内存。
> - **异常机制**：堆是垃圾回收（GC）管理的主要区域。当堆内存不足且无法继续扩展时，JVM 会抛出 `OutOfMemoryError: Java heap space` 异常。

---

### 2. 堆的内存组成划分
Java 堆在结构上主要划分为：**年轻代（Young Generation）** 和 **老年代（Old Generation）**（默认空间大小比例为 `1 : 2`）：

#### (1) 年轻代（Young Generation）
* **细分区域**：年轻代被划分为三部分 —— **Eden 区** 和两个大小严格相同的 **Survivor 区（From Survivor / S0 和 To Survivor / S1）**。
* **空间比例**：默认比例通常为 `Eden : S0 : S1 = 8 : 1 : 1`（可通过 `-XX:SurvivorRatio` 调节）。
* **对象生命周期**：
  * 新创建的对象优先在 **Eden 区** 分配。
  * 当 Eden 区空间占满时，JVM 会触发 **Minor GC（也称 Young GC）** 进行垃圾回收。
  * 存活下来的对象会被复制移动到 Survivor 区，并在两个 Survivor 区之间来回复制，对象的“年龄计数器”每次 GC 增加 1。

#### (2) 老年代（Old Generation）
* **存储对象**：主要保存 **生命周期较长** 的对象（长期存活的对象），以及特别大的大对象（避免在年轻代频繁复制拷贝）。
* **晋升机制**：当 Survivor 区中的对象年龄达到阈值（默认 **15 岁**，可通过参数 `-XX:MaxTenuringThreshold` 设置）时，晋升进入老年代。
* **回收机制**：老年代空间不足时会触发 **Major GC / Full GC**，Full GC 耗时较长、开销较大。

---

### 3. JDK 1.7 与 JDK 1.8 的核心区别（永久代 vs 元空间）

| 比较维度 | JDK 1.7（及之前） | JDK 1.8（及之后） |
| :--- | :--- | :--- |
| **方法区实现** | 使用 **永久代（PermGen）** 实现 | 移除了永久代，改用 **元空间（Metaspace）** |
| **内存位置** | 属于 **JVM 进程管理的内存**（受限于 JVM 配置大小） | 移至 **本地直接内存（Native Memory）** |
| **存储内容** | 存放类的元数据、静态变量、字符串常量池、编译后的代码等 | **元空间** 仅存放类的元数据；<br>而 **字符串常量池、静态变量** 移至 **堆** 中存储 |
| **OOM 风险** | 容易因类加载过多导致 `OOM: PermGen space`（默认上限较小） | 默认只受本机物理可用内存限制，**大幅降低了内存溢出（OOM）风险** |

## 虚拟机栈
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201613290.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201809139.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201834182.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822201856399.png)

### 1. 什么是虚拟机栈（JVM Stacks）？
> 💡 **核心概念**：
> - **线程私有**：每个线程在运行时所需要的内存空间，称为虚拟机栈。生命周期与线程相同。
> - **栈帧（Frame）**：每个栈由多个栈帧组成。**每个方法被调用时都会创建一个栈帧**，对应着该方法调用所占用的内存（存储参数、局部变量、返回地址等）。
> - **先进后出（LIFO）**：方法调用即“压栈（Push）”，方法执行完毕即“出栈（Pop）”。
> - **活动栈帧**：每个线程在任意时刻**只能有一个活动栈帧**，对应着当前正在执行的那个方法（即位于栈顶的栈帧）。

#### 栈帧的“先进后出（LIFO）”与执行顺序关系

* **核心规律**：
  * **最先调用**的方法最先入栈（在栈底），但**最后执行完（最后出栈）**。
  * **最后调用**的方法最后入栈（在栈顶），但**最先执行完（最先出栈）**。
* **为什么活动栈帧永远是栈顶？**
  * JVM 的程序计数器（PC 寄存器）和执行引擎在任意时刻**只针对栈顶栈帧**执行字节码指令。
  * 栈下方的其他栈帧处于**挂起/等待状态**，完整保存着自己的局部变量和现场数据，等待上层方法执行完毕并返回。

```java
public class StackDemo {
    public static void main(String[] args) {
        System.out.println("1. main 开始");
        methodA();
        System.out.println("5. main 结束");
    }

    public static void methodA() {
        System.out.println("2. methodA 开始");
        methodB();
        System.out.println("4. methodA 恢复执行并结束");
    }

    public static void methodB() {
        System.out.println("3. methodB 正在执行（当前处于栈顶活动栈帧）");
    }
}
```

**栈帧压栈与出栈时序图解**：
```
【1. 调用 main()】      【2. main 调 A()】       【3. A 调 B()】
                                               ┌─────────────┐ <─ 当前活动栈帧（正在执行 B）
                                               │  methodB 帧 │
                        ┌─────────────┐        ├─────────────┤
                        │  methodA 帧 │        │  methodA 帧 │ (挂起等待 B 返回)
┌─────────────┐         ├─────────────┤        ├─────────────┤
│   main 帧   │         │   main 帧   │        │   main 帧   │ (挂起等待 A 返回)
└─────────────┘         └─────────────┘        └─────────────┘
 main 最先入栈(栈底)        A 随后入栈             B 最后入栈(栈顶)
──────────────────────────────────────────────────────────────────
【4. B 执行完毕返回】    【5. A 执行完毕返回】    【6. main 执行完毕】
                        
┌─────────────┐ <─ 重新成为栈顶(恢复执行A)
│  methodA 帧 │         
├─────────────┤         ┌─────────────┐ <─ 重新成为栈顶(恢复执行main)
│   main 帧   │         │   main 帧   │        [ 栈完全清空，线程结束 ]
└─────────────┘         └─────────────┘        
 B 最先出栈(弹栈释放)       A 第二个出栈            main 最后出栈
```

---


![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822202005772.png)

### 2. 虚拟机栈常见面试题

#### Q1：垃圾回收（GC）是否涉及栈内存？
> **不涉及**。
> 垃圾回收主要针对 **堆内存**。栈内存中的栈帧在方法执行结束时会自动弹栈释放，不需要垃圾回收器来处理。

#### Q2：栈内存分配越大越好吗？
> **未必**。
> - JVM 默认每个线程的栈内存大小通常为 `1024K`（可通过 `-Xss` 参数调整）。
> - 栈内存设得过大会导致**可创建的并发线程数变少**。
> - 例如：物理总内存为 `512M`，每个栈分配 `1024K`，理论上最多支持约 512 个线程并发；若将栈内存改为 `2048K`，则能支持的并发线程数直接减半（约 256 个）。

---

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822202347572.png)

#### Q3：方法内的局部变量是否线程安全？（结合代码深度剖析）

> 💡 **核心判断原则**：
> - **如果局部变量没有逃离方法的作用范围**：它是 **线程安全** 的（每个线程有独立的栈和栈帧，互不影响）。
> - **如果局部变量引用了对象，并逃离了方法的作用范围**：它需要考虑 **线程安全** 问题。

#### 代码实战与栈帧分析：

```java
// 1. 线程安全（未逃逸）
public static void m1() {
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append(2);
    System.out.println(sb.toString());
}

// 2. 线程不安全（入参导致对象逃逸）
public static void m2(StringBuilder sb) {
    sb.append(3);
    sb.append(4);
    System.out.println(sb.toString());
}

// 3. 线程不安全（返回值导致对象逃逸）
public static StringBuilder m3() {
    StringBuilder sb = new StringBuilder();
    sb.append(5);
    sb.append(6);
    return sb;
}

// 4. 主函数演示
public static void main(String[] args) {
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append(2);
    new Thread(() -> {
        m2(sb); // 将主线程的 sb 传给子线程，导致多线程共享堆中同一个对象
    }).start();
}
```

#### 逐个方法深度解析：
1. **`m1()` —— 【线程安全】**：
   * `StringBuilder sb` 在方法内部创建，保存在当前线程专属的栈帧局部变量表中。
   * 多线程同时调用 `m1()` 时，每个线程都在各自的栈帧中操作各自的 `sb` 对象，彼此隔离，互不干扰。
2. **`m2(StringBuilder sb)` —— 【线程不安全】**：
   * `sb` 是通过参数传入的，引用由外部提供。
   * 多个线程可能同时把同一个 `sb` 对象的引用传入 `m2`，此时多个线程的栈帧指向堆中同一个对象实例，由于 `StringBuilder` 线程不安全，并发 `append()` 会发生冲突。
3. **`m3()` —— 【线程不安全】**：
   * 虽然 `sb` 在方法内部创建，但通过 `return sb;` 将对象暴露给了外部。
   * 对象“逃逸”出当前栈帧，后续可能被外部多个线程并发操作，因此存在线程安全隐患。

---

#### Q4：执行 `String A = new String("AAA");` 时，对象和引用分别存储在哪里？

> 💡 **核心结论**：**对象实体永远在堆中，引用变量 `A` 的存储位置取决于它的声明作用域**。
>
> 1. **`A` 为方法内的【局部变量】**（最常见）：
>    - 存储在 **虚拟机栈** 当前栈帧的 **局部变量表（Local Variable Table）** 中，存放的是指向堆中对象的内存地址指针。
> 2. **`A` 为类的【成员变量 / 实例变量】**：
>    - 存储在 **Java 堆** 中，跟随所属的对象实例一同分配在堆中（作为实例数据的一部分）。
> 3. **`A` 为类的【静态变量 (`static`)】**：
>    - 存储在 **Java 堆** 中，存放在该类对应的 `java.lang.Class` 镜像对象中（JDK 8+）。
>
> 📌 **完整内存分布图解（以局部变量为例）**：
> - `new String("AAA")` 实例对象：存储在 **Java 堆** 中；
> - `"AAA"` 字符串字面量：存储在 **堆中的字符串常量池（StringTable）** 中；
> - 引用变量 `A`：存放在 **虚拟机栈** 的局部变量表中，指向堆中的 `String` 实例。

### 4. 虚拟机栈的内存溢出问题（StackOverflowError）
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822203523770.png)

> 💡 **什么情况下会导致栈内存溢出？**
> 1. **栈帧过多导致栈内存溢出**：
>    * **典型场景**：递归调用没有正确的退出条件（死递归），或者调用层次过深。
>    * 示例代码：
>      ```java
>      public static void m4() {
>          m4(); // 无限递归压栈，导致栈深度耗尽
>      }
>      ```
>    * 抛出异常：`java.lang.StackOverflowError`。
> 2. **栈帧过大导致栈内存溢出**：
>    * 单个方法内部声明了极多的局部变量，导致单个栈帧体积超出栈限制。

---

### 5. 虚拟机栈核心总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822203643414.png)

> 📌 **虚拟机栈面试 6 问速记**：
> 1. **什么是虚拟机栈？**
>    每个线程运行时所需的内存，称为虚拟机栈。先进后出（LIFO），由多个栈帧组成（每次方法调用分配一个栈帧），活动栈帧永远是位于栈顶的那一个。
> 2. **垃圾回收是否涉及栈内存？**
>    不涉及。GC 只针对堆内存，栈帧在方法执行完毕后自动出栈释放。
> 3. **栈内存分配越大越好吗？**
>    未必。默认 1024K（`-Xss`），栈内存越大，可并发创建的线程总数越少。
> 4. **方法内的局部变量是否线程安全？**
>    - 未逃离方法作用范围：线程安全（栈帧线程私有）；
>    - 逃离方法作用范围（入参传入或 return 导出）：需考虑线程安全。
> 5. **对象与引用的存储位置关系？**
>    对象实体在堆（Heap），引用变量随作用域决定（局部变量在栈的局部变量表，成员/静态变量在堆中）。
> 6. **什么情况下会导致栈内存溢出？**
>    栈帧过多（死递归、调用链太长）或单个栈帧过大，抛出 `java.lang.StackOverflowError`。

---

## 方法区（Method Area）
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822203830650.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822204016572.png)

### 1. 什么是方法区？
> 💡 **核心概念**：
> - **线程共享**：各个线程共享的内存区域。
> - **生命周期**：虚拟机启动时创建，关闭虚拟机时释放。
> - **存储内容**：主要存储 **类的信息（Class 元数据）**、**类加载器（ClassLoader）**、**运行时常量池** 等。
> - **物理位置**：
>   * **JDK 1.7 及之前**：由 **永久代（PermGen）** 实现，属于 JVM 进程管理的内存（逻辑上属于堆）。
>   * **JDK 1.8 及之后**：移除了永久代，改用 **元空间（Metaspace）** 实现，存储在 **本地直接物理内存（Native Memory）** 中。
> - **异常情况**：如果方法区域中的内存无法满足分配请求，则会抛出 `java.lang.OutOfMemoryError: Metaspace`。

---

### 2. 方法区与堆（Heap）的核心区别（重点对比）

> 💡 **通俗比喻**：**方法区是“图纸档案室”，堆是“实体加工厂”**。
> - **方法区**：记录类的结构图纸（类名、修饰符、字段定义、方法字节码指令等）。
> - **堆**：存放根据图纸创建出来的具体对象实体（`new Object()`）和数组。

| 对比维度 | 方法区（元空间 Metaspace） | 堆（Heap） |
| :--- | :--- | :--- |
| **存储内容 (JDK 1.8)** | 类元信息、方法字节码、ClassLoader、运行时常量池 | 对象实例（`new` 出来的实体）、数组、字符串常量池（StringTable）、静态变量 |
| **物理内存位置** | **本地直接内存（Native Memory）**，使用系统物理内存 | **JVM 虚拟机内存**，受 `-Xms`、`-Xmx` 堆参数限制 |
| **垃圾回收 (GC)** | **极少发生 GC**（类卸载条件极其苛刻） | **GC 极其频繁活跃**（Minor GC / Major GC / Full GC） |
| **OOM 报错信息** | `java.lang.OutOfMemoryError: Metaspace`<br>（动态生成加载类过多） | `java.lang.OutOfMemoryError: Java heap space`<br>（对象实例过多耗尽堆空间） |
| **调优参数** | `-XX:MetaspaceSize`、`-XX:MaxMetaspaceSize` | `-Xms`、`-Xmx` |

---

### 3. 元空间内存溢出实战（Metaspace OOM 演示）

**本地内存指的是系统本地的物理内存**。以下代码通过 `ClassWriter` 动态生成大量类字节码并加载，演示元空间内存溢出：

```java
/**
 * 演示元空间内存溢出 java.lang.OutOfMemoryError: Metaspace
 * JVM 启动参数: -XX:MaxMetaspaceSize=8m
 */
public class MetaspaceDemo extends ClassLoader { // ClassLoader 可以用来加载类的二进制字节码
    public static void main(String[] args) {
        MetaspaceDemo test = new MetaspaceDemo();
        for (int i = 0; i < 10000; i++) {
            // ClassWriter 作用是生成类的二进制字节码
            ClassWriter cw = new ClassWriter(0);
            // 访问修饰符, public, 类名, 包名, 父类, 接口
            cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
            // 返回 byte[] 二进制字节码
            byte[] code = cw.toByteArray();
            // 执行类的加载，生成 Class 对象并存入元空间
            test.defineClass("Class" + i, code, 0, code.length);
        }
    }
}
```

> 💡 **实验结论**：
> - 设置 VM 参数 `-XX:MaxMetaspaceSize=8m`，加载 10,000 个动态类时，元空间因被大量 Class 元数据占满而报错：`java.lang.OutOfMemoryError: Metaspace`。
> - 如果去除该 VM 参数限制，由于本地物理内存足够大，则不会报错。

```bash
# 运行报错示例
-XX:MaxMetaspaceSize=8m -cp 'C:\JavaProjects\面试题目\jvm-demo\target\classes' com.heima.jvm.MetaspaceDemo
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
Error occurred during initialization of VM
MaxMetaspaceSize is too small.
```

---

### 4. 常量池 与 运行时常量池

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822205455307.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822205609119.png)

#### (1) 常量池（Constant Pool，静态文件内）
* **定义**：存在于编译后的 `*.class` 二进制文件中。
* **作用**：可以看作是**一张静态符号表**。JVM 字节码指令通过符号引用（如 `#1`、`#2`、`#3`）查表翻译，找到要执行的类名、方法名、参数类型、字面量等信息。
* **查看方式**：通过反编译命令 `javap -v Application.class` 查看常量池表结构。

#### (2) 运行时常量池（Runtime Constant Pool，内存中）
* **定义**：当 `*.class` 文件被类加载器加载到内存中后，其常量池信息会被放入**方法区（元空间）中的运行时常量池**。
* **核心动作（符号引用 ➡️ 直接地址）**：将 class 文件中的**符号引用/符号地址**转化为**真实的物理内存直接地址**。

---

### 5. 方法区与常量池总结

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822205720574.png)

> 📌 **高频面试 2 问速记**：
> 1. **能不能解释一下方法区？**
>    - 线程共享的内存区域，主要存储类的信息、ClassLoader、运行时常量池；
>    - JVM 启动时创建，关闭时释放；
>    - JDK 1.8 采用本地内存的元空间实现，空间不足抛出 `OutOfMemoryError: Metaspace`。
> 2. **介绍一下运行时常量池：**
>    - 常量池是 `*.class` 文件中的静态表，用于指令查表翻译；
>    - 类加载后放入方法区的运行时常量池，将符号引用转化为物理内存真实地址。
## 直接内存
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822210511657.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822210615938.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822210948979.png)
java无法直接读取系统内存，所以涉及两个缓冲区，效率较低
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260822211058578.png)
少了一层缓冲区的复制，效率会高

**总结**
你听过直接内存吗？
● 并不属于VM中的内存结构，不由JVM进行管理。是虚拟机的系统内存
● 常见于NIO操作时，用于数据缓冲区，分配回收成本较高，但读写性能高，不受JVM 内存回收管理
# 类加载器
## 类加载器与双亲委派机制
### 什么是类加载器
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823133724534.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823133813235.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134020205.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134106644.png)
> 一般我们自己写的代码都是由应用类加载器加载的
### 什么是双亲委派
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134249161.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134429976.png)
> 对于我们自己编写的Student类，是从AppClassLoader开始加载，委托上一级即ExtClassLoader,再区委托上一级即BootStrap ClassLoder，而/jre/lib目录下没有这个Student类，所以AppClassLoader自己加载Student类

> String类也是同理，从AppClassLoader向上，到达BootStrap ClassLoader, 存在于/jre/lib目录下，加载完成直接返回给子加载器

### 为什么要使用双亲委派机制
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134847895.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823134941156.png)

## 类装载的执行过程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823135809304.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823135929131.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823140107558.png)

### 1. 加载（Loading）
* **核心 3 步**：
  1. **找字节流**：通过类加载器（双亲委派）获取 `.class` 二进制字节流；
  2. **入元空间**：将静态结构解析并存入 **元空间（方法区）**，生成 C++ 层的 `InstanceKlass`（存储方法字节码、虚方法表、类结构等）；
  3. **建堆入口**：在 **Java 堆** 中创建对应的 `java.lang.Class` 镜像对象（作为 Java 访问入口、存放 `static` 静态变量）。

> 💡 **重点辨析：元空间 Klass vs 堆 Class 对象 vs 实例（张三/李四）**
> - **元空间 `InstanceKlass`（图纸真身）**：底层 C++ 结构体，存类的完整元数据与方法字节码指令。
> - **堆中 `Class<Person>`（镜像代理）**：供 Java 代码（反射/API）访问元数据的入口，`static` 静态变量存放在此。
> - **堆中 `张三 / 李四`（具体实例）**：存各自的属性数据（`name="张三"`）；**对象头中的类型指针直接指向元空间的 `InstanceKlass`**，调用方法时直接定位元空间执行指令。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823140411556.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823140526808.png)
> 类变量即static修饰的静态变量，b在准备阶段会赋默认值0,在初始化阶段才会赋值。而final修饰的c和d都是在准备阶段完成赋值，static和final修饰的引用类型，赋值也在初始化阶段完成

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823140853750.png)
> #1 #2 #3 #4即为符号引用，通过一层层的查找，找到#31 #32 对应的代码，就是把符号引用转换为直接引用的过程。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823141238772.png)
```
public class Application {

    public static void main(String[] args) throws InterruptedException {
        // System.out.println("hello world");

        // 1. 首次访问这个类的静态变量或静态方法时
        //System.out.println(Animal.num);
        // 2. 子类初始化，如果父类还没初始化，会引发父类先初始化
        // System.out.println(Cat.sex);
        // 3. 子类访问父类静态变量，只触发父类初始化
        // System.out.println(Cat.num);
    }

}

class Animal {
    static int num = 55;
    static {
        System.out.println("Animal 静态代码块...");
    }
}

class Cat extends Animal {
    static boolean sex = false;
    static {
        System.out.println("Cat 静态代码块...1");
    }

    static {
        System.out.println("Cat 静态代码块...2");
    }
}
```
直接访问父类静态变量时，静态代码块也会执行
```
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
Animal 静态代码块...
55
```
子类初始化，则会先初始化起父类
```
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
Animal 静态代码块...
Cat 静态代码块...1
Cat 静态代码块...2
false
```
通过子类访问父类的静态变量，只会初始化父类的静态代码块，不会初始化子类的
```
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
Animal 静态代码块...
55
```
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823142313107.png)
卸载即代码执行完毕之后，jvm销毁创建的class对象，相当于把类卸载了
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823142557050.png)
# 垃圾回收
## 什么时候可以被垃圾回收
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823151826147.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823151914793.png)
## 定位垃圾的两种方式
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152055251.png)
>demo最初只想堆中的String对象，此时计数为1，而将demo指向null的时候，计数为0、
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152336380.png)
>只要有引用，计数器就加1
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152434706.png)
>将ab都指向null，此时没有任何引用指向堆中对象，但计数仍然不为0，导致无法被垃圾回收，出现了循环引用，可能会导致内存泄漏
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152641428.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152748788.png)
>demo,a demo ,a都可以作为gc root
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823152927194.png)

## 垃圾回收算法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153122858.png)
### 标记清除算法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153240960.png)
### 标记整理算法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153321678.png)
### 复制算法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153419654.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153527301.png)

## Jvm分代回收
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153704499.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823153949239.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154110756.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154228061.png)
## Jvm垃圾回收器
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154316082.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154345431.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154438027.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154522860.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154714737.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154749779.png)
>初始标记阶段，只标记和gc roots关联的对象

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154842128.png)
>并发标记阶段，则会标记和A关联的对象

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823154959065.png)
>重新标记，在并发标记阶段，可能代码运行又产生了引用，把x标记为引用,也有可能把D断开引用，标记D未被引用

>重新标记完成后，才会真正的并发清理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155216636.png)

## G1垃圾回收器
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155304336.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155409516.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155535425.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155559162.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155721902.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155738855.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155831354.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823155947439.png)
>优先回收存活数量少的old区意味着可以释放更多的内存
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160059420.png)
>将eden分区和幸存分区的存活对象复制到新的幸存区，同时把老年代和幸存区的存活时间久的对象复制到一个新的老年代区
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160240298.png)
> 复制完成，内存得到释放，当进行了多轮的混合回收，又会进入到新生代回收，并发标记，混合回收的阶段
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160446468.png)
>如果一份分区内存不够存储巨型对象，那么会分配连续的内存空间存储巨型对象
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160623496.png)
## 强引用，软引用，弱引用，虚引用
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160736931.png)

### 1. 四种引用特性速览
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823160916985.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823161059864.png)

| 引用类型 | 回收时机 | 特点 / `get()` 表现 | 经典使用场景 | 引用队列的作用 |
| :--- | :--- | :--- | :--- | :--- |
| **强引用 (Strong)** | 只要 GC Roots 可达，**绝不回收** | 最普遍的引用方式（`Object o = new Object()`） | 绝大部分常规业务对象 | 无需队列 |
| **软引用 (Soft)** | **内存不足时** 触发回收 | 内存充足保留，不足才清理（`SoftReference`） | 内存敏感的高速缓存（如图片缓存） | 可选：清理 Map 中的空壳包装对象 |
| **弱引用 (Weak)** | **只要发生 GC** 就会被回收 | 无论内存是否充足，GC 即回收（`WeakReference`） | 防内存泄漏的缓存、`WeakHashMap`、`ThreadLocal` | 可选：清理 Map 中的空壳包装对象 |
| **虚引用 (Phantom)** | **随时可能被回收** | `get()` 永远返回 `null`，形同虚设（`PhantomReference`） | 监控对象回收、释放**堆外直接内存**（如 `DirectByteBuffer`） | **必须配合**：通知后台线程释放堆外直接内存 |

---

### 2. 引用队列（ReferenceQueue）与资源释放机制
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823161348824.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823161517993.png)

> 💡 **核心辨析：不同引用配合队列释放的“资源”有何不同？**

#### (1) 虚引用 ➡️ 释放【堆外系统直接内存】
* **痛点**：JVM 垃圾回收器只管 Java 堆内存，管不到操作系统本地内存。
* **流程**：
  1. 创建 `DirectByteBuffer` 时绑定虚引用 `Cleaner` 和 `ReferenceQueue`；
  2. 堆内 `DirectByteBuffer` 被 GC 回收时，JVM 将 `Cleaner` 虚引用推进引用队列；
  3. 后台守护线程（`ReferenceHandler`）从队列拿到通知，调用底层 `Unsafe.freeMemory()` 释放**堆外直接物理内存**。

#### (2) 软引用 / 弱引用 ➡️ 释放【堆内空壳包装对象，防止容器内存泄漏】
* **痛点**：目标真实大对象被 GC 回收后（`.get() == null`），外层的 `SoftReference` / `WeakReference` 包装壳及 Map 中的 Key-Value 键值对依然被强引用残留在缓存 Map 中。
* **流程**：
  1. 真实数据被回收后，JVM 将失效的引用包装壳推进 `ReferenceQueue`；
  2. 业务代码或 `WeakHashMap` 从队列轮询取出失效的包装引用，执行 `map.remove()` 从集合中彻底剔除，**释放 Map 占用的堆内存**。
# Jvm实践
## Jvm如何设置参数
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823164032373.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823164056325.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823164143422.png)
>windows目录下后缀是bat
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823164305670.png)
>java -jar是springboot启动方式，中间即jvm相关参数

JVM调优的参数可以在哪里设置参数值
- war包部署在tomcat中设置
修改TOMCAT_HOME/bin/catalina.sh文件
- jar包部署在启动参数设置
java -Xms512m -Xmx1024m -jar xxxx.jar
## Jvm调优常见参数有哪些？

对于JVM调优，主要就是调整年轻代、老年代、元空间的内存空间大小及使用的垃圾回收器类型。
https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html
- 设置堆空间大小
- 虚拟机栈的设置
- 年轻代中Eden区和两个Survivor区的大小比例
- 年轻代晋升老年代阈值
- 设置垃圾回收收集器

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165004335.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165058090.png)
>减少栈内存，确实可以让线程变多，但如果方法，局部变量多， 可能会导致栈内存溢出的情况

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165246143.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165324953.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165434161.png)
## JVM调优工具
命令工具
进程状态信息
- jps
查看java进程内线程的堆栈信息
- jstack
查看堆转信息
- jmap
堆转储快照分析工具
- jhat
- jstatJVM统计监测工具
可视化工具
- jconsole用于对jvm的内存，线程，类的监控
- VisualVM 能够监控线程，内存情况
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823165745392.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170026267.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170122255.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170227300.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170227300.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170414135.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170433576.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170452552.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170532839.png)

## Java内存泄漏排查思路
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170612606.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170701825.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823170827634.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171042385.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171109339.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171132212.png)
java内存泄露的排查思路？
内存泄漏通常是指堆内存，通常是指一些大对象不被回收的情况
1、通过jmap或设置jvm参数获取堆内存快照dump
2、通过工具，VisualVM去分析dump文件，VisualVM可以加载离线的dump文件
3、通过查看堆信息的情况，可以大概定位内存溢出是哪行代码出了问题
4、找到对应的代码，通过阅读上下文的情况，进行修复即可
## Cpu飙高排查思路
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171419025.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171719407.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171816155.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260823171854914.png)
CPU飙高排查方案与思路？
1.使用top命令查看占用cpu的情况
2.通过top命令查看后，可以查看是哪一个进程占用cpu较高
3.使用ps命令查看进程中的线程信息
4.使用jstack命令查看进程中哪些线程出现了问题，最终定位问题