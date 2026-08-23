---
title: "利用Redis的BitMap实现用户签到功能"
published: 2025-06-18
description: "利用 Redis 的 BitMap 实现亿级用户签到功能：传统表结构瓶颈分析与海量数据位运算低成本存储实战。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817162903558.png"
tags: ["redis","java"]
category: "Redis"
author: "Cx"
draft: false
---

Redis的BitMap结结构，利用此结构实现用户签到功能
<!--more-->
# BitMap功能演示

## 传统方案的问题

我们针对签到功能完全可以通过MySQL来完成，比如说以下这张表：
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250617192125390.png)

**存储成本分析：**
- 用户一次签到，就是一条记录
- 假如有1000万用户，平均每人每年签到次数为10次，则这张表一年的数据量为 **1亿条**
- 每签到一次需要使用（8 + 8 + 1 + 1 + 3 + 1）共**22字节**的内存，一个月则最多需要**600多字节**

## BitMap解决方案

### 什么是BitMap？

我们如何能够简化一点呢？其实可以考虑小时候一个挺常见的方案，就是小时候，咱们准备一张小小的卡片，你只要签到就打上一个勾，我最后判断你是否签到，其实只需要到小卡片上看一看就知道了。

我们可以采用类似这样的方案来实现我们的签到需求：

- 我们按月来统计用户签到信息，**签到记录为1，未签到则记录为0**
- 把每一个bit位对应当月的每一天，形成了映射关系
- 用0和1标示业务状态，这种思路就称为**位图（BitMap）**
- 这样我们就用极小的空间，来实现了大量数据的表示

### BitMap的优势

**存储空间对比：**
- 传统方案：每天22字节，一个月最多682字节
- BitMap方案：一个月最多31位 = 4字节
- **节省存储空间约99%**

### Redis中的BitMap

Redis中是利用**string类型**数据结构实现BitMap，因此最大上限是**512M**，转换为bit则是 **2^32个bit位**。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250617192159218.png)

## BitMap常用命令详解

BitMap的操作命令有：

### 基础操作命令

#### 1. SETBIT - 设置位值
```redis
SETBIT key offset value
```
- **作用**：向指定位置（offset）存入一个0或1
- **示例**：`SETBIT user:1001:202506 0 1` （设置6月1号已签到）

#### 2. GETBIT - 获取位值  
```redis
GETBIT key offset
```
- **作用**：获取指定位置（offset）的bit值
- **示例**：`GETBIT user:1001:202506 0` （获取6月1号签到状态）

#### 3. BITCOUNT - 统计1的个数
```redis
BITCOUNT key [start end]
```
- **作用**：统计BitMap中值为1的bit位的数量
- **示例**：`BITCOUNT user:1001:202506` （统计6月总签到天数）

### 高级操作命令

#### 4. BITFIELD - 批量位操作
```redis
BITFIELD key GET type offset [GET type offset ...]
BITFIELD key SET type offset value [SET type offset value ...]
```
- **作用**：操作（查询、修改、自增）BitMap中bit数组中的指定位置（offset）的值
- **示例**：`BITFIELD user:1001:202506 GET u15 0` （获取前15天的签到数据）

#### 5. BITFIELD_RO - 只读批量获取
```redis
BITFIELD_RO key GET type offset [GET type offset ...]
```
- **作用**：获取BitMap中bit数组，并以十进制形式返回（只读版本）

#### 6. BITOP - 位运算操作
```redis
BITOP operation destkey key [key ...]
```
- **作用**：将多个BitMap的结果做位运算（与、或、异或）
- **operation**：AND、OR、XOR、NOT
- **示例**：`BITOP AND result user:1001:202506 user:1002:202506`

#### 7. BITPOS - 查找特定位
```redis
BITPOS key bit [start] [end]
```
- **作用**：查找bit数组中指定范围内第一个0或1出现的位置
- **示例**：`BITPOS user:1001:202506 1` （查找第一次签到的位置）

## Java中BitMap操作详解

### 环境准备

