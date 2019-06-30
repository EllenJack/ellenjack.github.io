---
title: 什么是一致性Hash算法
date: 2019-02-17 18:30:01
tags:
- 算法
- hash
category:
- 专题
- 算法
---

## Hash算法的问题

- 分布式架构缓存处理：
    Hash算法分散数据存储hash(n)%4
    同时也可以快速查找数据而不用遍历所有的服务器

- 业务拓展缓存服务器加一台
    要么缓存服务器数据全部需要重新计算存储 -----hash(n)%5  。
    要么需要遍历所有缓存服务器。
    
![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jcbjnas0j30cx0bdq3w.jpg)

## 一致性Hash算法

- Hash环
    一致性Hash算法是对2^32取模，对服务器确定确定此数据在环上的位置(比如A,B,C,D)

- 数据存放
    数据进来后对2^32 取模，得到一个值K1，在Hash环中顺时针找到服务器节点
 
![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jcbjkr56j30c00abq4z.jpg)
  
## 使用场景

- B服务失效
    如果是B失效了,将B的数据迁移至C即可，对于原本散列在A和D的数据，不需要做任何改变。

- 总结
    一致性hash算法（DHT）通过减少影响范围的方式解决了增减服务器导致的数据散列问题，从而解决了分布式环境下负载均衡问题。

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jcbjgel2j30am09b768.jpg)









