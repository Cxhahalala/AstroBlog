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

### 核心痛点与底层机制深度剖析

#### 1. 传统八股写法辨析（为什么 `ORDER BY id` 容易误导？）
很多市面教程（包括上图截取的传统题解示例）常写成：

```sql
-- 常见八股写法（容易掩盖问题本质）
SELECT * FROM tb_sku t, (
    SELECT id FROM tb_sku ORDER BY id LIMIT 9000000, 10
) a WHERE t.id = a.id;
```

> ⚠️ **认知纠偏**：如果排序字段本来就是主键 `id`，走的就是聚簇索引树，叶子节点本来就存放了整行数据，谈不上“利用覆盖索引避免回表”。
> 
> **“覆盖索引 + 延迟关联”真正发挥威力的核心业务场景，是按非主键的二级索引列排序或过滤**（例如按创建时间 `create_time`、更新时间或业务状态排序的深度分页）。

---

#### 2. 核心痛点：海量“无效回表”
在电商、订单、日志等实际业务中，最典型的深分页是按业务字段排序：

```sql
-- 原始慢 SQL：耗时可能达到数十秒甚至直接查询超时
SELECT * FROM tb_sku ORDER BY create_time LIMIT 9000000, 10;
```

- **瓶颈所在**：
  1. 表中建立了二级索引 `idx_create_time(create_time)`，二级索引叶子节点仅存储 `(create_time, id)`；
  2. 为了获取 `SELECT *` 的全部列，MySQL 顺着二级索引 B+ 树叶子节点的双向链表逐条线性扫描；
  3. **每扫描一条记录，就必须拿着对应的 `id` 去聚簇索引回表一次**以读取完整行数据；
  4. **崩塌结果**：整整执行了 **9,000,010 次回表查询**！把 900 万行沉重的完整数据页从磁盘加载到 Buffer Pool 后又全量丢弃，产生海量随机 I/O 与严重的缓存污染。

---

#### 3. 核心优化方案：覆盖索引 + 延迟关联（子查询）

```sql
-- 优化后 SQL：耗时从数十秒大幅降低至数百毫秒
SELECT * FROM tb_sku t JOIN (
    SELECT id FROM tb_sku ORDER BY create_time LIMIT 9000000, 10
) a ON t.id = a.id;
```

##### 优化本质：将“重操作”转化为“轻操作”

| 对比维度 | 原始查询 (`SELECT *`) | 优化查询 (子查询 + JOIN 延迟关联) |
| :--- | :--- | :--- |
| **跳过阶段 (前 900 万行)** | 遍历二级索引 + **900 万次回表读完整行**（巨额随机 I/O） | **0 次回表**！二级索引天然包含 `create_time` 和 `id`，满足覆盖索引，纯走轻量级二级索引树 |
| **取数阶段 (最终 10 行)** | 已在前期连带回表读出 | 仅拿最终这 10 个目标 `id` 进行 **10 次精准主键回表** 取完整行 |
| **性能差异根源** | 900 万次随机磁盘 I/O + 缓存置换 | 极少量连续索引页 I/O + 仅 10 次点查回表，性能提升数十倍 |

> 💡 **核心认知纠偏**：
> - **子查询并未消除 offset**：`LIMIT 9000000, 10` 的子查询依然需要遍历前 900 万条记录，并没有“瞬间跳过”。
> - **Offset 不是跳转而是“数数”**：MySQL 没有物理行号，也无法预知第 N 条记录的 ID，只能顺着链表逐条线性扫描计数。
> - **子查询为什么快？**：因为它走的是覆盖索引，索引页体积小且常驻内存，**它把原本极其昂贵的“900 万次磁盘回表”变成了极轻量的“纯内存数数”**。

---

#### 4. 补充：如果真的纯按主键 `id` 排序怎么办？

当业务确实仅按主键 `id` 排序时，无法通过二级索引来消除回表，此时的**终极最优解是游标分页（Keyset Pagination / 标签分页）**：

```sql
-- 游标分页（记录上一页最后一条的主键 ID）
SELECT * FROM tb_sku WHERE id > 9000000 ORDER BY id LIMIT 10;
```

- **执行机制**：利用主键 B+ 树的平衡二分查找特性，仅需 $O(\log N)$ 步树遍历直接定位到目标叶子节点，随后向后顺序读取 10 条数据。
- **性能优势**：彻底避免了前 900 万条记录的线性扫描，**无论翻到第 1 页还是第 1000 万页，耗时均恒定在毫秒级**。

---

## 回答

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9fe2cd27235e8125.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/641bb8b893caf986.png)

> **面试满分回答话术梳理**：
>
> 1. **切中痛点**：深分页（如 `LIMIT 9000000, 10`）变慢的本质是**海量无效回表**。以按业务列（如 `create_time`）排序为例，为了获取全列数据，MySQL 在二级索引扫描时每遇到一条记录都要回表一次，导致前 900 万条产生了 900 万次无效的随机 I/O，最终数据却被全部丢弃。
> 2. **给出解法（覆盖索引 + 延迟关联）**：使用子查询先查 `SELECT id FROM tb_sku ORDER BY create_time LIMIT 9000000, 10`。因为二级索引叶子节点本身包含排序列与主键 `id`，**命中覆盖索引，前 900 万行实现 0 次回表**；然后再通过主键 `JOIN` 原表，仅对最终的 10 条数据进行 **10 次精准回表**，大幅降低 I/O 消耗。
> 3. **高阶进阶（游标分页）**：指出子查询依然需要线性遍历 900 万次索引记录。若业务允许（如瀑布流、滚动加载），或排序字段本身就是主键 `id`，最优方案是采用**游标分页（`WHERE id > last_id LIMIT 10`）**，利用 B+ 树 $O(\log N)$ 步直接定位目标行，彻底消除分页性能瓶颈。
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