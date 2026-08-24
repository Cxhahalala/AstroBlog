---
title: "Java 常见业务与高频技术场景面试题汇总"
published: 2026-08-24
description: "全面整理 Java 开发高频技术场景与实战面试题：单点登录（SSO/JWT）、RBAC 权限认证、数据与文件上传安全、ELK 日志采集与排查、Linux 运维命令、Arthas 生产问题排查与系统性能瓶颈调优。"
tags: ["Java", "技术场景", "面试"]
category: "面试"
draft: false
---

```mermaid
flowchart LR
    Root["常见技术场景"]

    Root --> N1["单点登录这块怎么实现的"]
    Root --> N2["权限认证是如何实现的"]
    Root --> N3["上传数据的安全性你们怎么控制？"]
    Root --> N4["你负责项目的时候遇到了哪些比较棘手的问题"]
    Root --> N5["你们是怎么做压测(性能测试)的"]
    Root --> N6["你们项目中日志怎么采集的"]
    Root --> N7["查看日志的命令"]
    Root --> N8["怎么快速定位系统的瓶颈"]
    Root --> N9["生产问题怎么排查"]

    style Root fill:#aedee6,stroke:#72b7c4,color:#222,stroke-width:2px
    
    classDef item fill:#2d7e97,stroke:#1e5e72,color:#fff;
    class N1,N2,N3,N4,N5,N6,N7,N8,N9 item;

```

# 单点登录
**单点登录这块怎么实现的**
单点登录的英文名叫做：Single Sign On（简称SSO），只需要登录一次，就可以访问所有信任的应用系统
**单体 vs 微服务/分布式**
- **单体**：单个tomcat服务session可以共享
- **微服务、分布式**：多个tomcat服务session不共享

**单点登录解决方案**
- JWT(常见)
- Oauth2
- CAS

```mermaid
flowchart TB
    subgraph Monolith ["单体架构（单个tomcat服务session可以共享）"]
        direction TB
        subgraph MonoBox ["单一应用服务容器"]
            direction LR
            M_Login["登录"]
            M_Product["商品"]
            M_Order["订单"]
            M_Pay["支付"]
        end
    end

    subgraph Microservices ["微服务、分布式（多个tomcat服务session不共享）"]
        direction TB
        subgraph MS1 ["登录服务"]
            MS_Login["登录"]
        end
        subgraph MS2 ["商品服务"]
            MS_Product["商品"]
        end
        subgraph MS3 ["订单服务"]
            MS_Order["订单"]
        end
        subgraph MS4 ["支付服务"]
            MS_Pay["支付"]
        end
    end

    classDef redBlock fill:#b53232,stroke:#851e1e,color:#fff;
    class M_Login,M_Product,M_Order,M_Pay,MS_Login,MS_Product,MS_Order,MS_Pay redBlock;

    style MonoBox fill:#fff,stroke:#b53232,stroke-width:2px
    style MS1 fill:#fff,stroke:#b53232,stroke-width:2px
    style MS2 fill:#fff,stroke:#b53232,stroke-width:2px
    style MS3 fill:#fff,stroke:#b53232,stroke-width:2px
    style MS4 fill:#fff,stroke:#b53232,stroke-width:2px

```

**JWT解决单点登录**

```mermaid
flowchart LR
    Browser["🌐 浏览器 / 客户端"] --> Gateway["网关<br/><br/>Gateway"]

    Gateway --> Login["登录"]
    Gateway --> Order["订单"]
    Gateway --> Product["商品"]
    Gateway --> Pay["支付"]

    %% 样式美化（还原原图橙/红色系）
    style Browser fill:#0284c7,stroke:#0369a1,color:#fff
    style Gateway fill:#f97316,stroke:#ea580c,color:#fff,min-height:120px

    classDef redService fill:#b91c1c,stroke:#991b1b,color:#fff;
    class Login,Order,Product,Pay redService;

```

流程图

```mermaid
sequenceDiagram
    autonumber
    participant Browser as 浏览器
    participant Gateway as 网关
    participant OrderService as 订单服务
    participant AuthService as 登录服务

    %% 未登录访问订单
    Browser->>+Gateway: 下单操作
    Gateway->>Gateway: 校验token是否有效
    Gateway-->>-Browser: 返回401 (授权失败)

    %% 登录认证获取 Token
    Browser->>+Gateway: 认证登录 (用户名密码)
    Gateway->>+AuthService: 路由到登录服务
    AuthService->>AuthService: 认证校验，返回token (JWT)
    AuthService-->>-Browser: 登录成功，返回token
    Browser->>Browser: 把token写入cookie
    deactivate Gateway

    %% 携带 Token 再次下单
    Browser->>+Gateway: 下单操作(携带token)
    Gateway->>Gateway: 校验token是否有效
    Gateway->>+OrderService: 路由到订单服务
    OrderService->>OrderService: 处理订单
    deactivate OrderService
    deactivate Gateway

```

