---
title: "Redis的Set结构完整指南"
published: 2025-06-12
description: "Redis Set 与 ZSet 结构完整指南：数据结构底层原理、高频业务场景分析及点赞/共同关注实战代码。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817163236565.png"
tags: ["redis","java"]
category: "Redis"
author: "Cx"
draft: false
---

Redis Set和ZSet数据结构的完整使用指南，包含基本操作、应用场景和实战代码。

<!-- more -->

# Redis Set结构完整指南

## Set结构

### 基本概念
Set是Redis中的无序集合，具有以下特性：
- **唯一性**：元素不重复s
- **无序性**：元素没有固定顺序
- **快速查找**：O(1)时间复杂度判断元素是否存在

### 数据结构
```text
Set 结构：
key: "user:tags"
├── "java"
├── "redis" 
├── "spring"
└── "mysql"
```

### 基本操作

#### Java代码示例
```java
// 添加元素
stringRedisTemplate.opsForSet().add("user:tags", "java", "redis", "spring");

// 判断元素是否存在
Boolean exists = stringRedisTemplate.opsForSet().isMember("user:tags", "java");

// 获取所有元素
Set<String> tags = stringRedisTemplate.opsForSet().members("user:tags");

// 删除元素
stringRedisTemplate.opsForSet().remove("user:tags", "java");

// 获取集合大小
Long size = stringRedisTemplate.opsForSet().size("user:tags");

// 随机获取元素
String randomTag = stringRedisTemplate.opsForSet().randomMember("user:tags");
```

#### Redis命令
```bash
# 添加元素
SADD user:tags java redis spring

# 判断元素是否存在
SISMEMBER user:tags java

# 获取所有元素
SMEMBERS user:tags

# 删除元素
SREM user:tags java

# 获取集合大小
SCARD user:tags

# 随机获取元素
SRANDMEMBER user:tags
```

### 集合操作

#### 交集、并集、差集
```java
// 交集：用户1和用户2共同的标签
Set<String> intersection = stringRedisTemplate.opsForSet()
    .intersect("user:1:tags", "user:2:tags");

// 并集：用户1和用户2所有的标签
Set<String> union = stringRedisTemplate.opsForSet()
    .union("user:1:tags", "user:2:tags");

// 差集：用户1有但用户2没有的标签
Set<String> difference = stringRedisTemplate.opsForSet()
    .difference("user:1:tags", "user:2:tags");
```

### 适用场景
1. **标签系统**：用户标签、商品标签
2. **去重场景**：IP白名单、黑名单
3. **社交关系**：好友列表、粉丝列表
4. **推荐系统**：基于共同兴趣推荐

---

## ZSet结构

### 基本概念
ZSet（有序集合）是Redis中的有序集合，具有以下特性：
- **唯一性**：成员不重复
- **有序性**：按分数自动排序
- **双重索引**：支持按成员查找和按分数范围查找

### 数据结构
```text
ZSet 结构：
key: "blog:liked:1001"
├── member: "user1", score: 1623456789123
├── member: "user2", score: 1623456790456
├── member: "user3", score: 1623456791789
└── 按score升序排列
```

### 内部实现
```text
ZSet 内部架构：
┌─────────────────────────────────────┐
│              ZSet                   │
├─────────────────┬───────────────────┤
│   Hash Table    │    Skip List      │
│  (字典/dict)     │   (跳跃表/zsl)     │
├─────────────────┼───────────────────┤
│ member → score  │ score → member    │
│    O(1)查找     │   O(log n)范围     │
└─────────────────┴───────────────────┘
```

### 基本操作