在Java中使用Redis BitMap需要引入相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 基础BitMap操作

#### 1. 设置位值 - setBit
```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

/**
 * 设置指定位置的bit值
 * @param key Redis键
 * @param offset 位置偏移量
 * @param value 要设置的值(true/false)
 * @return 设置前该位置的值
 */
public Boolean setBit(String key, long offset, boolean value) {
    return stringRedisTemplate.opsForValue().setBit(key, offset, value);
}

// 使用示例
public void userSignIn(Long userId) {
    LocalDateTime now = LocalDateTime.now();
    String key = "user:sign:" + userId + ":" + now.format(DateTimeFormatter.ofPattern("yyyyMM"));
    int dayOfMonth = now.getDayOfMonth();
    
    // 设置今天已签到
    setBit(key, dayOfMonth - 1, true);
    
    // 设置过期时间(32天)
    stringRedisTemplate.expire(key, Duration.ofDays(32));
}
```

#### 2. 获取位值 - getBit
```java
/**
 * 获取指定位置的bit值
 * @param key Redis键
 * @param offset 位置偏移量
 * @return 该位置的值
 */
public Boolean getBit(String key, long offset) {
    return stringRedisTemplate.opsForValue().getBit(key, offset);
}

// 使用示例
public boolean isUserSignedToday(Long userId) {
    LocalDateTime now = LocalDateTime.now();
    String key = "user:sign:" + userId + ":" + now.format(DateTimeFormatter.ofPattern("yyyyMM"));
    int dayOfMonth = now.getDayOfMonth();
    
    // 检查今天是否已签到
    return getBit(key, dayOfMonth - 1);
}
```

#### 3. 统计位数 - bitCount
```java
/**
 * 统计指定范围内1的个数
 * @param key Redis键
 * @param start 开始字节位置
 * @param end 结束字节位置
 * @return 1的个数
 */
public Long bitCount(String key, long start, long end) {
    return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
        return connection.bitCount(key.getBytes(), start, end);
    });
}

// 重载方法：统计整个key的1的个数
public Long bitCount(String key) {
    return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
        return connection.bitCount(key.getBytes());
    });
}

// 使用示例
public int getUserMonthSignCount(Long userId) {
    LocalDateTime now = LocalDateTime.now();
    String key = "user:sign:" + userId + ":" + now.format(DateTimeFormatter.ofPattern("yyyyMM"));
    
    // 统计本月总签到天数
    Long count = bitCount(key);
    return count != null ? count.intValue() : 0;
}
```

### 高级BitMap操作

#### 4. 批量位操作 - bitField
```java
/**
 * 批量位操作
 * @param key Redis键
 * @param subCommands 子命令
 * @return 操作结果列表
 */
public List<Long> bitField(String key, BitFieldSubCommands subCommands) {
    return stringRedisTemplate.opsForValue().bitField(key, subCommands);
}

// 使用示例：获取本月到今天的所有签到记录
public List<Long> getMonthSignData(Long userId) {
    LocalDateTime now = LocalDateTime.now();
    String key = "user:sign:" + userId + ":" + now.format(DateTimeFormatter.ofPattern("yyyyMM"));
    int dayOfMonth = now.getDayOfMonth();
    
    return bitField(key, 
        BitFieldSubCommands.create()
            .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
            .valueAt(0)
    );
}

// 使用示例：批量设置多个位
public void batchSetBits(String key, Map<Integer, Boolean> bitMap) {
    BitFieldSubCommands.BitFieldSubCommandsBuilder builder = BitFieldSubCommands.create();
    
    bitMap.forEach((offset, value) -> {
        builder.set(BitFieldSubCommands.BitFieldType.unsigned(1))
               .valueAt(offset)
               .to(value ? 1 : 0);
    });
    
    bitField(key, builder);
}
```

