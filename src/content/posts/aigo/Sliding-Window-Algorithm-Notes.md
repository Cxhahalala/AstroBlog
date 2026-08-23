---
title: "滑动窗口算法"
published: 2025-07-11
description: "滑动窗口算法笔记：详解双指针与滑动窗口在字符串及数组问题中的核心思想、解题通用模板与经典例题解析。"
image: "https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20260817163106371.png"
tags: ["滑动窗口","算法"]
category: "算法笔记"
draft: false
---

# 滑动窗口算法笔记

## 什么是滑动窗口算法

滑动窗口算法是一种常用的算法技巧，主要用于解决数组/字符串的子数组/子字符串问题。它通过维护一个窗口，在线性时间内解决一些原本需要嵌套循环的问题。

### 核心思想

- 用两个指针（left 和 right）表示窗口的左右边界
- right 指针负责扩展窗口，left 指针负责收缩窗口
- 通过移动指针来调整窗口大小，寻找满足条件的子数组

### 适用场景

1. **固定窗口大小**：窗口大小固定，求窗口内的最大值、最小值、平均值等
2. **可变窗口大小**：根据条件动态调整窗口大小
3. **字符串匹配**：在字符串中寻找包含特定字符的子串

## 滑动窗口的类型

### 1. 固定窗口
```java
public void fixedWindow(int[] nums, int k) {
    for (int i = 0; i <= nums.length - k; i++) {
        // 处理窗口 [i, i+k-1]
        for (int j = i; j < i + k; j++) {
            // 处理窗口内元素
        }
    }
}
```

### 2. 可变窗口（收缩型）
```java
public void variableWindow(int[] nums) {
    int left = 0;
    for (int right = 0; right < nums.length; right++) {
        // 扩展窗口
        // 添加 nums[right] 到窗口
        
        while (/* 窗口需要收缩的条件 */) {
            // 收缩窗口
            // 移除 nums[left] 从窗口
            left++;
        }
        
        // 更新结果
    }
}
```

## 解题模板

```java
public int slidingWindow(int[] nums, int target) {
    int left = 0, right = 0;
    int windowSum = 0; // 窗口内元素和（根据题目调整）
    int result = 初始值;
    
    while (right < nums.length) {
        // 1. 扩展窗口：将 nums[right] 加入窗口
        windowSum += nums[right];
        
        // 2. 判断窗口是否需要收缩
        while (/* 收缩条件 */) {
            // 3. 更新结果（在收缩前）
            result = Math.min/max(result, right - left + 1);
            
            // 4. 收缩窗口：将 nums[left] 移出窗口
            windowSum -= nums[left];
            left++;
        }
        
        // 5. 移动右指针
        right++;
    }
    
    return result;
}
```

## 实战题目：LeetCode 209 - 长度最小的子数组

### 题目描述
给定一个含有 n 个正整数的数组和一个正整数 target，找出该数组中满足其和 ≥ target 的长度最小的连续子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/20250710213115202.png)
### 解题思路

1. **暴力解法**：时间复杂度 O(n²)，枚举所有子数组
2. **滑动窗口**：时间复杂度 O(n)，用双指针维护窗口

### 代码实现

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int length = nums.length;
        int left = 0;                    // 左指针
        int result = Integer.MAX_VALUE;  // 结果：最小长度
        int sum = 0;                     // 当前窗口和
        
        // 特殊情况处理
        if (length == 0) {
            return 0;
        }
        
        // 滑动窗口
        for (int right = 0; right < length; right++) {
            // 1. 扩展窗口：将 nums[right] 加入窗口
            sum += nums[right];
            
            // 2. 收缩窗口：当窗口和 >= target 时
            while (sum >= target) {
                // 3. 更新结果：记录当前窗口长度
                result = Math.min(result, right - left + 1);
                
                // 4. 收缩窗口：移除 nums[left]
                sum -= nums[left];
                left++;
            }
        }
        
        // 5. 返回结果
        return result == Integer.MAX_VALUE ? 0 : result;
    }
}
```

### 算法分析

**时间复杂度**：O(n)
- 虽然有嵌套循环，但每个元素最多被访问两次（一次被 right 指针访问，一次被 left 指针访问）

**空间复杂度**：O(1)
- 只使用了常数个额外变量

### 执行过程示例

以 `nums = [2,3,1,2,4,3]`, `target = 7` 为例：

```
初始: left=0, right=0, sum=0, result=∞

right=0: sum=2, sum<7, 继续扩展
right=1: sum=5, sum<7, 继续扩展  
right=2: sum=6, sum<7, 继续扩展
right=3: sum=8, sum>=7, 开始收缩
    result = min(∞, 3-0+1) = 4
    left=0: sum=8-2=6, sum<7, 停止收缩
right=4: sum=10, sum>=7, 开始收缩
    result = min(4, 4-1+1) = 4
    left=1: sum=10-3=7, sum>=7, 继续收缩
    result = min(4, 4-2+1) = 3
    left=2: sum=7-1=6, sum<7, 停止收缩
right=5: sum=9, sum>=7, 开始收缩
    result = min(3, 5-3+1) = 3
    left=3: sum=9-2=7, sum>=7, 继续收缩
    result = min(3, 5-4+1) = 2
    left=4: sum=7-4=3, sum<7, 停止收缩

最终结果: 2
```

## 常见变形题目

1. **LeetCode 3 - 无重复字符的最长子串**
2. **LeetCode 76 - 最小覆盖子串**
3. **LeetCode 438 - 找到字符串中所有字母异位词**
4. **LeetCode 567 - 字符串的排列**

## 总结

滑动窗口算法的关键在于：
1. **正确定义窗口的含义**
2. **明确扩展和收缩的条件**
3. **在合适的时机更新结果**

掌握了滑动窗口的思想和模板，就能高效解决一大类数组和字符串问题。
