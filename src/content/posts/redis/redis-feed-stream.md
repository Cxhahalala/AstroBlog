---
title: "Feed流"
published: 2025-06-18
description: "Feed 流架构设计与实现：深入讲解推模式（Push）、拉模式（Pull）以及推拉结合模式在大规模社交网络中的应用实践。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817163106371.png"
tags: ["redis","java"]
category: "Redis"
author: "Cx"
draft: false
---

Feed流是一种为用户持续提供内容的推送机制,本文详细介绍Feed流的两种实现模式：拉模式、推模式和推拉结合模式 
<!-- more -->

# Feed流
当我们关注了用户后，这个用户发了动态，那么我们应该把这些数据推送给用户，这个需求，其实我们又把他叫做Feed流，关注推送也叫做Feed流，直译为投喂。为用户持续的提供“沉浸式”的体验，通过无限下拉刷新获取新的信息。

对于传统的模式的内容解锁：我们是需要用户去通过搜索引擎或者是其他的方式去解锁想要看的内容
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616211740292.png)

对于新型的Feed流的的效果：不需要我们用户再去推送信息，而是系统分析用户到底想要什么，然后直接把内容推送给用户，从而使用户能够更加的节约时间，不用主动去寻找。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616211753747.png)

Feed流的实现有两种模式：

Feed流产品有两种常见模式：
Timeline：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈

* 优点：信息全面，不会有缺失。并且实现也相对简单
* 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低

智能排序：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户

* 优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷
* 缺点：如果算法不精准，可能起到反作用
  本例中的个人页面，是基于关注的好友来做Feed流，因此采用Timeline的模式。该模式的实现方案有三种：

我们本次针对好友的操作，采用的就是Timeline的方式，只需要拿到我们关注用户的信息，然后按照时间排序即可

，因此采用Timeline的模式。该模式的实现方案有三种：

* 拉模式
* 推模式
* 推拉结合

**拉模式**：也叫做读扩散

该模式的核心含义就是：当张三和李四和王五发了消息后，都会保存在自己的邮箱中，假设赵六要读取信息，那么他会从读取他自己的收件箱，此时系统会从他关注的人群中，把他关注人的信息全部都进行拉取，然后在进行排序

优点：比较节约空间，因为赵六在读信息时，并没有重复读取，而且读取完之后可以把他的收件箱进行清楚。

缺点：比较延迟，当用户读取数据时才去关注的人里边去读取数据，假设用户关注了大量的用户，那么此时就会拉取海量的内容，对服务器压力巨大。

该模式的核心含义就是：当张三和李四和王五发了消息后，都会保存在自己的邮箱中，假设赵六要读取信息，那么他会从读取他自己的收件箱，此时系统会从他关注的人群中，把他关注人的信息全部都进行拉取，然后在进行排序

优点：比较节约空间，因为赵六在读信息时，并没有重复读取，而且读取完之后可以把他的收件箱进行清楚。

缺点：比较延迟，当用户读取数据时才去关注的人里边去读取数据，假设用户关注了大量的用户，那么此时就会拉取海量的内容，对服务器压力巨大。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616211931280.png)

**推模式**：也叫做写扩散。

推模式是没有写邮箱的，当张三写了一个内容，此时会主动的把张三写的内容发送到他的粉丝收件箱中去，假设此时李四再来读取，就不用再去临时拉取了

优点：时效快，不用临时拉取

缺点：内存压力大，假设一个大V写信息，很多人关注他， 就会写很多分数据到粉丝那边去

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616212023253.png)

**推拉结合模式**：也叫做读写混合，兼具推和拉两种模式的优点。

推拉模式是一个折中的方案，站在发件人这一段，如果是个普通的人，那么我们采用写扩散的方式，直接把数据写入到他的粉丝中去，因为普通的人他的粉丝关注量比较小，所以这样做没有压力，如果是大V，那么他是直接将数据先写入到一份到发件箱里边去，然后再直接写一份到活跃粉丝收件箱里边去，现在站在收件人这端来看，如果是活跃粉丝，那么大V和普通的人发的都会直接写入到自己收件箱里边来，而如果是普通的粉丝，由于他们上线不是很频繁，所以等他们上线时，再从发件箱里边去拉信息。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616212044197.png)