#### 5. 位运算操作 - bitOp
```java
/**
 * 位运算操作
 * @param op 运算类型
 * @param destination 目标key
 * @param keys 源key列表
 * @return 结果长度
 */
public Long bitOp(RedisStringCommands.BitOperation op, String destination, String... keys) {
    return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
        byte[][] keyBytes = new byte[keys.length][];
        for (int i = 0; i < keys.length; i++) {
            keyBytes[i] = keys[i].getBytes();
        }
        return connection.bitOp(op, destination.getBytes(), keyBytes);
    });
}

// 使用示例：计算两个用户都签到的天数
public int getCommonSignDays(Long userId1, Long userId2, String month) {
    String key1 = "user:sign:" + userId1 + ":" + month;
    String key2 = "user:sign:" + userId2 + ":" + month;
    String resultKey = "temp:common:" + userId1 + ":" + userId2 + ":" + month;
    
    // 执行AND运算
    bitOp(RedisStringCommands.BitOperation.AND, resultKey, key1, key2);
    
    // 统计结果中1的个数
    Long count = bitCount(resultKey);
    
    // 删除临时key
    stringRedisTemplate.delete(resultKey);
    
    return count != null ? count.intValue() : 0;
}
```

#### 6. 查找位位置 - bitPos
```java
/**
 * 查找指定位值的位置
 * @param key Redis键
 * @param bit 要查找的位值(true/false)
 * @param start 开始位置
 * @param end 结束位置
 * @return 位置索引，-1表示未找到
 */
public Long bitPos(String key, boolean bit, long start, long end) {
    return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
        return connection.bitPos(key.getBytes(), bit, start, end);
    });
}

// 重载方法：从头开始查找
public Long bitPos(String key, boolean bit) {
    return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
        return connection.bitPos(key.getBytes(), bit);
    });
}

// 使用示例：查找用户第一次签到的日期
public int getUserFirstSignDay(Long userId, String month) {
    String key = "user:sign:" + userId + ":" + month;
    
    // 查找第一个1的位置
    Long position = bitPos(key, true);
    
    if (position != null && position >= 0) {
        return (int) (position + 1); // 转换为日期(1-31)
    }
    return -1; // 未找到
}
```

### 实用工具类

#### BitMap工具类封装
```java
@Component
public class BitMapUtil {
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    /**
     * 用户签到
     */
    public boolean userSignIn(Long userId, LocalDateTime signTime) {
        String key = buildSignKey(userId, signTime);
        int dayOfMonth = signTime.getDayOfMonth();
        
        Boolean result = stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
        
        // 设置过期时间
        stringRedisTemplate.expire(key, Duration.ofDays(32));
        
        return result != null ? result : false;
    }
    
    /**
     * 检查用户是否签到
     */
    public boolean isUserSigned(Long userId, LocalDateTime checkTime) {
        String key = buildSignKey(userId, checkTime);
        int dayOfMonth = checkTime.getDayOfMonth();
        
        Boolean result = stringRedisTemplate.opsForValue().getBit(key, dayOfMonth - 1);
        return result != null ? result : false;
    }
    
    /**
     * 获取用户月签到次数
     */
    public int getUserMonthSignCount(Long userId, LocalDateTime month) {
        String key = buildSignKey(userId, month);
        
        Long count = stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
            return connection.bitCount(key.getBytes());
        });
        
        return count != null ? count.intValue() : 0;
    }
    
    /**
     * 获取用户连续签到天数
     */
    public int getUserContinuousSignCount(Long userId, LocalDateTime endTime) {
        String key = buildSignKey(userId, endTime);
        int dayOfMonth = endTime.getDayOfMonth();
        
        // 获取本月到今天的签到数据
        List<Long> result = stringRedisTemplate.opsForValue().bitField(
            key,
            BitFieldSubCommands.create()
                .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
                .valueAt(0)
        );
        
        if (result == null || result.isEmpty()) {
            return 0;
        }
        
        Long num = result.get(0);
        if (num == null || num == 0) {
            return 0;
        }
        
        // 从后往前统计连续签到天数
        int count = 0;
        while (num > 0) {
            if ((num & 1) == 0) {
                break;
            }
            count++;
            num >>>= 1;
        }
        
        return count;
    }
    
    /**
     * 获取用户指定日期范围的签到记录
     */
    public List<Integer> getUserSignRecord(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
        String key = buildSignKey(userId, startTime);
        List<Integer> signDays = new ArrayList<>();
        
        int startDay = startTime.getDayOfMonth();
        int endDay = endTime.getDayOfMonth();
        
        for (int day = startDay; day <= endDay; day++) {
            Boolean isSigned = stringRedisTemplate.opsForValue().getBit(key, day - 1);
            if (isSigned != null && isSigned) {
                signDays.add(day);
            }
        }
        
        return signDays;
    }
    
    /**
     * 批量设置用户签到记录
     */
    public void batchSetUserSign(Long userId, LocalDateTime month, List<Integer> signDays) {
        String key = buildSignKey(userId, month);
        
        BitFieldSubCommands.BitFieldSubCommandsBuilder builder = BitFieldSubCommands.create();
        
        for (Integer day : signDays) {
            builder.set(BitFieldSubCommands.BitFieldType.unsigned(1))
                   .valueAt(day - 1)
                   .to(1);
        }
        
        stringRedisTemplate.opsForValue().bitField(key, builder);
        stringRedisTemplate.expire(key, Duration.ofDays(32));
    }
    
    /**
     * 构建签到key
     */
    private String buildSignKey(Long userId, LocalDateTime time) {
        return "user:sign:" + userId + ":" + time.format(DateTimeFormatter.ofPattern("yyyyMM"));
    }
}
```

