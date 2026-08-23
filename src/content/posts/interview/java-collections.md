---
title: "Java 常见集合与底层源码面试题解析（List / Map / HashMap）"
published: 2026-08-22
description: "深度解析 Java 集合框架时间/空间复杂度、ArrayList 扩容机制与底层原理、LinkedList 双向链表、二叉树与红黑树、HashMap 散列表原理与 JDK1.7/1.8 差异、put 方法流程图与寻址算法。"
tags: ["Java","集合框架","HashMap","源码分析","面试"]
category: "面试"
draft: false
---

# Java集合框架
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c3a382b6dd3c3343.png)
# 复杂度
## 时间复杂度
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/11bae08045e0205d.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ef4fa4ddec3e353a.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/5ef9ded14cb3d232.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d844c2f0c124bfde.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7aa9a5bcc0c0fffe.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/fb8b78d041f20cc1.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6b5bc67db7db26ac.png)
## 空间复杂度
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8521957cbdb04ec0.png)
## 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/88856d7de79d4e47.png)
# List
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/40c603ea617a99a8.png)
## 数组
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2bfb96474650976e.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ea92183b680788ce.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/daf770f2c4da88ae.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f3c5c480a9b7ea6b.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/31e4ab2f5f3a70ce.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d91507a4998e60ad.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/e4901cee119cf7f1.png)
## ArrayList源码分析
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6f8d77c7cbd95fb8.png)
### 成员变量
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/707bfa8a163db110.png)
### 构造方法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2961b422bcc590b8.png)
### 添加数据
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/975a5d9c1498cdfa.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1435805151182422.png)
**第2到10次添加数据都不会扩容**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8e7c943131549692.png)
扩容即，将数组长度添加到原来的**1.5倍**
## 题目
### 底层实现原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/311eec63690177a7.png)
### 有参构造函数扩容问题
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/db3a7129856b9859.png)
### 如何实现List和数组的转换
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b74d953c799dc472.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/21fbf8458707e65e.png)
asList方法只涉及了对象的引用，不涉及对象的创建，因此修改数组也会影响列表
toArray方法则是复制一份，修改List不会影响数组
### ArrayList与LinkedList区别
#### 单向链表
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ed37cee98a21c9d2.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8fc0d0d9a0442587.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/295c90a81cae24b0.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b2ac874850cd4990.png)
#### 双向链表
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1ff34583e5164b45.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d20d4bacd8d49e45.png)
#### 链表对比
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/a7f48cf24751c963.png)
#### 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b9996dd839f87fa9.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/7e472475de390293.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/ff5371b90f2a3651.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/58af174900ee650c.png)
# Map
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8b4f55d2a22f84c8.png)
## 二叉树
### 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0b04556600549862.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/d0c1c5a9493a8eee.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/05b98378e223be35.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2bfc25242d63338f.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/31ba794c63f83d4c.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/71326450c391d327.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/304440a31ac9ab0a.png)
## 红黑树
### 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c24666ad4a57bd8f.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/366dcc1dbe9bb958.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6b5eb48732d1b7b2.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/9e455f0cee09ff9f.png)
## 散列表
### 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/75e42222bdc1035f.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/246f6536d0f94b1a.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/4449cfe7b471c1d6.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1349ebc2cc0b5679.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b7acdfded40047eb.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/cff676756812fcbc.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/0fa5637dc057c128.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f348d09fae75b650.png)
**链表太长会导致查询效率变为O(n)，通过将链表转换为红黑树解决**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/46524a82b7000d48.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3349d98eda70004c.png)
### 总结
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/43334db3e8d0072d.png)
## 说一下HashMap的实现原理<br>### 原理
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8df034798b8617e5.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/2e6aee585d51434f.png)
**追问：jdk1.8和jdk1.7的hashmap实现有什么区别**
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/6be9c3e065d04f3a.png)
### 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/07ff155cdc19ae48.png)
## HashMap的put方法具体流程
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/729d99eda4c41e6e.png)
### 源码分析
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8b2d1eb326e4db38.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8ecfd33e4c8fcb56.png)
### 添加数据流程图
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b8ecbeb72a547db8.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8a01c94f3d686b8e.png)
### 源码
```yaml
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断数组是否未初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        //如果未初始化，调用resize方法 进行初始化
        n = (tab = resize()).length;
    //通过 & 运算求出该数据（key）的数组下标并判断该下标位置是否有数据
    if ((p = tab[i = (n - 1) & hash]) == null)
        //如果没有，直接将数据放在该下标位置
        tab[i] = newNode(hash, key, value, null);
    //该数组下标有数据的情况
    else {
        Node<K,V> e; K k;
        //判断该位置数据的key和新来的数据是否一样
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //如果一样，证明为修改操作，该节点的数据赋值给e,后边会用到
            e = p;
        //判断是不是红黑树
        else if (p instanceof TreeNode)
            //如果是红黑树的话，进行红黑树的操作
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //新数据和当前数组既不相同，也不是红黑树节点，证明是链表
        else {
            //遍历链表
            for (int binCount = 0; ; ++binCount) {
                //判断next节点，如果为空的话，证明遍历到链表尾部了
                if ((e = p.next) == null) {
                    //把新值放入链表尾部
                    p.next = newNode(hash, key, value, null);
                    //因为新插入了一条数据，所以判断链表长度是不是大于等于8
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //如果是，进行转换红黑树操作
                        treeifyBin(tab, hash);
                    break;
                }
                //判断链表当中有数据相同的值，如果一样，证明为修改操作
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                //把下一个节点赋值为当前节点
                p = e;
            }
        }
        //判断e是否为空（e值为修改操作存放原数据的变量）
        if (e != null) { // existing mapping for key
            //不为空的话证明是修改操作，取出老值
            V oldValue = e.value;
            //一定会执行  onlyIfAbsent传进来的是false
            if (!onlyIfAbsent || oldValue == null)
                //将新值赋值当前节点
                e.value = value;
            afterNodeAccess(e);
            //返回老值
            return oldValue;
        }
    }
    //计数器，计算当前节点的修改次数
    ++modCount;
    //当前数组中的数据数量如果大于扩容阈值
    if (++size > threshold)
        //进行扩容操作
        resize();
    //空方法
    afterNodeInsertion(evict);
    //添加操作时 返回空值
    return null;
}
```
### 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/b1db08989aab5d88.png)
## HashMap扩容机制
### 流程图
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/c4fedde228fd0424.png)
### 回答
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/8ce7448154c5b7a7.png)
## HashMap寻址算法
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/f7a7952c53f13d50.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/1813af16236c2a82.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/36c5dcebe6e9df04.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/102aa163fea065ea.png)
## HashMap死循环问题
![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/05faa6407cbf52a4.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/3b302962d3579ae6.png)

![](https://cdn.jsdelivr.net/gh/Cxhahalala/hexo-images-1@main/images/973bef4b1fd3e314.png)