# 推送
## Feed流的分页问题

Feed流中的数据会不断更新，所以数据的角标也在变化，因此不能采用传统的分页模式。

传统了分页在feed流是不适用的，因为我们的数据会随时发生变化

假设在t1 时刻，我们去读取第一页，此时page = 1 ，size = 5 ，那么我们拿到的就是10~6 这几条记录，假设现在t2时候又发布了一条记录，此时t3 时刻，我们来读取第二页，读取第二页传入的参数是page=2 ，size=5 ，那么此时读取到的第二页实际上是从6 开始，然后是6~2 ，那么我们就读取到了重复的数据，所以feed流的分页，不能采用原始方案来做。
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616212307052.png)

## Feed流的滚动分页

我们需要记录每次操作的最后一条，然后从这个位置开始去读取数据

举个例子：我们从t1时刻开始，拿第一页数据，拿到了10~6，然后记录下当前最后一次拿取的记录，就是6，t2时刻发布了新的记录，此时这个11放到最顶上，但是不会影响我们之前记录的6，此时t3时刻来拿第二页，第二页这个时候拿数据，还是从6后一点的5去拿，就拿到了5-1的记录。我们这个地方可以采用sortedSet来做，可以进行范围查询，并且还可以记录当前获取数据时间戳最小值，就可以实现滚动分页了

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250616212506826.png)

体操作如下：

1、每次查询完成后，我们要分析出查询出数据的最小时间戳，这个值会作为下一次查询的条件

2、我们需要找到与上一次查询相同的查询个数作为偏移量，下次查询时，跳过这些查询过的数据，拿到我们需要的数据

综上：我们的请求参数中就需要携带 lastId：上一次查询的最小时间戳 和偏移量这两个参数。

## Redis ZSet实现滚动分页

### 为什么选择reverseRangeByScoreWithScores方法

在实现Feed流的滚动分页时，我们使用Redis的ZSet数据结构，其中：
- **Score（分数）**：存储时间戳，分数越大表示动态越新
- **Value（值）**：存储动态ID或内容

`reverseRangeByScoreWithScores`方法的核心优势：

1. **Reverse（反向排序）**：按分数从高到低排序，确保最新动态在前
2. **ByScore（按分数范围）**：可以指定分数范围，实现精确的时间窗口查询
3. **WithScores（包含分数）**：同时返回元素值和分数，便于记录最后查询的时间戳
4. **支持偏移量和限制**：实现分页功能

### 数据结构示例

```java
// Redis ZSet数据：key = "user:123:feed"
/*
动态ID    分数(时间戳)    说明
post:005   1686900000    ← 最新（分数最高）
post:004   1686800000    
post:003   1686700000    
post:002   1686600000    
post:001   1686500000    ← 最旧（分数最低）
*/
```

### 滚动分页的问题与解决

#### 传统偏移量方式的问题

```java
// 第一次请求：获取最新2条
Set<ZSetOperations.TypedTuple<String>> page1 = stringRedisTemplate.opsForZSet()
    .reverseRangeByScoreWithScores("user:123:feed", 0, Double.MAX_VALUE, 0, 2);
// 返回：post:005, post:004

// 此时有新动态插入！
/*
post:007   1687100000    ← 最新插入
post:006   1687000000    ← 新插入  
post:005   1686900000    ← 原来的最新
post:004   1686800000    
post:003   1686700000    
post:002   1686600000    
post:001   1686500000    
*/

// 第二次请求：使用偏移量会导致重复
Set<ZSetOperations.TypedTuple<String>> page2 = stringRedisTemplate.opsForZSet()
    .reverseRangeByScoreWithScores("user:123:feed", 0, Double.MAX_VALUE, 2, 2);
// 偏移量=2，跳过前2条（现在是post:007, post:006）
// 返回：post:005, post:004 （重复了！）
```

#### 解决方案：记录lastScore + 偏移量

