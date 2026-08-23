---
title: "MySQL 核心面试题：索引底层、深分页、事务与 MVCC 实战解析"
published: 2026-08-22
description: "深入分析 MySQL 慢查询定位、B+ 树索引结构、聚集与非聚集索引（回表查询）、覆盖索引、千万级超大分页优化、最左匹配原则与索引失效、redo/undo log、并发事务与 MVCC 多版本并发控制。"
tags: ["MySQL","数据库","SQL优化","MVCC","面试"]
category: "面试"
draft: false
---

# 如何定位慢查询
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783833094027-ac054202-4271-4347-8548-e754cbfa72f5.png)
## 回答
注意没有用过skywalking可以不说
而开启mysql的慢查询日志通常在调试阶段，因为会损耗mysql性能
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783833159668-1a86c170-e8e1-49d9-bfa0-0b0401b2d138.png)
# 如何优化慢索引
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783833672300-13ae4eff-f8e2-4ae2-8742-dca7ce9fab54.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783833810617-2ee03061-6aed-45d8-b9b8-4570a179460a.png)
如果出现useing index condition代表索引使用有优化空间
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783833939720-e7373134-5a14-4ae7-8c9a-cfcd4021ac43.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783834012561-6594739c-20c1-4bc6-b9e4-1a50ca860915.png)
# 索引
## 什么是索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783834207308-c8b68b4b-b234-499c-ac8f-e33a545f6a33.png)
这里是以二叉搜索树示例
## 索引的底层数据结构
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783836515561-910176a1-91cf-4446-a8f4-f33f0c2722e5.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783836162149-dc495ed7-521f-4245-a2f5-882a66b00318.png)
磁盘读写代价低时因为非叶子节点只是存储指针，在便利的时候不需要每个节点都加载数据
查询效率稳定，数据都寸在叶子节点上
便于扫库和区间检查，因为叶子节点也是一个双向链表，不需要再从根节点遍历
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783836261565-c58b2075-723f-4362-bb5a-6c7e879a8aa6.png)
# 聚集索引和非聚集索引
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783843684309-33ba2b07-7f73-4c97-acc4-47ad11ffcb03.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783843778662-00a4f7af-532f-4323-bf67-151ca310e2ef.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783843890230-75d84dcf-0e18-4f24-b3e2-e37502b8cdd5.png)
简单来说，先通过二级索引拿到对应的主键，再通过主键获取真正的数据。相当于读了两次表，就叫做回表查询
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844040181-98fa1602-3bc9-45c0-ac05-925b5f2821e9.png)
面试官可能会问，什么是回表查询，此时就可以先解释什么是聚簇索引和非聚簇索引，然后解释回表查询
# 覆盖索引
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844348113-bbfda965-5f35-45cc-99ab-c6661a89b438.png)
第一个是直接通过主键，是聚簇索引，叶子节点存储全部的数据，不需要回表查询，即覆盖索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844650026-7b9a4c4a-53f5-447f-807e-778fb3c396bb.png)
第二个通过name字段，二级索引叶子节点刚好也存储了主键，刚好要找的是name 和 id，所以也是聚簇索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844628862-c05d3bb7-b4f0-4a08-8b2c-eeec853ef165.png)
第三个，索引并没有gender，因此触发回表查询，通过id又要查一次表，所以为非聚簇索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844700762-927f32fb-a745-4fcf-ba2b-53f0e468e59a.png)
回表查询性能相对来说较低
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844834189-bd83c55a-fe39-4fe3-9cf5-4476980104d7.png)
# 如何处理超大分页
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783844935226-e4f5b01b-8d3d-4974-aed8-7c3f96b5e457.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783845086269-b7bbe83e-db56-4366-9709-79c92cd95900.png)
📝 本次对话核心总结：MySQL 深分页原理与优化
核心认知纠偏
子查询未消除 Offset：LIMIT 9000000, 10 的子查询写法依然要遍历前 900 万条记录，并没有“跳过”这一步。
Offset 不是跳转而是“数数”：MySQL 没有物理行号，也无法预知第 N 条记录的 ID。它只能沿着 B+ 树叶子节点的双向链表逐条线性扫描，内部计数器达到 offset 后才开始返回数据。时间复杂度为 O(offset)。
为什么子查询更快？
本质是将“重操作”转化为“轻操作”：
对比维度 原查询 SELECT \* 优化查询（子查询 + JOIN）
跳过阶段 遍历索引 + 回表读完整行（或扫描巨大的聚簇索引页） 仅遍历索引，只读 id（覆盖索引，纯内存/索引页操作）
取数阶段 已在跳过过程中完成 用 10 个 id 精准回表 10 次
性能差异根源 海量磁盘 I/O（读数据页） 极少量 I/O + CPU 扫索引页，快数十倍
⚠️ 回表触发条件：当使用二级索引时，叶子节点只有 索引列 + 主键id，SELECT \* 必须拿 id 去聚簇索引二次查找完整行，900 万次回表是性能崩塌的主因。若走聚簇索引虽无需回表，但页大行少，I/O 量依然巨大。
终极解决方案：游标分页
彻底抛弃 LIMIT offset，改用 Keyset Pagination：
SELECT \* FROM tb_sku WHERE id \> \{last_id\} ORDER BY id LIMIT 10;
利用 B+ 树 O(log n) 精确定位，将“线性扫描 900 万次”变为“树查找 1 次”。
无论翻到第几页，耗时恒定，是生产环境解决深分页的根本之道。
💡 一句话精华
LIMIT offset 的本质是沿链表逐条“数数”，子查询只是让“数数”时读的数据变小了，而游标分页才是真正把“数数”变成了“跳转”。
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783845143356-de3bd8dd-6946-400c-aef1-65d6ee5c82cd.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783845187432-b3b187fc-2765-473b-b7d7-93b2c794f7c1.png)
# 创建索引的原则
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783846478724-9507bad2-c3ef-4268-a0af-56bc601a074e.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783846517653-cb6b3937-ffb3-4c54-943d-3e8eec62d401.png)
截取一部分作为前缀索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783846612734-78c83dd0-aa2d-4cb4-9b12-d0c82dc44001.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783846659292-e3119c1a-74b5-4b24-be0b-3c41cfb2229b.png)
## 聚合索引相关
```yaml
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    age INT,
    department VARCHAR(50),
    salary DECIMAL(10,2)
);
-- 创建联合索引：(department, age, salary)
CREATE INDEX idx_dept_age_salary ON employees(department, age, salary);
```
**避免回表查询**
```yaml
EXPLAIN SELECT department, age, salary FROM employees WHERE department = '技术部' AND age > 25;
```
**最左侧原则**
<table header-row="true">
<tr>
<td>**查询条件**</td>
<td>**能否走索引**</td>
<td>**说明**</td>
</tr>
<tr>
<td>`**WHERE department = 'X'**`</td>
<td>✅</td>
<td>**用了第1列**</td>
</tr>
<tr>
<td>`**WHERE department = 'X' AND age = 25**`</td>
<td>✅</td>
<td>**用了前2列**</td>
</tr>
<tr>
<td>`**WHERE department = 'X' AND age = 25 AND salary > 5000**`</td>
<td>✅</td>
<td>**3列全用**</td>
</tr>
<tr>
<td>`**WHERE age = 25**`</td>
<td>❌</td>
<td>**跳过了第1列，索引失效**</td>
</tr>
<tr>
<td>`**WHERE department = 'X' AND salary > 5000**`</td>
<td>⚠️\*\* 部分\*\*</td>
<td>**只用 department，salary 因中间断了 age 而无法用于索引查找（但可能用于索引过滤）**</td>
</tr>
<tr>
<td>`**WHERE department = 'X' ORDER BY age**`</td>
<td>✅</td>
<td>**排序也能利用索引有序性，避免 filesort**</td>
</tr>
</table>
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783846707103-c3b3ec73-85b5-49aa-b3ce-f663455da100.png)
# 索引失效
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847314771-5677ed9d-c7ef-4031-9828-3db032c1529a.png)
这三个是按照索引顺序来的，所以可以命中索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847417289-58a33a52-7286-4ed3-8a96-e6c65639ac48.png)
第二条跳过了一条，只有最左侧索引生效，所以key_len与只使用name的时候一样
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847525912-b226c5e4-656a-4e56-a437-847560402a91.png)
第二条status使用了范围查询，因此address失效，adress未命中索引
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847632118-d6caf2b0-6261-4f30-b455-11693617482f.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847692898-263652d4-a50a-48dd-94b4-161e3c4041a3.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847738762-1697f0c5-0c8a-479c-b345-a704457ecc4d.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783847826215-c2b4fb33-18a3-4a75-845c-ed05c10d2632.png)
# Sql优化
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848015582-e73db08b-c057-480d-96be-968db656b7bc.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848152115-e302eab2-692c-4396-88c5-5285504713fa.png)
小表放外面，比如小表只有10行，大表都100行，那么只需要连接数据库3次，每次操作100次
如果大表在外面，那么就要连接数据库100次，效率低
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848314183-89c0c1ee-f9f9-4fed-98c9-216a83fd51a0.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848385278-3d5f7e24-331c-45bc-a2bc-f3793fd0352a.png)
分库分表一般发生在大的数据量中，后面补充
# 什么是事务
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848519468-b2048028-b1e0-4c19-be7c-04e6c2243b3d.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848582554-46107b5a-33a9-41a6-b58c-b5d357947876.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848609115-d96efa69-3997-4555-8e52-f982a5212fe8.png)
# 并发事务问题
## 问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848674012-a273eeed-3ebd-4b2a-870c-cb1b7ff35405.png)
##
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848843707-1a3251ce-c7a1-4e7f-8279-d09183035787.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783848951533-03db5421-1f9b-4af6-acc1-9f45d6bb5d1b.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783849087697-df1e7603-d6d2-43b6-a6b6-c956fcff8cc4.png)
解决了不可重复读的问题后，读数据不存在，插入数据又失败，再读还是不存在，就出现了幻读问题
## 事务隔离性
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783849355676-8c48faa0-d6a0-4826-969c-cf2be30b7ae6.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783849511712-e7886d0b-7476-467c-9840-0806a266057b.png)
# 事务undo log与redo log
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851272883-7c867651-6b03-4090-9ad6-a052da746d8e.png)
如果数据库宕机了，那么内存数据可能就无法存储到磁盘中
无法持久化
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851462332-5a1d027b-1814-4811-8e07-29fd0908d948.png)
当内存数据发生变化立刻写入到硬盘不是也可以吗？原理上可以，但是当发生大量的增删改查，那么就需要进行频繁的磁盘io损耗性能
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851638246-5ab168ad-2a4f-4cef-8a53-59cb68d83ec5.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851679975-b59b1a70-2909-491e-bfc7-d6ec226c6e4b.png)
# Mvcc
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851881091-11620040-1dca-4777-9cc6-6b5e816c4dbd.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783851985670-dbf3ec20-4da2-4732-88da-116b396de681.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852062049-e4cb9c27-fa7a-4bff-98c9-1e1d99e57d2d.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852296057-a5b7a0e9-4f9e-48c2-a773-4e31de72bc28.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852473566-f90ba17a-0708-4027-b937-c1d7590b1af4.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852555680-fa090f62-fa05-4631-9517-28e48506a170.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852667299-4c00a97d-4c77-4327-9f9b-46ca54225f99.png)
这个不用记忆
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852743585-33d10958-b113-4ddb-a736-d05da0619aa6.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852893737-8c5c8939-2de5-4d3f-a4ee-f628830b3145.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783852944645-c7a58293-cba1-4449-81e3-50371e6b98ca.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853154083-b35e9702-40c0-4a6a-a382-3e6e0bc1e5f3.png)
# Mysql主从同步原理
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853282712-c4854a3f-343a-40fe-be52-e75fe4baa374.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853385724-1a476e15-35f0-4864-be9f-c6c976d6df98.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853443117-5a4cab50-4858-4867-8363-dd459be1939a.png)
# 分库分表
## 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853589107-13058ba6-e819-4e41-a90e-4a28ab973290.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853635051-68aea444-63db-4504-9bfa-8aba070ca3ee.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853732749-fd9d2805-296c-4366-9d4a-ca33fb1ee66c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853857424-099b6f25-6c67-4406-bc04-2bdab4556338.png)
垂直分表不一定会把一个表分到两个数据库中
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783853998904-a735ba0c-b000-4e4f-a6fe-a19945d43b15.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783854073590-38027fdd-32b0-4059-9756-dbbae3ccd785.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783854178550-5ef66c76-d11d-4d21-b55b-36a8499b0685.png)
## 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1783854301881-d0bd659c-9a9a-4f43-933f-9ff12c5d5a48.png)