#### Java代码示例
```java
// 添加元素
stringRedisTemplate.opsForZSet().add("blog:liked:1001", "user123", System.currentTimeMillis());

// 批量添加
Set<ZSetOperations.TypedTuple<String>> tuples = new HashSet<>();
tuples.add(ZSetOperations.TypedTuple.of("user1", 100.0));
tuples.add(ZSetOperations.TypedTuple.of("user2", 90.0));
stringRedisTemplate.opsForZSet().add("leaderboard", tuples);

// 获取分数
Double score = stringRedisTemplate.opsForZSet().score("blog:liked:1001", "user123");

// 获取排名（升序，从0开始）
Long rank = stringRedisTemplate.opsForZSet().rank("leaderboard", "user1");

// 获取排名（降序，从0开始）
Long reverseRank = stringRedisTemplate.opsForZSet().reverseRank("leaderboard", "user1");

// 范围查询详解
// 假设leaderboard数据：user3->78分, user1->85分, user5->88分, user2->92分, user4->95分

// 1. 按排名查询（升序：从低分到高分）
Set<String> range = stringRedisTemplate.opsForZSet().range("leaderboard", 0, 4);
// 参数：(key, start排名, end排名) - 排名从0开始
// 结果：["user3", "user1", "user5", "user2", "user4"] (78→95分顺序)

// 2. 按排名查询（降序：从高分到低分）- 真正的排行榜前5名
Set<String> top5 = stringRedisTemplate.opsForZSet().reverseRange("leaderboard", 0, 4);
// 结果：["user4", "user2", "user5", "user1", "user3"] (95→78分顺序)

// 3. 按分数查询（升序）
Set<String> scoreRange = stringRedisTemplate.opsForZSet()
    .rangeByScore("leaderboard", 85, 95);
// 参数：(key, 最小分数, 最大分数)
// 结果：["user1", "user5", "user2", "user4"] (85→95分范围，升序)

// 4. 按分数查询（降序）
Set<String> highScores = stringRedisTemplate.opsForZSet()
    .reverseRangeByScore("leaderboard", 85, 95);
// 结果：["user4", "user2", "user5", "user1"] (95→85分范围，降序)

// 删除元素
stringRedisTemplate.opsForZSet().remove("blog:liked:1001", "user123");

// 增加分数
stringRedisTemplate.opsForZSet().incrementScore("leaderboard", "user1", 10.0);
```

#### Redis命令
```bash
# 添加元素
ZADD blog:liked:1001 1623456789123 user123

# 获取分数
ZSCORE blog:liked:1001 user123

# 获取排名
ZRANK leaderboard user1

# 范围查询
ZRANGE leaderboard 0 4
ZREVRANGE leaderboard 0 4  # 降序

# 按分数范围查询
ZRANGEBYSCORE leaderboard 90 100

# 删除元素
ZREM blog:liked:1001 user123
```

#### ZSet方法对照表

| 查询类型 | 升序方法 | 降序方法 | 参数说明 | 使用场景 |
|---------|---------|---------|---------|---------|
| 按排名 | `range(key, start, end)` | `reverseRange(key, start, end)` | start/end是排名位置 | 获取前N名或特定排名段 |
| 按分数 | `rangeByScore(key, min, max)` | `reverseRangeByScore(key, min, max)` | min/max是分数值 | 获取特定分数段的成员 |

**重要提示：**
- 排行榜应用通常使用 `reverseRange`（降序），因为我们需要"分数高的排在前面"
- `range` 系列方法默认都是升序（从低分到高分）
- 排名从0开始计算

### 操作复杂度
| 操作类型 | 时间复杂度 | 示例 |
|---------|-----------|------|
| 添加/更新 | O(log n) | ZADD |
| 查找分数 | O(1) | ZSCORE |
| 范围查询 | O(log n + m) | ZRANGE |
| 排名查询 | O(log n) | ZRANK |
| 删除元素 | O(log n) | ZREM |

### 适用场景
1. **排行榜系统**：游戏排行榜、销量排行
2. **时间序列**：按时间排序的数据
3. **优先队列**：任务调度、消息队列
4. **点赞系统**：按时间排序的点赞用户

---

## 实战案例：博客点赞系统

### 场景描述
实现一个博客点赞功能，需要：
- 记录谁点赞了
- 按点赞时间排序
- 快速查询用户是否点赞
- 获取最早点赞的用户列表

### 核心代码实现

#### 点赞功能
```java
@Override
public Result likeBlog(Long id) {
    // 获取当前用户
    Long userId = UserHolder.getUser().getId();
    String key = RedisConstants.BLOG_LIKED_KEY + id;
    
    // 检查是否已点赞
    Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
    
    if (score == null) {
        // 未点赞，执行点赞
        boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
        if (isSuccess) {
            // 写入Redis，使用时间戳作为分数
            stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
        }
    } else {
        // 已点赞，取消点赞
        boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
        if (isSuccess) {
            stringRedisTemplate.opsForZSet().remove(key, userId.toString());
        }
    }
    
    return Result.ok();
}
```