**单点登录这块怎么实现的**
1. **先解释什么是单点登录**
	单点登录的英文名叫做：Single Sign On（简称**SSO**）
2. **介绍自己项目中涉及到的单点登录**（即使没涉及过，也可以说实现的思路）
3. **介绍单点登录的解决方案，以JWT为例**
	- I. 用户访问其他系统，会在网关判断token是否有效
	- II. 如果token无效则会返回401（认证失败）前端跳转到登录页面
	- III. 用户发送登录请求，返回浏览器一个token，浏览器把token保存到cookie
	- IV. 再去访问其他服务的时候，都需要携带token，由网关统一验证后路由到目标服务

# 权限认证
**权限认证是如何实现的**
后台的管理系统，更注重权限控制，最常见的就是RBAC模型来指导实现权限
RBAC(Role-Based Access Control)基于角色的访问控制
- 3个基础部分组成：用户、角色、权限
- 具体实现
	- 5张表（用户表、角色表、权限表、用户角色中间表、角色权限中间表）
	- 7张表（用户表、角色表、权限表、菜单表、用户角色中间表、角色权限中间表、权限菜单中间表）

**RBAC权限模型**

```mermaid
classDiagram
    direction TB

    class t_user {
        id
        username
        password
    }

    class t_role {
        id
        name
        code
        description
    }

    class t_permission {
        id
        name
        code
        description
    }

    class user_role {
        id
        user_id
        role_id
    }

    class role_permission {
        id
        permission_id
        role_id
    }

    t_user "1" -- "n" user_role
    t_role "1" -- "n" user_role
    t_role "1" -- "n" role_permission
    t_permission "1" -- "n" role_permission

```

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/41d50e731b905b5e.png)

**权限认证是如何实现的**
- 后台管理系统的开发经验
- 介绍RBAC权限模型5张表的关系（用户、角色、权限）
- 权限框架：Spring security

# 上传数据的安全性
**上传数据的安全性你们怎么控制？**
使用**非对称加密（或对称加密）**，给前端一个公钥让他把数据加密后传到后台，后台负责解密后处理数据

```mermaid
flowchart LR
    Client["🌐 浏览器 / 前端 (使用公钥加密)"] --> Backend

    subgraph Backend ["后台服务 (负责解密后处理数据)"]
        direction TB
        subgraph Row1 [" "]
            direction LR
            S1["服务A"]
            S2["服务B"]
        end
        subgraph Row2 [" "]
            direction LR
            S3["服务C"]
            S4["服务D"]
        end
        Row1 ~~~ Row2
    end

    style Client fill:#0284c7,stroke:#0369a1,color:#fff
    style Backend fill:#ffffff,stroke:#333333,stroke-width:2px
    style Row1 fill:transparent,stroke:transparent
    style Row2 fill:transparent,stroke:transparent

    classDef serviceNode fill:#d1d5db,stroke:#6b7280,color:#111827;
    class S1,S2,S3,S4 serviceNode;

```

**对称加密**
文件加密和解密使用相同的密钥，即加密密钥也可以用作解密密钥

```mermaid
graph LR
    %% 定义节点
    A["原始数据 (123456)"]:::red
    B(("加密")):::orange
    C["原始数据 (123456)"]:::red
    D(("解密")):::orange
    Key(("🔑 秘钥")):::key

    %% 定义连接线
    A -->|"传输明文"| B
    B -.->|"网络传输\n密文: b0baee9d279d34fa1dfd71aadb908c3f"| D
    D -->|"还原明文"| C
    
    %% 秘钥连接
    Key -->|"加密密钥"| B
    Key -->|"解密密钥"| D

    %% 样式定义
    classDef red fill:#a83232,stroke:#a83232,color:#ffffff,stroke-width:2px;
    classDef orange fill:#e67e22,stroke:#e67e22,color:#ffffff,stroke-width:2px;
    classDef key fill:#ffffff,stroke:#333333,stroke-width:2px,color:#333;

```

- **优点：** 加密速度快，效率高
- **缺点：** 相对不太安全（不要保存敏感信息）

**非对称加密**

