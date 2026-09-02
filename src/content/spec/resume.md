# Java 后端开发

> 💡 **求职意向**：Java 后端开发工程师 ｜ 随时到岗 ｜ 深圳 / 广州 / 杭州 / 上海 / 远程

<div class="flex flex-wrap gap-3 my-4">
  <span class="px-3 py-1 bg-[var(--primary)]/10 text-[var(--primary)] rounded-full text-sm font-medium">👨 男 ｜ 22 岁</span>
  <span class="px-3 py-1 bg-[var(--primary)]/10 text-[var(--primary)] rounded-full text-sm font-medium">📱 15978886186</span>
  <span class="px-3 py-1 bg-[var(--primary)]/10 text-[var(--primary)] rounded-full text-sm font-medium">📧 tommorrow_better@163.com</span>
  <span class="px-3 py-1 bg-[var(--primary)]/10 text-[var(--primary)] rounded-full text-sm font-medium">🐱 <a href="https://github.com/Cxhahalala" target="_blank" class="underline">GitHub 主页</a></span>
</div>

---

## 🎓 教育背景

### **洛阳师范学院** ｜ 本科 ｜ 数据科学与大数据技术
*2022.09 - 2026.09*

- 🏆 **竞赛奖项**：第十五届蓝桥杯软件类 Java 软件开发大学 B 组 **省级三等奖**
- 💼 **在校职务**：在校期间担任实验室项目负责人，主导校园系统研发与迭代

---

## 💼 实习经历

### **东莞市虎门海瑞思特电子科技有限公司** — Java 后端开发实习生
*2025.03 - 2025.06*

- **业务交付**：参与公司进销存管理系统的后端迭代开发，基于 **Spring Boot + MyBatis** 完成采购、库存、订单模块的接口开发与维护，按两周迭代节奏高质量交付业务需求。
- **大文件流式导入导出**：负责管理后台 CRUD 及批量导入导出功能，引入 **EasyExcel 流式读取** 机制，彻底规避传统一次性加载造成的 JVM 大文件 OOM 问题，代码经导师 Review 后平稳上线。
- **排错与 SQL 调优**：参与线上运维与缺陷排查，利用日志与调用链路快速定位空指针及慢 SQL，为高频查询及联表字段补充针对性复合索引；协同测试完成迭代回归，维护完整接口文档。

---

## 🚀 核心项目经历

### 1. 统一优惠券营销平台
*2025.09 - 2026.03 ｜ 核心开发*

- **项目简介**：面向高并发抢券、千万级名单批量分发和订单下单用券核销场景的综合优惠券系统。涵盖模板治理、抢券链路、批量分发与结算核销全流程，利用 Redis Lua、RocketMQ 异步化支撑高并发下的配额原子预扣、领券幂等与耗时任务拆分。
- **技术栈**：Spring Boot 3、Redis、RocketMQ、MySQL、ShardingSphere、Redisson、EasyExcel、Canal、Elasticsearch

#### 核心技术亮点与架构实践：
1. **高并发抢券库存防超卖与防重复**：
   - 采用 **Redis Lua 脚本** 将模板库存判断、原子递减（`HINCRBY`）、用户限领频次校验与计数在单次原子请求中完成，将 99% 的无效流量直接拦截在数据库之前；
   - 数据库侧以 **CAS 条件更新 (`stock > 0`)** 配合 `(user_id, coupon_template_id, receive_count)` **复合唯一索引** 作为最后防线，杜绝超发与重发；
   - 引入 **RocketMQ 异步削峰** 解耦落库操作，并在消费侧自研 **`SET NX GET PX` 幂等切面** 拦截重复消费，遇到异常主动回删幂等键确保消息安全重投。
2. **热点模板查询高可用（防穿透与防击穿）**：
   - 采用 **布隆过滤器 + 空值缓存（TTL 30min）** 双层防护拦截海量恶意与不存在的模板 ID，消除缓存穿透风险；
   - 热点模板缓存过期瞬间，利用 **Redisson 分布式锁 + 锁内双重判定（Double-Checked Locking）** 确保单节点回源重建缓存；
   - 缓存写入通过 **Lua 脚本将 `HMSET` 与 `EXPIREAT` 打包为原子操作**，TTL 严格锚定活动结束时间，杜绝击穿与悬挂 Key。