```java
public class FeedService {
    
    public ScrollResult getUserFeed(Long userId, Double lastScore, Long offset, Integer count) {
        String key = "user:" + userId + ":feed";
        
        Set<ZSetOperations.TypedTuple<String>> typedTuples;
        
        if (lastScore == null) {
            // 首次加载：获取最新数据
            typedTuples = stringRedisTemplate.opsForZSet()
                .reverseRangeByScoreWithScores(key, 0, Double.MAX_VALUE, offset, count);
        } else {
            // 加载更多：获取比lastScore更小的数据（更老的数据）
            typedTuples = stringRedisTemplate.opsForZSet()
                .reverseRangeByScoreWithScores(key, 0, lastScore - 1, offset, count);
        }
        
        List<FeedDTO> feeds = new ArrayList<>();
        Double minScore = null;
        
        for (ZSetOperations.TypedTuple<String> tuple : typedTuples) {
            feeds.add(buildFeedDTO(tuple.getValue(), tuple.getScore()));
            minScore = tuple.getScore(); // 记录本次查询的最小分数
        }
        
        return new ScrollResult(feeds, minScore, 0L);
    }
}
```

### 完整的分页流程示例

```java
// 第一次请求：首屏加载
ScrollResult result1 = feedService.getUserFeed(123L, null, 0L, 2);
/*
查询：reverseRangeByScoreWithScores(key, 0, Double.MAX_VALUE, 0, 2)
返回：post:005(1686900000), post:004(1686800000)
lastScore = 1686800000 （记录本次查询的最小分数）
*/

// 新数据插入后...

// 第二次请求：加载历史动态
ScrollResult result2 = feedService.getUserFeed(123L, 1686800000.0, 0L, 2);
/*
查询：reverseRangeByScoreWithScores(key, 0, 1686799999, 0, 2)
查询条件：分数 < 1686800000（排除已看过的post:004）
返回：post:003(1686700000), post:002(1686600000)
lastScore = 1686600000
*/

// 第三次请求：继续加载
ScrollResult result3 = feedService.getUserFeed(123L, 1686600000.0, 0L, 2);
/*
查询：reverseRangeByScoreWithScores(key, 0, 1686599999, 0, 2)  
返回：post:001(1686500000)
lastScore = 1686500000
*/
```

### 方法参数详解

```java
Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
    .reverseRangeByScoreWithScores(key, 0, max, offset, count);
```

- **key**：Redis键名，存储用户的feed流数据
- **0**：最小分数（包含），通常设为0表示不限制下限
- **max**：最大分数（包含），首次查询用Double.MAX_VALUE，后续用lastScore-1
- **offset**：偏移量，跳过前面多少个符合条件的元素
- **count**：返回数量限制，控制每页返回的元素个数

### 优势总结

1. **避免数据重复**：通过分数过滤确保不会因新数据插入而重复返回
2. **避免数据遗漏**：保证时间线的连续性，用户不会错过任何动态
3. **高性能查询**：利用ZSet的有序特性，查询效率O(log(N)+M)
4. **支持大数据量**：适合处理海量feed数据的分页场景
5. **实现简单**：相比复杂的分页算法，实现更加直观和可靠

### 前端调用示例

```javascript
// 前端维护分页状态
let lastScore = null;

// 首次加载
async function loadFirstPage() {
    const result = await api.getFeed(userId, null, 0, 10);
    lastScore = result.lastScore;
    renderFeeds(result.feeds);
}

// 加载更多（下拉刷新）
async function loadMore() {
    const result = await api.getFeed(userId, lastScore, 0, 10);
    lastScore = result.lastScore;
    appendFeeds(result.feeds);
}
```

这种方案在微博、朋友圈、抖音等主流社交平台的feed流实现中被广泛采用，能够很好地解决动态数据环境下的分页问题。

## 实际代码实现详解

### 核心分页查询方法