```mermaid
flowchart LR
    PubKey["🔑 公钥"]
    PriKey["🔑 私钥"]

    RawData1["原始数据 (123456)"] --> Encrypt(("加密"))
    Encrypt -.->|"网络传输<br/>密文: b0baee9d279d34fa1dfd71aadb908c3f"| Decrypt(("解密"))
    Decrypt --> RawData2["原始数据 (123456)"]

    PubKey --> Encrypt
    PriKey --> Decrypt

    %% 样式美化
    classDef rawNode fill:#b91c1c,stroke:#991b1b,color:#fff;
    class RawData1,RawData2 rawNode;

    classDef opNode fill:#ea580c,stroke:#c2410c,color:#fff;
    class Encrypt,Decrypt opNode;

    style PubKey fill:#ffffff,stroke:#333333,stroke-width:2px,color:#000
    style PriKey fill:#ffffff,stroke:#333333,stroke-width:2px,color:#000

```

- **优点：** 与对称加密相比，安全性更高
- **缺点：** 加密和解密速度慢，建议少量数据加密

**上传数据的安全性你们怎么控制？**
使用**非对称加密（或对称加密）**，给前端一个公钥让他把数据加密后传到后台，后台负责解密后处理数据
- 文件很大建议使用对称加密，不过不能保存敏感信息
- 文件较小，要求安全性高，建议采用非对称加密

# 棘手问题
**你负责项目的时候遇到了哪些比较棘手的问题？怎么解决的**
**回答框架：**
1. 什么背景（技术问题）
2. 过程（解决问题的过程）
3. 最终落地方案

```mermaid
flowchart TD
    %% 核心中心
    Ready(("提前准备"))

    %% 01 设计模式
    subgraph G1 ["01 设计模式"]
        direction TB
        P1["工厂"]
        P2["策略"]
        P3["责任链"]
        P4["..."]
    end

    %% 02 线上BUG
    subgraph G2 ["02 线上BUG"]
        direction TB
        B1["CPU飙高"]
        B2["内存泄漏"]
        B3["线程死锁"]
        B4["..."]
    end

    %% 03 调优
    subgraph G3 ["03 调优"]
        direction TB
        O1["慢接口"]
        O2["慢SQL"]
        O3["缓存方案"]
        O4["..."]
    end

    %% 04 组件封装
    subgraph G4 ["04 组件封装"]
        direction TB
        C1["分布式锁"]
        C2["接口幂等"]
        C3["分布式事务"]
        C4["支付通用"]
    end

    %% 连线
    Ready --> G1
    Ready --> G2
    Ready --> G3
    Ready --> G4

    %% 样式
    style Ready fill:#22c55e,stroke:#16a34a,color:#fff,stroke-width:2px

    style G1 fill:#f0f9ff,stroke:#0284c7,stroke-width:2px
    classDef blueItem fill:#0284c7,stroke:#0369a1,color:#fff;
    class P1,P2,P3,P4 blueItem;

    style G2 fill:#fef2f2,stroke:#dc2626,stroke-width:2px
    classDef redItem fill:#dc2626,stroke:#b91c1c,color:#fff;
    class B1,B2,B3,B4 redItem;

    style G3 fill:#f7fee7,stroke:#65a30d,stroke-width:2px
    classDef greenItem fill:#65a30d,stroke:#4d7c0f,color:#fff;
    class O1,O2,O3,O4 greenItem;

    style G4 fill:#faf5ff,stroke:#9333ea,stroke-width:2px
    classDef purpleItem fill:#9333ea,stroke:#7e22ce,color:#fff;
    class C1,C2,C3,C4 purpleItem;

```

# 日志如何采集
**你们项目中日志怎么采集的**
**1，为什么要采集日志？**
日志是定位系统问题的重要手段，可以根据日志信息快速定位系统中的问题
**2，采集日志的方式有哪些？**
- **ELK：** 即Elasticsearch、Logstash和Kibana三个软件的首字母
- **常规采集：** 按天保存到一个日志文件

```plain text
designpattern-demo-8081-2023-05-14.log
designpattern-demo-8081-2023-05-15.log
designpattern-demo-8081-2023-05-16.log
designpattern-demo-8081-2023-05-17.log
designpattern-demo-8081-2023-05-18.log
designpattern-demo-8082-2023-05-18.log
designpattern-demo-8083-2023-05-18.log

```

**实际开发中使用较多的是ELK**
**ELK即Elasticsearch、Logstash和Kibana三个开源软件的缩写**
**LogBack即Spring Boot 官方默认的日志框架**。只要引入 Spring Boot 起步依赖，无需额外引包即可开箱即用。