3. **卡包 ZSet 缓存与延迟关券**：
   - 领券成功后发送 **RocketMQ 延迟消息**，到期时优先从用户卡包 Redis ZSet 中剔除过期券，再以 `status = UNUSED` 作为条件更新 DB，避免误改已锁定或已使用的券。
4. **万人级 Excel 批量发券两级削峰**：
   - 针对运营上传万级名单同步执行易阻塞主线程的问题，设计 **两级 MQ 异步化方案**：一级消息触发 EasyExcel 流式逐行读取并执行 Lua 原子预扣，二级消费者攒批落库，失败记录自动落表并支持导出补发对账。
5. **下单结算试算与用券状态机**：
   - 结算页采用 **Redis Pipeline** 批量预检卡包数据，结合 **`CompletableFuture` 多线程并行** 试算最优优惠组合，大幅压缩响应时延；
   - 用券状态流转（`UNUSED` → `LOCKING` → `USED` 及退款）依托 **分布式锁 + CAS 条件更新 + 编程式事务 (`TransactionTemplate`)** 实现精准边界控制，防状态错乱。

---

### 2. 校园信息服务反馈平台
*2024.09 - 2025.02 ｜ 独立负责人*

- **项目简介**：面向全校师生的教学信息反馈与协同处理平台。实现学生在线提交反馈、学院审批分派、教师整改回复的线上化闭环流转，已在全校上线运行，替代了原有的 QQ 群表格登记流转方式。
- **技术栈**：Spring Boot、Spring Security、JWT、MySQL、Redis、Vue 3、Element UI

#### 核心技术亮点：
1. **多角色 RBAC 权限与数据隔离**：
   - 基于 RBAC 模型划分学生、学院管理员、教师、系统管理员四类角色，结合 **JWT 无状态 Token 与接口级注解鉴权** 控制访问；
   - 实现**学院维度数据权限切分**，学生仅可见本人反馈，学院管理人员仅可查看与处理本院数据，杜绝越权访问。
2. **工单生命周期状态机设计**：
   - 设计工单流转状态机：`待审批` → `已分派` → `整改中` → `已回复` → `已关闭`，每次流转数据库留痕并触发站内即时通知，权责清晰可追溯。
3. **高频接口缓存与防重复提交**：
   - 对高频访问的学院组织架构、系统公告配置 Redis 缓存；针对表单提交、工单审核等敏感写接口引入自定义防重提交拦截。
4. **大文件异步导出**：
   - 针对期末统计反馈报表导出耗时问题，采用 Spring 异步线程池解耦 Excel 文件生成，避免主线程超时。

---

## 🛠️ 专业技能

| 领域 | 核心技术技能 |
| :--- | :--- |
| **Java 基础 & 并发** | 熟练掌握 Java 核心基础、集合源码体系（HashMap/ConcurrentHashMap）；深入理解 JUC 并发编程（CAS、AQS、线程池调优与 `CompletableFuture` 异步编排）。 |
| **框架与微服务** | 熟练使用 Spring Boot、Spring MVC、MyBatis / MyBatis-Plus；熟悉 Spring Cloud Gateway、Nacos 等微服务架构组件。 |
| **缓存与消息中间件** | 熟练掌握 Redis 核心数据结构与高并发场景解决方案（缓存穿透/击穿治理、Lua 原子脚本、Redisson 分布式锁、ZSet 延迟结构）；深入理解 RocketMQ 异步解耦、顺序消费、事务与延迟消息。 |
| **数据库与分库分表** | 熟练掌握 MySQL InnoDB 底层架构、B+ 树索引原理、事务隔离级别与 MVCC；熟练使用 ShardingSphere 进行分库分表实践与慢 SQL 调优。 |
| **安全与工程运维** | 熟练掌握 Spring Security + JWT 权限认证架构；熟悉 Linux 常用运维排查命令与 Docker 容器化部署；熟练使用 Git、Maven、EasyExcel 等工程工具。 |

---

## 🌟 自我评价

- **交付与闭环能力**：具备校园级真实上线项目的全生命周期独立研发与交付经验，具备良好的工程规范与线上问题快速定位能力。
- **深度钻研与原理导向**：习惯深挖技术底层与架构边界，针对高并发库存超卖、双写一致性、幂等重投等难题均对照源码验证并设计容灾补偿机制。
- **积极自驱与团队协作**：热爱技术分享，持续维护个人技术博客与笔记沉淀，具备良好的沟通协同与抗压能力。