```java
@Override
public Result queryFeedOfFollow(Long maxScore, Integer offset) {
    // 1.获取当前用户ID
    Long userId = UserHolder.getUser().getId();
    
    // 2.构建Redis key，通常格式为：feed:userId
    String key = FEED_KEY + userId;
    
    // 3.执行滚动分页查询
    // ZREVRANGEBYSCORE key maxScore minScore LIMIT offset count
    Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
        .reverseRangeByScoreWithScores(key, 0, maxScore, offset, 2);
    
    // 4.非空判断
    if (typedTuples == null || typedTuples.isEmpty()) {
        return Result.ok();
    }
    
    // 5.解析数据并计算下次查询参数
    List<Long> ids = new ArrayList<>(typedTuples.size());
    long minTime = 0;        // 本次查询结果中的最小时间戳
    int offsetNext = 1;      // 下次查询的偏移量初始值
    
    // 6.遍历查询结果，提取ID和计算offset
    for (ZSetOperations.TypedTuple<String> tuple : typedTuples) {
        // 6.1.提取数据ID
        ids.add(Long.valueOf(tuple.getValue()));
        
        // 6.2.获取时间戳分数
        long currentTime = tuple.getScore().longValue();
        
        // 6.3.计算下次查询的offset（核心算法）
        if (currentTime == minTime) {
            // 相同时间戳：计数累加
            offsetNext++;
        } else {
            // 不同时间戳：更新最小时间戳，重置计数
            minTime = currentTime;
            offsetNext = 1;
        }
    }
    
    // 7.最终offset计算（关键逻辑）
    // 如果本次最小时间戳等于查询的最大时间戳，说明还在相同时间戳范围内
    // 需要累加之前的offset；否则进入新时间戳范围，使用当前计算的offset
    offsetNext = minTime == maxScore ? offsetNext + offset : offsetNext;
    
    // 8.根据ID查询具体业务数据
    String idStr = StrUtil.join(",", ids);
    List<FeedItem> feedItems = query().in("id", ids)
        .last("ORDER BY FIELD(id," + idStr + ")").list();
    
    // 9.业务数据处理（如查询用户信息、点赞状态等）
    for (FeedItem item : feedItems) {
        // 处理业务相关数据
        processFeedItem(item);
    }
    
    // 10.封装分页结果
    ScrollResult result = new ScrollResult();
    result.setList(feedItems);
    result.setOffset(offsetNext);      // 下次查询的偏移量
    result.setMinTime(minTime);        // 下次查询的最大分数
    
    return Result.ok(result);
}
```

### 算法核心逻辑详解

#### offset计算算法

```java
/**
 * offset核心计算逻辑
 * 目标：计算下次查询时需要跳过多少条记录
 */
long minTime = 0;        // 本次查询的最小时间戳
int offsetNext = 1;      // 下次查询的偏移量

for (ZSetOperations.TypedTuple<String> tuple : typedTuples) {
    long currentTime = tuple.getScore().longValue();
    
    if (currentTime == minTime) {
        // 情况1：相同时间戳
        // 在同一个时间戳下，记录出现的次数
        offsetNext++;
    } else {
        // 情况2：不同时间戳  
        // 更新为当前时间戳，重置计数为1（当前这条是该时间戳的第1条）
        minTime = currentTime;
        offsetNext = 1;
    }
}

// 最终offset计算公式
offsetNext = minTime == maxScore ? offsetNext + offset : offsetNext;
```

#### 为什么需要这样计算？

**核心问题：相同时间戳的数据分页**

```java
// 假设Redis中的数据（时间戳相同的情况）
/*
feed:005   1687000000    ← 相同时间戳
feed:004   1687000000    ← 相同时间戳  
feed:003   1687000000    ← 相同时间戳
feed:002   1686900000    ← 不同时间戳
feed:001   1686800000    
*/

// 分页查询过程演示：

// 第1次查询：queryFeedOfFollow(Long.MAX_VALUE, 0)
// 查询条件：score <= MAX_VALUE, offset=0, count=2
// 返回结果：feed:005(1687000000), feed:004(1687000000)
// 
// 算法计算过程：
// 1. feed:005: currentTime=1687000000, minTime=0 
//    -> 不同时间戳，minTime=1687000000, offsetNext=1
// 2. feed:004: currentTime=1687000000, minTime=1687000000
//    -> 相同时间戳，offsetNext=2
// 
// 最终：minTime=1687000000, offsetNext=2
// 因为 minTime != maxScore(MAX_VALUE)，所以：offsetNext = 2
// 
// 返回给前端：{minTime: 1687000000, offset: 2}

// 第2次查询：queryFeedOfFollow(1687000000, 2)
// 查询条件：score <= 1687000000, offset=2, count=2
// Redis执行：在时间戳<=1687000000的数据中，跳过前2条，取接下来2条
// 返回结果：feed:003(1687000000), feed:002(1686900000)
//
// 算法计算过程：
// 1. feed:003: currentTime=1687000000, minTime=0
//    -> 不同时间戳，minTime=1687000000, offsetNext=1  
// 2. feed:002: currentTime=1686900000, minTime=1687000000
//    -> 不同时间戳，minTime=1686900000, offsetNext=1
//
// 最终：minTime=1686900000, offsetNext=1
// 因为 minTime != maxScore(1687000000)，所以：offsetNext = 1
//
// 返回给前端：{minTime: 1686900000, offset: 1}
```

