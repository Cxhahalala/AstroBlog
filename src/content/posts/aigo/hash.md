---
title: "算法中的哈希思想：从 O(N²) 到 O(N)"
published: 2025-07-13
description: "算法中的哈希思想：从 O(N²) 暴力优化至 O(N) 高效映射，深入浅出解析哈希表在经典算法中的核心思想与解题模板。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817162903558.png"
tags: ["哈希","算法"]
category: "算法笔记"
draft: false
---

## 什么是哈希思想？

在算法中，“哈希”（Hash）或称“散列”，核心思想是**建立一种映射关系**。它能将一个较大或复杂范围的数据（比如一个字符串、一个对象）通过一个“哈希函数”转换成一个易于处理的、通常是固定长度的“指纹”或“地址”（哈希值），然后我们可以利用这个“指纹”来快速地存取数据。

最经典的应用就是**哈希表**（`HashMap`），它提供**键-值（Key-Value）**对的存储方式。哈希表的魔力在于，理想情况下，它能让我们以 **O(1)** 的平均时间复杂度完成插入、删除和查找操作，速度非常快。

**简单类比：**
把哈希表想象成一本字典。
- **Key**：你要查的单词（如 "apple"）。
- **Value**：这个单词的释义。
- **哈希函数**：就是字典的索引（比如按首字母 'a' 查找）。你不需要从头翻到尾，直接翻到 'a' 的部分就能快速定位。

在解题时，哈希思想的关键在于：**找到一个合适的“特征”作为 Key，将具有相同“特征”的数据聚合到一起处理。**

---

## 案例分析：LeetCode 49. 字母异位词分组

### 题目描述

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

> 字母异位词 是由相同字母按不同顺序排列后形成的单词。

**示例:**
**输入:** `strs = ["eat", "tea", "tan", "ate", "nat", "bat"]`
**输出:** `[["bat"],["nat","tan"],["ate","eat","tea"]]`

---

### 你的解法：双重循环 + 暴力比较

这是你最初的实现思路，它代表了一种直接、朴素的解法。