### 性能优化建议

#### 1. 批量操作优化
```java
/**
 * 批量获取多个用户的签到状态
 */
public Map<Long, Boolean> batchGetUserSignStatus(List<Long> userIds, LocalDateTime checkTime) {
    Map<Long, Boolean> result = new HashMap<>();
    
    // 使用Pipeline批量执行
    List<Object> results = stringRedisTemplate.executePipelined(new RedisCallback<Object>() {
        @Override
        public Object doInRedis(RedisConnection connection) throws DataAccessException {
            int dayOfMonth = checkTime.getDayOfMonth();
            
            for (Long userId : userIds) {
                String key = buildSignKey(userId, checkTime);
                connection.getBit(key.getBytes(), dayOfMonth - 1);
            }
            return null;
        }
    });
    
    // 处理结果
    for (int i = 0; i < userIds.size(); i++) {
        result.put(userIds.get(i), (Boolean) results.get(i));
    }
    
    return result;
}
```

#### 2. 缓存优化
```java
/**
 * 带缓存的签到状态检查
 */
@Cacheable(value = "userSign", key = "#userId + ':' + #checkTime.format(T(java.time.format.DateTimeFormatter).ofPattern('yyyyMMdd'))")
public boolean isUserSignedWithCache(Long userId, LocalDateTime checkTime) {
    return isUserSigned(userId, checkTime);
}
```

## 签到功能实现

我们可以把年和月作为bitMap的key，然后保存到一个bitMap中，每次签到就到对应的位上把数字从0变成1，只要对应是1，就表明说明这一天已经签到了，反之则没有签到。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250617192409193.png)

### 签到接口实现

```java
/**
 * 用户签到
 */
@Override
public Result sign() {
    // 1.获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    // 2.获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3.拼接key
    // 3.1 获取月份 ,202506
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 3.2获取本月日期
    int dayOfMonth = now.getDayOfMonth();
    // 4.写入Redis SETBIT key offset 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
    return Result.ok();
}
```

**关键点解析：**
- **key设计**：`user:sign:用户ID:年月` 格式，如 `user:sign:1001:202506`
- **offset计算**：`dayOfMonth - 1`，因为数组索引从0开始
  - 1号 → 索引0，2号 → 索引1，以此类推
- **BitMap映射**：一个月最多31天，用31个bit位表示

## 签到统计功能

### 连续签到天数的定义
**什么叫做连续签到天数？**
从最后一次签到开始向前统计，直到遇到第一次未签到为止，计算总的签到次数，就是连续签到天数。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250617193409231.png)

### 实现思路分析