### 特殊场景处理

#### 场景1：连续相同时间戳

```java
// 数据场景：大量相同时间戳
/*
feed:008   1687000000    ← index 0
feed:007   1687000000    ← index 1
feed:006   1687000000    ← index 2  
feed:005   1687000000    ← index 3
feed:004   1686900000    ← index 4
*/

// 查询序列：
// 1次：maxScore=MAX, offset=0 -> 返回index[0,1], 下次offset=2
// 2次：maxScore=1687000000, offset=2 -> 返回index[2,3], 下次offset=4  
// 3次：maxScore=1687000000, offset=4 -> 返回index[4], 进入新时间戳
```

#### 场景2：修正错误的offset计算

```java
// 原始代码（错误）：
offsetNext = minTime == maxScore ? offsetNext : offsetNext + offset;

// 修正代码（正确）：  
offsetNext = minTime == maxScore ? offsetNext + offset : offsetNext;

// 修正原因：
// - 相同时间戳(minTime == maxScore)：需要累加offset，继续在同一时间戳范围内分页
// - 不同时间戳(minTime != maxScore)：进入新时间戳，offset重新开始计算
```

### 前端调用示例

```javascript
class FeedPagination {
    constructor() {
        this.maxScore = null;      // 上次查询的最小时间戳
        this.offset = 0;           // 当前偏移量
        this.loading = false;      // 加载状态
        this.hasMore = true;       // 是否还有更多数据
    }
    
    // 首次加载feed流
    async loadFirst() {
        this.loading = true;
        try {
            const response = await api.queryFeedOfFollow(null, 0);
            const result = response.data;
            
            // 更新分页参数
            this.maxScore = result.minTime;
            this.offset = result.offset;
            this.hasMore = result.list && result.list.length > 0;
            
            return result.list;
        } finally {
            this.loading = false;
        }
    }
    
    // 加载更多数据
    async loadMore() {
        if (this.loading || !this.hasMore) return [];
        
        this.loading = true;
        try {
            const response = await api.queryFeedOfFollow(this.maxScore, this.offset);
            const result = response.data;
            
            // 更新分页参数
            this.maxScore = result.minTime;
            this.offset = result.offset;
            this.hasMore = result.list && result.list.length > 0;
            
            return result.list;
        } finally {
            this.loading = false;
        }
    }
}

// 使用示例
const feedPagination = new FeedPagination();

// 首次加载
const firstPage = await feedPagination.loadFirst();
console.log('首页数据:', firstPage);

// 滚动加载更多
const secondPage = await feedPagination.loadMore();  
console.log('第二页数据:', secondPage);
```

### 算法优势总结

1. **处理相同时间戳**：通过offset累加机制，确保相同时间戳的数据不重复、不遗漏
2. **跨时间戳连续性**：不同时间戳之间平滑过渡，保持feed流的时间连续性
3. **高性能查询**：利用Redis ZSet的有序特性，查询复杂度为O(log(N)+M)
4. **内存友好**：支持真正的滚动分页，避免一次性加载大量数据
5. **实时更新兼容**：即使在分页过程中有新数据插入，也不会影响当前分页的连续性

这种算法已经在各大社交平台的feed流系统中得到了广泛验证和应用。