#### 查询点赞用户
```java
@Override
public Result queryBlogLikes(Long id) {
    String key = RedisConstants.BLOG_LIKED_KEY + id;
    
    // 获取最早点赞的前5个用户
    Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);
    
    if (top5 == null || top5.isEmpty()) {
        return Result.ok(Collections.emptyList());
    }
    
    // 类型转换和异常处理
    List<Long> ids = top5.stream()
        .map(item -> {
            try {
                return Long.valueOf(item);
            } catch (NumberFormatException e) {
                return null;
            }
        })
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
    
    if (ids.isEmpty()) {
        return Result.ok(Collections.emptyList());
    }
    
    // 查询用户信息，保持顺序
    String orderByClause = "ORDER BY FIELD(id," + 
        ids.stream().map(String::valueOf).collect(Collectors.joining(",")) + ")";
    
    List<UserDTO> userDTOS = userService.query()
        .in("id", ids)
        .last(orderByClause)
        .list()
        .stream()
        .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
        .collect(Collectors.toList());
    
    return Result.ok(userDTOS);
}
```

#### 检查点赞状态
```java
@Override
public void isBlogLiked(Blog blog) {
    UserDTO user = UserHolder.getUser();
    if (user == null) {
        return;
    }
    
    String key = RedisConstants.BLOG_LIKED_KEY + blog.getId();
    Double score = stringRedisTemplate.opsForZSet().score(key, user.getId().toString());
    blog.setIsLike(score != null);
}
```

### 关键技术点

#### 1. 使用时间戳作为分数
```java
// 使用当前时间戳，确保按点赞时间排序
stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
```

#### 2. 流处理优化
```java
// 安全的类型转换
List<Long> ids = top5.stream()
    .map(item -> {
        try {
            return Long.valueOf(item);
        } catch (NumberFormatException e) {
            log.warn("Invalid user id: {}", item);
            return null;
        }
    })
    .filter(Objects::nonNull)
    .collect(Collectors.toList());
```

#### 3. 保持查询顺序
```java
// 使用 FIELD 函数保持Redis中的顺序
String orderByClause = "ORDER BY FIELD(id," + 
    ids.stream().map(String::valueOf).collect(Collectors.joining(",")) + ")";
```

---

## Set vs ZSet 对比

| 特性 | Set | ZSet |
|------|-----|------|
| **排序** | ❌ 无序 | ✅ 有序 |
| **分数** | ❌ 无 | ✅ 有 |
| **查找复杂度** | O(1) | O(log n) |
| **范围查询** | ❌ 不支持 | ✅ 支持 |
| **内存占用** | 较低 | 较高 |
| **适用场景** | 去重、标签 | 排行榜、时间序列 |

## 最佳实践

### 1. 选择合适的数据结构
- **需要排序**：选择 ZSet
- **只需去重**：选择 Set
- **需要范围查询**：选择 ZSet

### 2. 性能优化
- **批量操作**：使用批量添加方法
- **合理设置过期时间**：避免内存泄漏
- **分页查询**：大数据量时分页获取

### 3. 错误处理
- **空值检查**：处理null返回值
- **类型转换**：安全的字符串转换
- **异常处理**：捕获并记录异常

### 4. 代码示例模板
```java
// Set操作模板
public void setOperation(String key, String value) {
    try {
        stringRedisTemplate.opsForSet().add(key, value);
    } catch (Exception e) {
        log.error("Set operation failed: {}", e.getMessage());
    }
}

// ZSet操作模板
public void zsetOperation(String key, String member, double score) {
    try {
        stringRedisTemplate.opsForZSet().add(key, member, score);
    } catch (Exception e) {
        log.error("ZSet operation failed: {}", e.getMessage());
    }
}
```

---

## 总结

Redis的Set和ZSet是两种强大的集合数据结构：

- **Set**：适合去重、标签管理、集合运算等场景
- **ZSet**：适合排行榜、时间序列、优先队列等需要排序的场景

选择合适的数据结构，配合正确的使用方法，能够大幅提升应用性能和用户体验。在实际项目中，要注意异常处理、性能优化和代码的可维护性。