**问题1：如何得到本月到今天为止的所有签到数据？**

使用BITFIELD命令：
```redis
BITFIELD key GET u[dayOfMonth] 0
```
假设今天是15号，那么我们就可以从当前月的第一天开始，获得到当前这一天的位数，去拿这15天的数据。

**问题2：如何从后向前遍历每个bit位？**

BitMap返回的数据是10进制数字，比如返回数字`21845`，我们需要知道哪些位是0，哪些位是1。

**核心算法：位运算**
- 让得到的10进制数字和1做**与运算**
- 因为1只有遇见1才是1，其他数字都是0
- 每与一次，就把签到结果向右移动一位
- 依次内推，完成逐个遍历的效果

### 位运算详解

#### 什么是位运算？

位运算是直接对整数在内存中的二进制位进行操作。

#### 核心位运算符

1. **与运算 (&)**
   ```
   1 & 1 = 1
   1 & 0 = 0  
   0 & 1 = 0
   0 & 0 = 0
   ```

2. **右移运算 (>>>)**
   - `>>>` 无符号右移，左边补0
   - `>>` 有符号右移，左边补符号位

#### 位运算在签到统计中的应用

**示例：假设今天是15号，签到数据的二进制为：**
```
原始数据: 101011111111111 (从右到左：1号到15号)
十进制值: 21845
```

**逐步位运算过程：**

```java
// 第1次循环 - 检查15号（最右边位）
num = 101011111111111 (21845)
num & 1 = 1 → 已签到，count = 1
num >>>= 1 → num = 10101111111111 (10922)

// 第2次循环 - 检查14号  
num = 10101111111111 (10922)
num & 1 = 1 → 已签到，count = 2
num >>>= 1 → num = 1010111111111 (5461)

// ...继续直到遇到0

// 第12次循环 - 检查4号
num = 1010 (10)
num & 1 = 0 → 未签到，break！
```

**关键理解：**
- `num & 1` 获取最后一位：因为1的二进制是`...00000001`
- 与运算后，只有最后一位被保留，其他位都变成0
- `num >>>= 1` 右移一位：相当于去掉已检查的位

### 完整实现代码

```java
/**
 * 获取连续签到天数
 */
@Override
public Result signCount() {
    // 1.获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    // 2.获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3.拼接key
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4.获取今天是第几天
    int dayOfMonth = now.getDayOfMonth();

    // 5.获取本月截至到今天的所有签到记录，返回的是一个十进制的数字 BITFIELD sign:5:202203 GET u14 0
    List<Long> result = stringRedisTemplate.opsForValue().bitField(
            key,
            BitFieldSubCommands
                    .create()
                    .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth))
                    .valueAt(0));
    
    if (result == null || result.isEmpty()) {
        // 没有任何签到结果
        return Result.ok(0);
    }
    
    Long num = result.get(0);
    if (num == null || num == 0) {
        // 没有签到
        return Result.ok(0);
    }
    
    // 6.循环遍历
    int count = 0;
    while (true) {
        // 6.1.让这个数字与1做与运算，得到数字的最后一个bit位
        // 判断这个bit位是否为0
        if ((num & 1) == 0) {
            // 如果为0，说明未签到，结束
            break;
        } else {
            // 如果不为0，说明已签到，计数器+1
            count++;
        }
        // 6.2.把数字右移一位，抛弃已检查的位，继续下一个bit位
        num >>>= 1;
    }
    return Result.ok(count);
}
```

**代码关键点：**
1. **BITFIELD命令**：一次性获取多个bit位，比逐个GETBIT高效
2. **位运算统计**：通过`&`和`>>>`运算，高效遍历每一位
3. **连续性检查**：遇到0就停止，确保统计的是连续天数
4. **性能优化**：时间复杂度为O(连续签到天数)，而不是O(总天数)

## BitMap的其他应用场景

### 1. 用户在线状态统计
```java
// 记录用户今天是否在线
SETBIT online:20250617 userId 1

// 统计今天在线用户数
BITCOUNT online:20250617

// 获取指定用户是否在线
GETBIT online:20250617 userId
```

