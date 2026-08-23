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
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/423d9a91feeb515e.png)
## 回答
注意没有用过skywalking可以不说
而开启mysql的慢查询日志通常在调试阶段，因为会损耗mysql性能
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3d302903509b7522.png)
# 如何优化慢索引
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/439306f4c950f13d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/28b7c798fd0e9111.png)
如果出现useing index condition代表索引使用有优化空间
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0d599b4533b49b00.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8068fc962b913fda.png)
# 索引
## 什么是索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c55fb5283e5192bb.png)
这里是以二叉搜索树示例
## 索引的底层数据结构
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c38233161c0489ad.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/cb3065a33d3b81c3.png)
磁盘读写代价低时因为非叶子节点只是存储指针，在便利的时候不需要每个节点都加载数据
查询效率稳定，数据都寸在叶子节点上
便于扫库和区间检查，因为叶子节点也是一个双向链表，不需要再从根节点遍历
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/75ac0c6a4c8f2b08.png)
# 聚集索引和非聚集索引
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1599efc907d4ba81.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/14bdc9af5f45b1a5.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/586a663447588bc6.png)
简单来说，先通过二级索引拿到对应的主键，再通过主键获取真正的数据。相当于读了两次表，就叫做回表查询
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3af181fd5d9d2ecf.png)
面试官可能会问，什么是回表查询，此时就可以先解释什么是聚簇索引和非聚簇索引，然后解释回表查询
# 覆盖索引
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a4f52732d2e28f95.png)
第一个是直接通过主键，是聚簇索引，叶子节点存储全部的数据，不需要回表查询，即覆盖索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8d75380af2014c21.png)
第二个通过name字段，二级索引叶子节点刚好也存储了主键，刚好要找的是name 和 id，所以也是聚簇索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/55ef1616590e174d.png)
第三个，索引并没有gender，因此触发回表查询，通过id又要查一次表，所以为非聚簇索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a4f6c8f2c6657bac.png)
回表查询性能相对来说较低
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/efdc3ce84467d722.png)
# 如何处理超大分页
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1f70b8e95203015d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/65b1b1f130cd4757.png)
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
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9fe2cd27235e8125.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/641bb8b893caf986.png)
# 创建索引的原则
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/bf8f8ff24a55e8ad.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9ed0dce64c163ef6.png)
截取一部分作为前缀索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f47fd73fd4ed253c.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/26f4df386f7a1b02.png)
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
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/678beebc20043ade.png)
# 索引失效
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/09cd9b4f99b10417.png)
这三个是按照索引顺序来的，所以可以命中索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7333f449bce17cd9.png)
第二条跳过了一条，只有最左侧索引生效，所以key_len与只使用name的时候一样
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ea2d5a4a01f7cc1e.png)
第二条status使用了范围查询，因此address失效，adress未命中索引
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/4cb5a7f1829a5c02.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/dc55920f3209c009.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/be380695673912e9.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/47046ca2b37925a0.png)
# Sql优化
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/faa0d243119c4983.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d56a28ffe6509110.png)
小表放外面，比如小表只有10行，大表都100行，那么只需要连接数据库3次，每次操作100次
如果大表在外面，那么就要连接数据库100次，效率低
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8d067b6c9a7198a3.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f6c171dc63f8cbda.png)
分库分表一般发生在大的数据量中，后面补充
# 什么是事务
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5ce62eeb02b0a7f0.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/83d382011e7d7ecf.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/14871b20b7904811.png)
# 并发事务问题
## 问题
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c1489d27093e263b.png)
##
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3ef5a5ae57121764.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/12a838758f37393d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/12b1e2656b0f9d7e.png)
解决了不可重复读的问题后，读数据不存在，插入数据又失败，再读还是不存在，就出现了幻读问题
## 事务隔离性
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3c173decb245b1c3.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/90bc94be8374fd7a.png)
# 事务undo log与redo log
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9820bac9fe2d6c2a.png)
如果数据库宕机了，那么内存数据可能就无法存储到磁盘中
无法持久化
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/725d70c831995799.png)
当内存数据发生变化立刻写入到硬盘不是也可以吗？原理上可以，但是当发生大量的增删改查，那么就需要进行频繁的磁盘io损耗性能
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c04a9512a9aa110e.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7b08ff4a0c2f51c5.png)
# Mvcc
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/af86d0cc352843e0.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ee76eb4389da50df.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a2907378d50f2dfb.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/071024a445ffdb99.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d119874d2ef28ddc.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0aadef2ed05c9ebe.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/78218db11373f8bf.png)
这个不用记忆
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7632e52ed99ef84e.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/be9b2b814eb847b7.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2a42233fe7d035ce.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3ec0cce032f13ee1.png)
# Mysql主从同步原理
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/42ee858eb34be05d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/330745b139fc2d8d.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/befb7c09b40098f9.png)
# 分库分表
## 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/365f4a07a68a4266.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3300c1efde3ef51f.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/dfec87c11ab5137e.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/03bc9b0ef69de4ce.png)
垂直分表不一定会把一个表分到两个数据库中
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/13c1e936eca1ec2d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2513da2d0d927494.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a5f9f55b81fbb903.png)
## 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1577f0e51591355c.png)