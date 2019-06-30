---
title: HashMap底层执行原理
date: 2019-02-17 17:34:14
tags:
- 集合
category:
- 专题
- 集合
---

# HashMap底层执行原理

## 数据结构

- HashMap的存储结构
    数组、链表、红黑树（Jdk1.8）
    
- 特点
    1>快速存储
    
    2>快速查找（时间复杂度0（1））
    
    3>可伸缩 
      
![](http://ww4.sinaimg.cn/large/006tNc79ly1g4j40kyx3cj30h008w0t2.jpg)

## Hash算法

- hash 算法
    所有的对象都有hashCode(使用key的)
    
    hash值的计算
    
    (hashCode) ^ (hashCode >>> 16) 

- 数组下标计算
    数组默认大小：16
    
    数组下标：hash&(16-1) = hash% 16

## Hash冲突

- Hash冲突：
    不同的对象算出来数组下标是相同的

- 单向链表
    用于解决Hash冲突的方案，加入一个next记录下一个节点

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4j40kvqmdj30c407mdi8.jpg)

## 扩容

- 扩容
    数组变长2倍 0.75

- 触发条件
    数组存储比例达到 75% --- 0.75

## 红黑树

- 红黑树
    一种二叉树，高效的检索效率

- 触发条件
    在链表长度大于8的时候，将后面的数据存在红黑树中

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4j40kr6ddj30go0da74r.jpg)