#### 代码实现

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        int length = strs.length;
        if(strs.length == 0){
            return new ArrayList<>();
        }
        List<List<String>> result = new ArrayList<>();
        // 1. 为每个字符串预先计算好字符频率 Map
        Map<Integer,Map<Character,Integer>> resultMap = new HashMap<>();
        int[] flag = new int[length]; // 标记是否已被分组
        for(int i =0 ;i<length;i++){
            String str = strs[i];
            Map<Character,Integer> map = new HashMap<>();
            for(int j=0;j<str.length();j++){
                char c = str.charAt(j);
                map.put(c, map.getOrDefault(c, 0) + 1);
            }
            resultMap.put(i,map);
        }

        // 2. 双重循环，比较每个字符串与其他所有字符串
        for(int i = 0;i<length;i++){
            if(flag[i] == 1){ // 如果已经被分过组，就跳过
                continue;
            }
            List<String> list = new ArrayList<>();
            list.add(strs[i]);
            flag[i]=1;
            
            Map<Character,Integer> nowMap = resultMap.get(i);
            for(int j = i + 1; j<length;j++){
                if(flag[j] == 1){
                    continue;
                }
                Map<Character,Integer> nextMap = resultMap.get(j);
                if(nowMap.equals(nextMap)){ // 比较两个词的频率 Map 是否相同
                    list.add(strs[j]);
                    flag[j]=1;
                }
            }
            result.add(list);
        }
        return result;
    }
}
```

#### 思路与评价

- **思考路径**：
    1. 如何判断两个词是字母异位词？—— 它们包含的字母和数量完全一样。
    2. 如何表示一个词的字母和数量？—— 用一个 `HashMap<Character, Integer>` 来统计频率。
    3. 如何分组？—— 拿第一个词，和后面所有词比较，把异位词都找出来；再拿下一个还没分组的词，重复这个过程。

- **优点**：
    - 逻辑直观，容易想到。
    - 准确抓住了“字母异位词”的核心——字符频率相同。

- **缺点（致命）**：
    - **效率太低**：使用了双重循环，时间复杂度高达 **O(N² * L)**，其中 N 是字符串数量，L 是字符串平均长度。当 N 很大时（题目限制到 10⁴），N² 会导致程序运行超时。
    - **代码略显复杂**：为了避免重复分组，额外使用了一个 `flag` 数组来标记状态，增加了代码的复杂性。

---

### 如何优化？—— 运用哈希思想

你的解法之所以慢，是因为进行了大量的“比较”。优化的核心在于**减少甚至消除“比较”**，转而采用**“分类”**的思想。

**思考的关键转变：**
> 不要去问 “A 和 B 是不是一类？”，而是去想 “A 属于哪一类？”

我们只需要为每一组字母异位词找到一个**独一无二、不会改变的“身份证”**（也就是 Key），然后把所有拥有相同“身份证”的单词都放到同一个组里。

这个“身份证”应该是什么？
1.  **思路一**：既然字母都一样，只是顺序不同，那我们把它们**排序**，顺序不就一样了吗？`"eat"`, `"tea"`, `"ate"` 排序后都是 `"aet"`。这个 `"aet"` 就是完美的 Key！
2.  **思路二**：既然字母数量都一样，那我们可以把这个**数量信息**变成一个字符串。比如，`"eat"` 可以表示为 `"a1e1t1"`（1个a, 1个e, 1个t）。这个字符串也可以作为 Key。

这两种思路都指向了官方的高效解法。

---

### 官方解法

#### 方法一：排序

将排序后的字符串作为哈希表的 Key。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }
        // Key: 排序后的字符串, Value: 原始字符串列表
        Map<String, List<String>> map = new HashMap<>();
        for (String str : strs) {
            // 将字符串转换为字符数组并排序
            char[] charArray = str.toCharArray();
            Arrays.sort(charArray);
            String key = new String(charArray);
            
            // 从 map 中获取列表，如果 key 不存在，则创建一个新列表
            List<String> list = map.getOrDefault(key, new ArrayList<>());
            
            // 将原始字符串添加到列表中
            list.add(str);
            
            // 将更新后的列表放回 map
            map.put(key, list);
        }
        // map 中所有的 value 集合就是最终结果
        return new ArrayList<>(map.values());
    }
}
```
- **时间复杂度**：`O(N * K log K)`，其中 N 是字符串数量，K 是字符串最大长度。遍历 N 个字符串，每个字符串排序耗时 `K log K`。
- **空间复杂度**：`O(N * K)`，用于存储哈希表。

#### 方法二：计数

将每个字符串的字符计数结果作为哈希表的 Key。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            int[] counts = new int[26]; // 26个小写字母
            int length = str.length();
            for (int i = 0; i < length; i++) {
                counts[str.charAt(i) - 'a']++;
            }
            
            // 将计数数组转换为唯一的字符串 Key
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < 26; i++) {
                if (counts[i] != 0) {
                    sb.append((char) ('a' + i));
                    sb.append(counts[i]);
                }
            }
            String key = sb.toString(); // e.g., "a1e1t1"
            
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```
- **时间复杂度**：`O(N * (K + |Σ|))`，其中 `|Σ|` 是字符集大小（本题为26）。遍历 N 个字符串，每个字符串计数耗时 K，生成 Key 耗时 `|Σ|`。通常比排序法更快。
- **空间复杂度**：`O(N * (K + |Σ|))`。

### 总结

从 O(N²) 到 O(N)，关键在于思维的转变：
- **抛弃“两两比较”**：这种暴力方法在数据量大时不可取。
- **拥抱“分类归档”**：寻找一个合适的**哈希键（Key）**，这个 Key 是某类数据的共同特征。
- **善用哈希表**：利用其 O(1) 的查找效率，将同类数据快速聚合。

下次遇到类似需要“分组”、“找重复”、“配对”的问题时，不妨先问问自己：**“我能为这些数据定义一个唯一的‘身份证’（Key）吗？”** 如果可以，哈希表可能就是你的最佳帮手。
