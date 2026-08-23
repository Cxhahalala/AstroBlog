---
title: "Java 常见集合与底层源码面试题解析（List / Map / HashMap）"
published: 2026-08-22
description: "深度解析 Java 集合框架时间/空间复杂度、ArrayList 扩容机制与底层原理、LinkedList 双向链表、二叉树与红黑树、HashMap 散列表原理与 JDK1.7/1.8 差异、put 方法流程图与寻址算法。"
tags: ["Java","集合框架","HashMap","源码分析","面试"]
category: "面试"
draft: false
---

# Java集合框架
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784624882669-b76476de-09a2-4524-bd21-4f516851a07e.png)
# 复杂度
## 时间复杂度
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625280776-588cc455-098c-43b9-8c11-6007a3848038.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625261727-bb7489b7-2ccc-46c1-af0b-9f3b221c7bbe.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625352285-49403731-d3f5-4abe-bb10-f941fa31a2b5.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625423963-e9066a1a-bbe7-4847-b3c0-275e482be84c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625552805-af800292-cca1-43d4-a21b-1a2e8c28da9d.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625618165-7a1e3fd8-50a3-4681-b45c-d8e40e92b19c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625644498-95b0cc0d-cf0f-4e76-b8a4-dbf57885a28a.png)
## 空间复杂度
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625708830-78c87039-94c4-425a-9f87-e2613fe8dc9d.png)
## 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625781266-782f8c36-13e6-4e9e-969f-f8b35bf5cc21.png)
# List
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625832647-6ada8b9f-16d3-4d45-96b9-8b0ee27065d9.png)
## 数组
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625899863-e8414806-c028-4394-b87f-501a8e5f672e.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784625979754-08c0cfee-1363-44e5-8d72-3d7be3e12b06.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626065969-84f2bb67-0276-47d9-9c75-6962358a4960.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626142588-a3e6c8f1-19d0-411a-a139-842bc6bdcfe6.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626185460-2eb88245-35f9-4426-b21b-037e50dca8b1.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626234471-455c50c1-75bc-41d4-b67f-6a9ae68cae64.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626290795-4d49f56d-544f-4f85-9f4b-48c6a3ecc021.png)
## ArrayList源码分析
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626378469-43cc8669-e88a-4833-9692-71f266248fd9.png)
### 成员变量
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626464948-4d60a1c9-b472-4cdf-889d-486d554398ca.png)
### 构造方法
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784626635051-05517512-f0fc-4294-bb0a-2def8af5625a.png)
### 添加数据
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627020160-a7902c1b-3332-4aff-b410-5e05934087b1.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627240194-6e406cdd-f3cb-489f-aadf-9d19f3f41181.png)
**第2到10次添加数据都不会扩容**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627480423-43ef726c-0c08-4c6a-9c0a-fe9b49a42d71.png)
扩容即，将数组长度添加到原来的**1.5倍**
## 题目
### 底层实现原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627632544-d99bfa0d-aba9-4cc5-983a-f8b9e7f4c404.png)
### 有参构造函数扩容问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627759297-09dc64e3-5eac-43d2-9a72-d59363e1e828.png)
### 如何实现List和数组的转换
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784627922383-cba4d14f-4c05-420a-a98e-4a77fdff41c6.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784628311095-844cea60-9949-4b90-8237-3e63f0e472ce.png)
asList方法只涉及了对象的引用，不涉及对象的创建，因此修改数组也会影响列表
toArray方法则是复制一份，修改List不会影响数组
### ArrayList与LinkedList区别
#### 单向链表
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629161230-33c029d0-3184-4236-9492-791a22269fdd.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629192457-663609a9-ebe4-4575-a124-9a9bdbc12144.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629239680-1bb333a1-c84b-49f8-884a-25ac891f3163.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629297118-01611bf4-0c51-43a0-a65a-0889765e0a55.png)
#### 双向链表
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629421653-b43be5d9-9595-468d-8870-15953c63104d.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629481196-090c2931-f2d3-4dc5-958d-fc2b83868874.png)
#### 链表对比
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629532253-b37a9c6d-2ade-44f2-98cd-d85ff16be9d9.png)
#### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629665103-c4860571-2fb7-4126-b2fb-7a5fc69c7d7c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629785310-c01c73ba-16aa-43ba-9bb3-7c3b013406b6.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629867941-7b73be5a-9020-49f8-8d75-437028f6d68f.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629903535-10cf4d48-1aab-44e9-aa99-44b7e5e1bf87.png)
# Map
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784629963667-021b7401-48b9-43f9-84a4-41e538975a0f.png)
## 二叉树
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630010943-cbfc9b03-e83d-4955-bc66-bdaf349e24a1.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630042037-50f63b53-e14d-4b1b-ba30-66be3dcf6efd.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630064854-8a4232ba-b114-4c4c-831f-44b47c2ed817.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630104999-d7f95928-2d3d-45fd-8e9e-a528c7cb82d7.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630187621-e4a42075-995c-431b-802f-4b00e6b13a8b.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630218062-6e6ff846-f684-4e72-9bd5-676ad7596cbc.png)
### 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630258411-98896f9b-4a1d-45b4-864f-0b7c56e594c4.png)
## 红黑树
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630300960-17074a9d-81f1-4347-a586-06b7b5dd5c2f.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630397673-01a5f2a5-25c7-43ce-8bac-032f70e46e09.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630449373-dccab6f4-7641-4ef8-b93d-6c06ba81559e.png)
### 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630458061-6dd75c8d-49c2-4448-a4bb-b89b7225ec7e.png)
## 散列表
### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630484288-0b56c42b-5e19-47a0-9507-b4426c9a6969.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630541973-ca24753a-7dfb-40de-ae0e-71fccae739fb.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630589185-ff1124ef-cd5a-43e3-bda5-905477e54016.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630653289-aa36e314-5ab5-41bf-a9a4-3cb607fa6fc7.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630703941-df819d47-4076-4ee3-8170-af5e2338d2ee.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630761185-54b3aef5-7f30-4d3b-ad24-1d5e51c0e2d3.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630788944-aa3e194a-f8e4-48a5-aba3-d094abcfd61c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630852137-326aff5f-dd17-435c-adad-41a729713f08.png)
**链表太长会导致查询效率变为O(n)，通过将链表转换为红黑树解决**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630949932-4ed3b299-e94d-4806-bc1d-6b961f2f581a.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784630973894-54583006-450c-4911-ac02-f14631991bab.png)
### 总结
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631010478-ee71040b-80a1-48c4-9997-91868d840b25.png)
## 说一下HashMap的实现原理<br>### 原理
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631158744-2183f938-e87e-4dd2-a56f-0a07961b370c.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631278911-c6e07b30-a567-4146-8026-8635bdd47090.png)
**追问：jdk1.8和jdk1.7的hashmap实现有什么区别**
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631346902-d3e648df-fe9b-4962-9af8-83337aecb4d6.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631424150-c7d92613-d604-4357-9c11-dbeb6f97178a.png)
## HashMap的put方法具体流程
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631478047-ffc4dd07-e658-4ff9-9f36-e12a8e6d04e7.png)
### 源码分析
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631579837-67a33ef1-b6c4-4148-a814-10819dc6a17f.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631631318-6654b04c-f3d2-433b-807d-91c58025a888.png)
### 添加数据流程图
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784631859537-c75712bb-b74e-48fa-85b6-a39debea8344.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784632090548-c7044510-7d48-4a9a-a5fc-884235d0e9e4.png)
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
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784632213430-ba9f3fab-2482-4a2a-8db1-c2041fd80871.png)
## HashMap扩容机制
### 流程图
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784633655523-c513de70-56b9-4c94-a6fe-989f99b10c3a.png)
### 回答
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784633870864-f786ed1f-0435-420b-a4a7-f1928a24d56d.png)
## HashMap寻址算法
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784634265892-d340e7f2-fcbe-4be1-8448-9b8456f75024.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784634650552-53acab9c-29f2-4c20-8ead-7986e529ff70.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784634831329-a42417bc-b97b-471e-a369-63c7ddcdbe5b.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784634890495-a81af6ae-4693-43a7-ad82-ee35e921bb9c.png)
## HashMap死循环问题
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784635069906-3a3afa2f-4fd2-43ea-83de-d0145b97fe1f.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784635114130-cce0c3ad-bd2f-42e6-ae47-21fbb4b88c38.png)
![](https://cdn.nlark.com/yuque/0/2026/png/32814455/1784635338363-1fb5effd-ed45-4d37-8210-aef936b3f837.png)