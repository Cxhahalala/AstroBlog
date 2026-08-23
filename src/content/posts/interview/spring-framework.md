---
title: "Java 框架篇面试题：Spring、Spring Boot 与 MyBatis 核心解析"
published: 2026-08-22
description: "全面解析 Spring 单例 Bean 线程安全、AOP 实践、声明式事务及失效场景、Bean 生命周期、三级缓存循环依赖、Spring Boot 自动配置与启动流程、MyBatis 延迟加载与缓存机制。"
tags: ["Java","Spring","SpringBoot","MyBatis","面试"]
category: "面试"
draft: false
---

---
## # **Spring框架中的单例Bean是线程安全的吗？**
## 一、这题在问什么
面试官其实是在考你 3 个点：
1. **Spring Bean 默认作用域是不是单例**
2. **单例对象在多线程环境下会不会有并发问题**
3. **什么情况下安全，什么情况下不安全，怎么解决**
重点不是只回答“安全”或“不安全”，而是要说清楚：
**单例 Bean 本身不天然线程安全，是否线程安全取决于这个 Bean 里有没有共享的可变状态。**
---
## 二、怎么理解
你可以这样记：
### 1）Spring 默认是单例
Spring 里 Bean 默认作用域是 `singleton`，也就是整个容器里通常只有一个对象实例。
### 2）单例 Bean 会被多个线程共享
在 Web 项目里，很多请求会同时进来，这些请求线程拿到的可能都是同一个 Bean 对象。
### 3）真正的问题在“成员变量”
如果这个 Bean：
- **没有可变成员变量**
- 或者是**无状态 Bean**
那一般就是安全的。
如果这个 Bean 里定义了：
- 会被修改的成员变量
- 并且多个线程都会访问它
那就可能出现线程安全问题。
---
## 三、标准回答思路
面试时推荐你按这个顺序答：
### 第一步：先给结论
**Spring 中的单例 Bean 不一定线程安全。**
### 第二步：解释原因
因为 Spring 默认单例 Bean 会被多个线程共享，如果 Bean 中存在共享的可变成员变量，就可能发生线程安全问题。
### 第三步：补充什么时候安全
如果 Bean 是无状态的，只包含方法局部变量，不保存请求过程中的数据，那么通常是线程安全的。
### 第四步：说解决办法
常见做法有：
- 尽量设计成**无状态 Bean**
- 把共享变量改成**局部变量**
- 必要时加**同步控制**（如 `synchronized`、锁）
- 或者改成**prototype** / request 等合适作用域<br>但一般核心思路还是：**尽量不要在单例 Bean 里存可变状态**
---
## 四、背诵版本（短版）
### 适合 30 秒到 1 分钟回答
**Spring 中的单例 Bean 不一定是线程安全的。因为 Spring 默认 Bean 是 singleton，多个线程会共享同一个 Bean 实例。如果这个 Bean 是无状态的，只使用方法内部局部变量，那通常就是线程安全的；但如果 Bean 中定义了可变的成员变量，并且在并发场景下被多个线程共同访问和修改，就会出现线程安全问题。解决方式一般是尽量把 Bean 设计成无状态，避免使用共享可变成员变量，必要时再通过加锁或调整作用域来处理。**
---
## 五、背诵版本（稍长版）
### 适合 1 到 2 分钟展开回答
**Spring 框架中的单例 Bean 默认不是绝对线程安全的，是否线程安全要看这个 Bean 的实现方式。因为 Spring 默认作用域是 singleton，也就是整个 IOC 容器中通常只有一个 Bean 实例。在 Web 应用中，多个请求线程会同时访问这个单例对象。如果这个 Bean 本身是无状态的，比如只做业务处理，不保存中间数据，只使用方法中的局部变量，那么一般不会有线程安全问题。**
**但是如果这个 Bean 中包含可变的成员变量，比如把请求数据、计算中间结果放到成员变量里，那么多个线程同时访问时就可能发生数据覆盖、脏读等并发问题。所以在实际开发中，我们通常会把 Service、Controller 这类单例 Bean 设计成无状态，避免定义可变成员变量。如果确实需要共享状态，可以考虑加锁、使用线程安全的数据结构，或者调整 Bean 的作用域，但最推荐的方式还是无状态设计。**
---
## 六、加分点
你再补一句，面试官会觉得你理解比较到位：
### 加分表达 1
**所以不能简单地说“单例 Bean 线程安全”或者“单例 Bean 线程不安全”，更准确地说，是“默认单例 Bean 会被多线程共享，是否安全取决于是否存在共享的可变状态”。**
### 加分表达 2
**像 Spring 里的 Service 一般推荐设计成无状态，这也是为什么大多数情况下虽然是单例，但实际使用中问题不大。**
---
## 七、超简洁记忆版
你可以直接背这一句：
**Spring 单例 Bean 不一定线程安全；无状态则通常安全，有共享可变成员变量则不安全。**
---
## 八、这题的面试答法模板
以后你看到类似题目，可以套这个模板：
**先说结论，再说原因，再说条件，再说解决方案。**
套进这题就是：
**单例 Bean 不一定线程安全。因为它会被多个线程共享。如果 Bean 是无状态的，通常安全；如果有共享可变成员变量，就可能不安全。解决方式是尽量无状态化，必要时加锁或调整作用域。**
---
可以，这张图上我按最开始那种风格，给你拆成三道题：
---
# 什么是AOP
**理解：**
AOP 就是**面向切面编程**，主要用来把和核心业务无关、但很多地方都会重复出现的公共功能抽出来统一处理，比如日志、事务、权限校验、异常处理等。这样可以在**不改业务代码**的前提下，对方法进行增强，减少重复代码，降低耦合。
**背诵版本：**
AOP 就是面向切面编程，它的核心思想是把日志、事务、权限校验这些和核心业务无关、但会被多个模块重复使用的公共功能抽取出来，统一进行处理。这样可以在不修改原有业务代码的情况下，对目标方法进行前置、后置或环绕增强，从而实现代码复用、降低耦合。
---
# 你们项目中有没有使用到AOP
**理解：**
这题不能只回答“用过”，要说出**具体场景**。项目里最常见的 AOP 使用场景有：
- 记录操作日志
- Spring 事务管理
- 缓存处理
- 权限校验
- 接口耗时统计
你可以重点说“操作日志”最容易讲。核心就是：
通过 **切点表达式** 找到需要拦截的方法，再通过 **环绕通知** 拿到方法信息、请求参数、执行结果、执行时间等，最后把这些日志保存到数据库或者日志系统里。
**背诵版本：**
我们项目中使用过 AOP，比较典型的场景是操作日志记录和 Spring 事务管理。比如在操作日志这块，我会通过切点表达式拦截指定的业务方法，然后在环绕通知中获取方法名、类名、请求参数、执行结果以及执行时间等信息，最后统一记录到数据库或者日志系统中。这样做的好处是不用在每个业务方法里重复写日志代码，维护起来也更方便。
---
# Spring中的事务是如何实现的
**理解：**
Spring 事务本质上就是基于 **AOP + 代理模式** 实现的。
当你在方法上加了 `@Transactional`，Spring 会为这个类生成一个代理对象。调用目标方法时，实际上先走代理逻辑：
- 方法执行前：开启事务
- 方法正常结束：提交事务
- 方法出现异常：根据规则回滚事务
所以本质不是业务代码自己控制事务，而是 Spring 在方法前后帮你做了增强。
**背诵版本：**
Spring 中的事务本质上是通过 AOP 和代理模式实现的。通常我们在方法上加上 `@Transactional` 注解后，Spring 会为目标对象生成代理对象。当调用这个方法时，实际上会先进入代理逻辑，在方法执行前开启事务，方法执行成功后提交事务，如果执行过程中出现异常，就根据事务的回滚规则进行回滚。所以 Spring 事务本质上就是对目标方法前后进行增强，把事务控制逻辑统一交给框架处理。
# Spring中事务失效的场景有哪些？
在项目中，我遇到过几种导致事务失效的场景：
1. 如果方法内部捕获并处理了异常，没有将异常抛出，会导致事务失效。因此，处理异常后应该确保异常能够被抛出。
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773924725900-2911de0a-f541-4933-abe1-6fb87d4426b6.png)
1. 如果方法抛出检查型异常（checked exception），并且没有在`@Transactional`注解上配置`rollbackFor`属性为`Exception`，那么异常发生时事务可能不会回滚。
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773924598488-79f1d406-78ad-40a9-a883-a9f490ee3967.png?x-oss-process=image%2Fformat%2Cwebp)
1. 如果事务注解的方法不是公开（public）修饰的，也可能导致事务失效。
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773924620750-03ace6d8-3991-45fb-b906-fddf60f16da4.png?x-oss-process=image%2Fformat%2Cwebp)
**理解：**
Spring 事务失效常见有三种情况：一是**异常被自己捕获了**，Spring 感知不到异常，就不会回滚；二是**抛出的是检查异常**，Spring 默认只对运行时异常回滚，所以要通过 `rollbackFor=Exception.class` 指定；三是**加了事务的方法不是 public**，因为 Spring 事务底层是 AOP 代理，非 public 方法可能不会被代理到，导致事务失效。
**背诵版本：**
Spring 事务失效常见场景主要有三种。第一，方法内部把异常自己捕获并处理了，没有继续抛出，这样 Spring 就感知不到异常，事务不会回滚。第二，抛出的是检查异常，Spring 默认只对运行时异常回滚，所以一般需要通过 `rollbackFor=Exception.class` 来指定回滚。第三，事务方法不是 public 的，由于 Spring 事务是基于 AOP 代理实现的，非 public 方法可能不会被代理，从而导致事务失效。
# Spring的bean的生命周期？
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773925149270-a3115b88-2733-40c1-ae18-b76460eb5102.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773925108799-d77b3ae5-65f6-4f64-996d-bdef43e22cf4.png)
Spring中bean的生命周期包括以下步骤：
1. 通过BeanDefinition获取bean的定义信息。
2. 调用构造函数实例化bean。
3. 进行bean的依赖注入，[例如通过setter方法或@Autowired注解](mailto:%E4%BE%8B%E5%A6%82%E9%80%9A%E8%BF%87setter%E6%96%B9%E6%B3%95%E6%88%96@Autowired%E6%B3%A8%E8%A7%A3)。
4. 处理实现了Aware接口的bean。
5. 执行BeanPostProcessor的前置处理器。
6. 调用初始化方法，如实现了InitializingBean接口或自定义的init-method。
7. 执行BeanPostProcessor的后置处理器，可能在这里产生代理对象。
8. 最后是销毁bean。
# 什么是Spring的循环
1. Spring中的循环引用？<br>候选人：<br>循环依赖发生在两个或两个以上的bean互相持有对方，形成闭环。Spring框架允许循环依赖存在，并通过三级缓存解决大部分循环依赖问题：
2. 一级缓存：单例池，缓存已完成初始化的bean对象。
3. 二级缓存：缓存尚未完成生命周期的早期bean对象。
4. 三级缓存：缓存ObjectFactory，用于创建bean对象。
# 解决循环引用的流程
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773925914995-80f6911a-8e4d-454c-82f4-b8e0ee4ad00a.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773925992155-29a3a54c-b37b-4c54-9192-92069d11e914.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926093246-e07334cd-fa49-4a6e-b40c-a127c63c45a0.png)
一级缓存无法解决循环依赖问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926151342-124740da-05ea-4a55-8ba6-d404faf4519e.png)
一级缓存和二级缓存一起工作
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926229286-532a449e-d24c-48af-bc40-aafdf3166008.png)
但一级缓存和二级缓存一起工作也无法完全解决循环依赖问题
假设A实际上是一个代理对象，根据Bean的生命周期，正常情况下是在初始化对象之后产生的，那么此时二级缓存中不是代理对象，B中注入的可能是原始对象A，就会导致有一个代理对象A和一个原始对象A
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926316372-0a09d1f8-0bc6-4621-ac6e-1f159fce11e5.png)
三级缓存解决循环依赖问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926429297-fbdee02d-9c15-4447-8b5e-40bda1fb11d7.png)
需要自己解决的循环依赖
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773926515184-dd2ea742-3a0c-41a4-a218-8d3581bd7f0f.png)
**理解：**
Spring 用三级缓存解决循环依赖，核心是为了在 Bean 还没完全初始化完成时，能够**提前暴露一个可用的对象引用**。
一级缓存放的是完成初始化的单例 Bean，二级缓存放的是提前暴露的 Bean，三级缓存放的是生成早期 Bean 的工厂 `ObjectFactory`。
如果只是普通对象，提前暴露原始对象即可；但如果这个 Bean 后续还需要被 AOP 代理，就不能直接把原始对象暴露出去。三级缓存的作用，就是在发生循环依赖时，通过工厂方法决定提前暴露原始对象还是代理对象，从而既解决循环依赖，又保证最终注入的对象一致。
**背诵版本：**
Spring 解决循环依赖主要依赖三级缓存机制。一级缓存存放的是完整初始化后的单例 Bean，二级缓存存放的是提前暴露的早期 Bean，三级缓存存放的是一个 `ObjectFactory` 工厂，用来生成早期 Bean 引用。当 A 依赖 B、B 又依赖 A 时，Spring 在创建 A 的过程中，会先实例化 A，并把它对应的工厂放入三级缓存；当创建 B 时如果需要注入 A，就可以通过三级缓存提前获取 A 的引用，从而打破循环依赖。如果 A 后续需要生成代理对象，Spring 也可以通过三级缓存提前暴露代理引用，而不是原始对象，避免出现对象不一致的问题。所以三级缓存不仅解决了循环依赖，还兼顾了 AOP 代理场景。
# 构造方法出现循环依赖的解决方法
**理解：**
构造方法循环依赖，Spring **默认解决不了**。因为构造器注入发生在 Bean 实例化阶段，A 创建时必须先拿到 B，B 创建时又必须先拿到 A，这时候两个对象都还没创建出来，Spring 没法像属性注入那样先实例化一个“半成品 Bean”再提前暴露，所以三级缓存也帮不上忙。常见做法是**改成 setter / **`**@Autowired**`\*\* 属性注入\*\*，或者对其中一个依赖加 `**@Lazy**`，让它延迟注入。
从 **Spring 启动和 Bean 创建** 的角度看，算解决了，因为程序能正常创建起来。
从 **代码设计** 的角度看，不算真正解决，因为 A 和 B 还是互相依赖，耦合关系还在。
**背诵版本：**
构造方法出现循环依赖时，Spring 一般无法直接解决。因为构造器注入是在 Bean 实例化最开始执行的，创建 A 时就必须先得到 B，创建 B 时又必须先得到 A，而这时两个 Bean 都还没有完成实例化，所以 Spring 无法通过提前暴露对象来处理。三级缓存解决的是属性注入场景下的循环依赖，不能解决构造器循环依赖。常见的解决方式是改用 setter 或 `@Autowired` 属性注入，或者在其中一个依赖上加 `@Lazy`，通过延迟加载来打破循环依赖。
# SpringMvc的执行流程
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773928799238-a793b7ca-8f8d-4f1e-b86d-873b18c2a7be.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773928857562-17d273b7-5f18-41a0-a98a-99490f32634c.png)
背诵
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773928963208-b594da88-3f1e-43e0-8311-4228ee2b8c0b.png)
#
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773929003229-e0128f19-3ece-489e-bcbb-c14a11871941.png)
# 介绍SpringBoot<br>这题很典型，面试官其实在考你三层：
1. **你知不知道 SpringBoot 是干嘛的**
2. **你能不能说出它相比 Spring 的核心优势**
3. **你是不是只会背概念，还是理解了自动配置这些关键点**
我给你整理成 **理解版 + 背诵版**。
---
## 一、理解版
## 1）这题在问什么
“介绍一下 SpringBoot” 本质上不是让你背定义，而是让你回答：
- SpringBoot 是什么
- 它解决了什么问题
- 它为什么流行
- 它的核心机制是什么
---
## 2）怎么理解 SpringBoot
你可以把它理解成：
**SpringBoot 是基于 Spring 的快速开发框架，用来简化 Spring 应用的配置、开发和部署。**
以前用 Spring 开发项目，麻烦点主要有这些：
- 要写很多 XML 配置
- 各种 Bean、数据源、事务都要手动配
- 依赖版本容易冲突
- 部署还经常要单独配 Tomcat
SpringBoot 就是来解决这些麻烦的。
---
## 3）SpringBoot 的核心特点
### 第一，约定大于配置
意思就是：
**很多东西 SpringBoot 先帮你配好了，你只要按它推荐的方式写就行。**
比如：
- 默认内嵌 Tomcat
- 默认日志框架
- 默认项目结构和配置方式
所以开发者不用从零开始折腾一堆配置。
---
### 第二，自动配置
这是 SpringBoot 最核心的点。
意思是：
**SpringBoot 会根据你引入的依赖，自动帮你创建和配置相关 Bean。**
比如：
- 你引入了 `spring-boot-starter-web`，它就帮你配 MVC、Tomcat 等 Web 环境
- 你引入了数据源相关依赖，它就尝试自动配置数据源
- 你引入了 Spring Security，它就自动装配安全相关配置
本质上就是：
**“你带什么材料来，我就给你配什么环境。”**
---
### 第三，起步依赖 starter
starter 可以理解为：
**把一组常用依赖打包好，让你不用一个个手动找。**
比如：
- `spring-boot-starter-web`
- `spring-boot-starter-test`
- `spring-boot-starter-data-jpa`
这样做的好处是：
- 依赖更统一
- 版本兼容性更好
- 配置更省事
---
### 第四，独立运行、部署方便
SpringBoot 常见方式是：
- 直接打成 jar 包
- 内嵌 Tomcat
- `java -jar` 就能跑
所以部署比传统 Spring 项目更方便。
---
## 4）面试里最好顺手提一句底层
如果面试官继续追问，你就说：
**SpringBoot 的核心在于自动配置，而自动配置的关键是 **`@EnableAutoConfiguration`**。它会加载很多自动配置类，再结合条件注解，比如 **`@ConditionalOnClass`**、**`@ConditionalOnMissingBean`**，来决定哪些配置生效。**
这句话一出来，面试官会觉得你不只是会背。
---
## 二、背诵版（1分钟）
**SpringBoot 是基于 Spring 的快速开发框架，主要作用是简化 Spring 应用的开发、配置和部署。**
在传统 Spring 项目里，我们通常需要写很多 XML 或配置类，还要自己处理依赖管理、Tomcat 部署这些问题，开发成本比较高。SpringBoot 就是为了解决这些问题而出现的。
**它最核心的特点有三个。**
第一是 **约定大于配置**，很多常用配置它都提供了默认值，开发者不需要手动配置大量细节。
第二是 **自动配置**，SpringBoot 会根据项目中引入的依赖，自动装配相关的 Bean 和运行环境。比如引入 Web 依赖后，它会自动配置 SpringMVC 和内嵌 Tomcat。
第三是 **starter 起步依赖**，它把常用依赖整合好了，避免我们一个个手动引入，也减少了版本冲突问题。
另外，SpringBoot 还支持**内嵌服务器、独立运行、打 jar 包直接部署**，所以开发效率和部署效率都更高。
**总结来说，SpringBoot 的核心价值就是让开发者把更多精力放在业务逻辑上，而不是繁琐的配置上。**
---
## 三、短背版（适合快问快答）
**SpringBoot 可以理解为 Spring 的快速开发脚手架，它通过约定大于配置、自动配置和 starter 依赖，简化了 Spring 项目的开发和部署。传统 Spring 需要写很多配置，还要单独部署 Tomcat，而 SpringBoot 内嵌服务器、开箱即用，能够让我们更专注于业务开发。**
---
## 四、面试加分版一句话
你可以最后补一句：
**SpringBoot 并不是替代 Spring，而是站在 Spring 的基础上，进一步提升开发效率。**
这句话很加分。
---
## 五、这个题你要注意别答成这样
面试里尽量别只说：
- “SpringBoot 很方便”
- “SpringBoot 就是简化开发”
- “SpringBoot 是 Spring 的升级版”
因为这些都太空了。
一定要落到这几个关键词上：
- **简化配置**
- **自动配置**
- **starter**
- **内嵌 Tomcat**
- **快速开发和部署**
---
# SpringBoot自动配置原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773929120968-64bec268-6efb-49ec-bf43-4001e27ca43e.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773929355168-b855a234-c728-4b9c-abd0-0364342c3d3c.png)
Spring Boot的自动配置原理基于`@SpringBootApplication`注解，它封装了`@SpringBootConfiguration`、`@EnableAutoConfiguration`和`@ComponentScan`。`@EnableAutoConfiguration`是核心，它通过`@Import`导入配置选择器，读取`META-INF/spring.factories`文件中的类名，根据条件注解决定是否将配置类中的Bean导入到Spring容器中。
# SpringBoot启动原理
### 理解版
Spring Boot 启动过程，本质上就是：
**从执行 main 方法开始，创建 Spring 容器，完成自动配置，注册 Bean，最后启动内嵌 Web 服务器。**
如果要按流程理解，可以分成下面几个步骤。
**第一步，启动入口。**
Spring Boot 项目通常从 `main` 方法开始，核心代码就是 `SpringApplication.run(XXX.class, args)`。
这行代码的作用可以理解为：**告诉 Spring Boot，开始启动整个应用。**
**第二步，创建 SpringApplication 对象。**
在执行 `run` 之前，Spring Boot 会先创建一个 `SpringApplication` 实例。
在这个过程中，它会做一些初始化工作，比如：
- 判断当前应用类型，是普通项目、Servlet Web 项目，还是响应式 Web 项目
- 加载应用启动监听器
- 加载初始化器
- 推断主启动类
这一步相当于先把“启动器”本身准备好。
**第三步，准备运行环境 Environment。**
Spring Boot 会创建并准备 `Environment` 对象，用来读取和管理配置。
这里会加载很多配置来源，比如：
- `application.yml`
- `application.properties`
- 命令行参数
- 系统环境变量
- JVM 参数
也就是说，这一步主要是在做：**把项目运行需要的配置先收集起来。**
**第四步，创建容器。**
Spring Boot 会根据应用类型创建对应的 Spring 容器。
如果是 Web 项目，通常创建的是 `AnnotationConfigServletWebServerApplicationContext`。
你可以把它理解成：**真正用来管理 Bean 的大容器开始建立了。**
**第五步，执行容器刷新 refresh。**
这是整个启动过程中最核心的一步。
在 `refresh()` 过程中，Spring 会完成很多关键动作，比如：
- 解析配置类
- 扫描组件
- 注册 BeanDefinition
- 实例化单例 Bean
- 处理依赖注入
- 执行各种后置处理器
- 完成自动配置
也就是说，**我们写的 Controller、Service、Mapper，以及各种自动配置类，基本都是在这个阶段被放进容器里的。**
**第六步，启动内嵌 Web 服务器。**
如果是 Web 项目，在容器刷新过程中还会启动内嵌服务器，比如 Tomcat。
这样项目启动完成后，就可以直接对外提供 HTTP 服务了。
**第七步，执行启动后的扩展逻辑。**
项目启动完成后，Spring Boot 还会执行一些回调，比如：
- `CommandLineRunner`
- `ApplicationRunner`
这些一般用于在项目启动后执行初始化逻辑。
所以整体可以总结成一句话：
**Spring Boot 启动过程就是：创建 SpringApplication → 准备环境 → 创建容器 → 刷新容器 → 完成自动配置和 Bean 注册 → 启动内嵌服务器。**
---
### 背诵版
Spring Boot 的启动过程可以从 `SpringApplication.run()` 这行代码开始理解。
它的整体流程是：先创建 `SpringApplication` 对象，然后准备运行环境 `Environment`，接着创建 Spring 容器，再通过 `refresh()` 方法完成容器刷新，最后启动内嵌的 Tomcat 等 Web 服务器。
其中最核心的步骤是 **容器刷新 refresh**。
在这个阶段，Spring 会完成配置类解析、组件扫描、BeanDefinition 注册、Bean 实例化、依赖注入以及自动配置等工作。我们平时写的 Controller、Service、以及各种自动配置类，基本都是在这个阶段加载到 Spring 容器中的。
如果项目是 Web 应用，那么在容器刷新过程中还会启动内嵌的 Tomcat，项目就可以直接对外提供服务了。
所以我认为，Spring Boot 启动过程的核心就是：**初始化环境、创建并刷新 Spring 容器、完成自动配置并启动 Web 服务。**
---
### 一分钟背诵版
Spring Boot 的启动入口通常是 `SpringApplication.run()`。
它启动时会先创建 `SpringApplication` 对象，接着准备运行环境，包括读取配置文件、系统参数等；然后根据应用类型创建 Spring 容器。
之后会执行容器的 `refresh()`，这是最核心的步骤，在这里会完成配置类解析、组件扫描、Bean 注册、依赖注入和自动配置。
如果是 Web 项目，在这个过程中还会启动内嵌的 Tomcat。
所以可以总结为：**Spring Boot 启动过程本质上就是准备环境、创建并刷新容器、完成自动配置并启动服务器。**
---
### 面试加分点
你可以补一句：
`@SpringBootApplication` 是一个组合注解，里面包含了 `@SpringBootConfiguration`、`@EnableAutoConfiguration` 和 `@ComponentScan`。
其中真正和自动配置最相关的是 `@EnableAutoConfiguration`。
再补一句会更加分：
Spring Boot 自动配置的底层核心是 **SpringFactories 机制**（老版本常见说法）或者 **自动配置类加载机制**，会把很多自动配置类导入进来，再结合条件注解，比如 `@ConditionalOnClass`、`@ConditionalOnMissingBean`，按需生效。
---
### 简化记忆版
你可以把启动流程记成五个词：
**入口启动 → 准备环境 → 创建容器 → 刷新容器 → 启动 Tomcat**
---
### 面试回答提醒
这题你不要一上来就只说源码类名，不然很容易显得像在背八股。
更好的回答方式是：
**先讲整体流程，再讲 refresh 是核心，最后再补充自动配置和内嵌 Tomcat。**
这样会显得你是真的理解了。
## 适合直接背的最终版
Spring Boot 的启动入口一般是 `SpringApplication.run()`。
它的启动过程大致可以分为几个步骤：首先创建 `SpringApplication` 对象，完成一些基础初始化；然后准备运行环境 `Environment`，读取配置文件、命令行参数和系统环境变量；接着根据应用类型创建对应的 Spring 容器。
随后会执行容器的 `refresh()` 方法，这一步是整个启动过程的核心。在这个阶段，Spring 会完成配置类解析、组件扫描、BeanDefinition 注册、Bean 实例化、依赖注入以及自动配置等工作。
如果当前是 Web 项目，那么在容器刷新过程中还会启动内嵌的 Tomcat 服务器，最终让应用对外提供服务。
所以我对 Spring Boot 启动过程的理解是：**从 **`run()`\*\* 开始，依次完成环境准备、容器创建、容器刷新、自动配置以及 Web 服务器启动。\*\*
# Spring常见注解Spring的常见注解包括：
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773933184323-6c924599-34cd-4ea5-83a3-11b98b2e58aa.png)
1. 声明Bean的注解：@Component、@Service、@Repository、@Controller。
2. 依赖注入相关注解：@Autowired、@Qualifier、@Resource。
3. 设置作用域的注解：@Scope。
4. 配置相关注解：@Configuration、@ComponentScan、@Bean。
5. AOP相关注解：@Aspect、@Before、@After、@Around、@Pointcut。
# SpringMvc常见注解
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773933254204-336891c1-9971-438b-a5d3-1ecb0cb90eba.png)
SpringMVC的常见注解有：
- @RequestMapping：映射请求路径。
- @RequestBody：接收HTTP请求的JSON数据。
- @RequestParam：指定请求参数名称。
- @PathVariable：从请求路径中获取参数。
- @ResponseBody：将Controller方法返回的对象转化为JSON。
- @RequestHeader：获取请求头数据。
- @PostMapping、@GetMapping等。
# SpringBoot常见注解
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773933288251-4fc0d3cf-dd66-47ac-8254-43f9c4cbfc57.png)
Spring Boot的常见注解包括：
- @SpringBootApplication：[由@SpringBootConfiguration](mailto:%E7%94%B1@SpringBootConfiguration)、@EnableAutoConfiguration和@ComponentScan组成。
- [其他注解如@RestController](mailto:%E5%85%B6%E4%BB%96%E6%B3%A8%E8%A7%A3%E5%A6%82@RestController)、@GetMapping、@PostMapping等，用于简化Spring MVC的配置。
-
# MyBatis执行流程
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773933638576-7481dd4e-7dd0-45e1-b07d-743c26625195.png)
MyBatis的执行流程如下：
1. 读取MyBatis配置文件mybatis-config.xml。
2. 构造会话工厂SqlSessionFactory。
3. 会话工厂创建SqlSession对象。
4. 操作数据库的接口，Executor执行器。
5. Executor执行方法中的MappedStatement参数。
6. 输入参数映射。
7. 输出结果映射。
# MyBatis是否支持延迟加载
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773933825492-65e8a2a2-ec20-45a3-b26a-ec6ece76af77.png)
### 简单理解
MyBatis 的延迟加载，就是对关联查询的数据 **按需加载**。
在执行主查询时，先只查主对象数据，关联对象的数据不立即查询，等真正访问关联属性时再发送 SQL。
### 举个例子
比如查询一个用户 `User`，这个用户下面有订单 `orders`。
正常立即加载：
- 查用户的时候，订单也一起查出来
延迟加载：
- 先只查用户
- 当代码真正调用 `user.getOrders()` 时，才去查订单
MyBatis支持延迟加载，即在需要用到数据时才加载。可以通过配置文件中的`lazyLoadingEnabled`配置启用或禁用延迟加载。
# 延迟加载的底层原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773934097234-2a68e4f8-cce8-4a51-acc4-a34eafee0ba8.png)
延迟加载的底层原理主要使用CGLIB动态代理实现：
1. 使用CGLIB创建目标对象的代理对象。
2. 调用目标方法时，如果发现是null值，则执行SQL查询。
3. 获取数据后，设置属性值并继续查询目标方法。
比如查用户信息时，如果用户下面还有订单列表，那在开启延迟加载的情况下，第一次查询可能只会把用户基本信息查出来，不会立刻查订单。
等代码真正访问 `orders` 这个属性时，MyBatis 才会再发一条 SQL 去查询订单数据。
它底层主要是通过代理对象实现的。
也就是说，主查询结束后，MyBatis 返回的不是一个完全初始化好的普通对象，而是一个带有延迟加载能力的代理对象。
当访问被延迟加载的属性时，代理会拦截这个操作，然后执行对应的 SQL，查到结果后再填充进去。
所以它的核心思想就是：**按需加载，避免不必要的关联查询。**
---
# MyBatis一级缓存和二级缓存
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773934595499-782598c2-f29f-40f8-93fa-1178917b0316.png)
## 一级缓存
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773934686896-5d5f5f19-692c-4686-95ca-e85ea08f247f.png)
userMapper1和userMapper2是使用的一个SqlSession
## 二级缓存
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773934900291-7d5cb24a-0ec2-479a-9721-27bc1373a967.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1773935005570-ceea66fc-21fe-431b-a057-18b59227ad13.png)
## 答案版本
MyBatis的一级缓存是基于`Perpetual\`\`Cache`的HashMap本地缓存，作用域为Session，默认开启。二级缓存需要单独开启，作用域为Namespace或mapper，默认也是采用`PerpetualCache`，HashMap存储。
# Mybatis的二级缓存什么时候会清理缓存中的数据？
当作用域（一级缓存Session/二级缓存Namespaces）进行了新增、修改、删除操作后，默认该作用域下所有select中的缓存将被清空。