```mermaid
flowchart LR
    %% 左侧应用系统
    subgraph Apps [" "]
        direction TB
        App1["应用系统1<br/><small>logback</small>"]
        App2["应用系统2<br/><small>logback</small>"]
        App3["应用系统3<br/><small>logback</small>"]
    end

    %% 中间与右侧核心组件
    subgraph LogstashBox ["动态收集筛选过滤数据（日志）"]
        Logstash["logstash"]
    end

    ES["Elasticsearch<br/><small>对数据存储，搜索、分析</small>"]
    Kibana["Kibana<br/><small>可视化展示，搜索、分析</small>"]

    %% 数据流向
    App1 --> Logstash
    App2 --> Logstash
    App3 --> Logstash
    Logstash --> ES
    ES --> Kibana

    %% 样式美化（还原原图绿、青、橙、蓝配色）
    style Apps fill:transparent,stroke:transparent
    style LogstashBox fill:#fff,stroke:#dc2626,stroke-width:2px

    classDef appNode fill:#84cc16,stroke:#65a30d,color:#fff;
    class App1,App2,App3 appNode;

    style Logstash fill:#06b6d4,stroke:#0891b2,color:#fff
    style ES fill:#f97316,stroke:#ea580c,color:#fff
    style Kibana fill:#2563eb,stroke:#1d4ed8,color:#fff

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <springProperty scope="context" name="serverPort" source="server.port"/>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>192.168.200.130:5044</destination>
        <!-- 日志输出编码 -->
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        <!--应用名称 -->
                        "app": "${springAppName}_${serverPort}",
                        <!--打印时间 -->
                        "timestamp": "%d{yyyy-MM-dd HH:mm:ss.SSS}",
                        <!--线程名称 -->
                        "thread": "%thread",
                        <!--日志级别 -->
                        "level": "%level",
                        <!--日志名称 -->
                        "logger_name": "%logger",
                        <!--日志信息 -->
                        "message": "%msg",
                        <!--日志堆栈 -->
                        "stack_trace": "%exception"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    <!--定义日志文件的存储地址,使用绝对路径-->
    <property name="LOG_HOME" value="/home/logs"/>
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <fileNamePattern>${LOG_HOME}/${springAppName}-${serverPort}-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
        <appender-ref ref="FILE" />
        <appender-ref ref="CONSOLE" />
    </root>

</configuration>

```

**你们项目中日志怎么采集的**
- 我们搭建了ELK日志采集系统
- 介绍ELK的三个组件：
	- **Elasticsearch**是全文搜索分析引擎，可以对数据存储、搜索、分析
	- **Logstash**是一个数据收集引擎，可以动态收集数据，可以对数据进行过滤、分析，将数据存储到指定的位置
	- **Kibana**是一个数据分析和可视化平台，配合Elasticsearch对数据进行搜索，分析，图表化展示

# 常见日志命令

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c325b7856374a1dd.png)

**查看日志的命令**
- **实时监控日志的变化**
	- 实时监控某一个日志文件的变化：`tail -f xx.log`
	- 实时监控日志最后100行日志：`tail -n 100 -f xx.log`
- **按照行号查询**
	- 查询日志尾部最后100行日志：`tail -n 100 xx.log`
	- 查询日志头部开始100行日志：`head -n 100 xx.log`
	- 查询某一个日志行号区间：`cat -n xx.log | tail -n +100 | head -n 100`（查询100行至200行的日志）
- **按照关键字找日志的信息**
	- 查询日志文件中包含debug的日志行号：`cat -n xx.log | grep "debug"`
- **按照日期查询**
	- `sed -n '/2023-05-18 14:22:31.070/,/ 2023-05-18 14:27:14.158/p' xx.log`
- **日志太多，处理方式**
	- 分页查询日志信息：`cat -n xx.log |grep "debug" | more`
	- 筛选过滤以后，输出到一个文件：`cat -n xx.log | grep "debug" >debug.txt`

> 前三条较为重要
# 如何排查生产环境
**生产问题怎么排查**
已经上线的bug排查的思路：
1，先分析日志，通常在业务中都会有日志的记录，或者查看系统日志，或者查看日志文件，然后定位问题
2，远程debug（通常公司的正式环境（生产环境）是不允许远程debug的。一般远程debug都是公司的测试环境，方便调试代码）
# 快速定位系统瓶颈
**怎么快速定位系统的瓶颈**
- 压测（性能测试），项目上线之前测评系统的压力
- 监控工具、链路追踪工具，项目上线之后监控
- 线上诊断工具Arthas(阿尔萨斯)，项目上线之后监控、排查

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ceda0a7a47c4a668.png)