### 2. 活跃用户统计
```java
// 用户访问记录
SETBIT active:202506 userId 1

// 统计6月活跃用户数
BITCOUNT active:202506

// 计算两个月都活跃的用户
BITOP AND result active:202505 active:202506
BITCOUNT result
```

### 3. 权限管理
```java
// 用户权限BitMap (假设有8种权限)
// 权限1：读取 权限2：写入 权限3：删除 ...
SETBIT user:1001:permission 0 1  // 有读取权限
SETBIT user:1001:permission 1 1  // 有写入权限
SETBIT user:1001:permission 2 0  // 无删除权限

// 检查用户是否有某权限
GETBIT user:1001:permission 0
```

## 位运算进阶技巧

### 常用位运算操作

#### 1. 判断奇偶性
```java
// 判断数字是否为奇数
boolean isOdd = (num & 1) == 1;

// 判断数字是否为偶数  
boolean isEven = (num & 1) == 0;
```

#### 2. 获取指定位的值
```java
// 获取第n位的值 (从右往左，从0开始)
int getBit(int num, int n) {
    return (num >> n) & 1;
}
```

#### 3. 设置指定位为1
```java
// 将第n位设置为1
int setBit(int num, int n) {
    return num | (1 << n);
}
```

#### 4. 清除指定位(设为0)
```java
// 将第n位设置为0
int clearBit(int num, int n) {
    return num & ~(1 << n);
}
```

#### 5. 翻转指定位
```java
// 翻转第n位
int toggleBit(int num, int n) {
    return num ^ (1 << n);
}
```

### 位运算性能优势

**时间复杂度对比：**
- 传统方案（数据库查询）：O(n)，n为需要查询的天数
- BitMap + 位运算：O(k)，k为连续签到天数

**空间复杂度对比：**
- 传统方案：O(n * 记录大小)
- BitMap：O(1)，固定的bit位数量

## 实际应用注意事项

### 1. 数据一致性
```java
// 使用Redis事务确保操作原子性
Multi multi = jedis.multi();
multi.setbit(key, offset, true);
multi.expire(key, 86400 * 32); // 设置过期时间
multi.exec();
```

### 2. 内存使用优化
```java
// 为BitMap设置合理的过期时间
stringRedisTemplate.expire(key, Duration.ofDays(32));

// 定期清理历史数据
@Scheduled(cron = "0 0 2 1 * ?") // 每月1号凌晨2点执行
public void cleanOldSignData() {
    // 清理3个月前的签到数据
}
```

### 3. 错误处理
```java
try {
    List<Long> result = stringRedisTemplate.opsForValue().bitField(/*...*/);
    // 处理业务逻辑
} catch (Exception e) {
    log.error("BitMap操作失败", e);
    // 降级处理或返回默认值
    return Result.ok(0);
}
```

## 总结

BitMap是Redis提供的一种高效的位存储数据结构，特别适合处理大量的布尔类型数据：

### 优势
1. **空间效率高**：相比传统存储方案节省99%的存储空间
2. **操作效率高**：位运算速度极快，时间复杂度低
3. **功能丰富**：支持各种位运算操作
4. **应用广泛**：签到、统计、权限等多种场景

### 适用场景
- ✅ 大量用户的状态记录（签到、在线状态等）
- ✅ 布尔类型的数据统计
- ✅ 需要高性能位运算的场合
- ❌ 数据稀疏的场景（大部分位为0）
- ❌ 需要存储复杂数据类型的场合

### 核心技巧
1. **合理设计key**：通常包含时间维度便于管理
2. **掌握位运算**：`&` `|` `^` `<<` `>>` 等基础操作
3. **注意索引映射**：数组索引从0开始的特性
4. **设置过期时间**：避免内存无限增长
5. **异常处理**：保证系统稳定性

通过BitMap和位运算的结合，我们可以用极小的存储空间和极高的运算效率，解决大规模的状态统计问题。这正是Redis BitMap的魅力所在！

