---
title: Transaction 注解
date: 2019-02-15 22:54:59
tags: 
- 事务
category:
- 专题
- 事务
---

> 默认情况下，数据库处于自动提交模式。每条sql处于一个单独的事务中，成功则提交，失败则回滚。事务处理就是将一组相关的sql放于一个事务中，因此必须关闭数据库的自动提交模式

## @Transaction注解一般写在什么位置? 如何控制其回滚？

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jegsglbsj30on0c1jsy.jpg)

## @Transaction事务回滚规则

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jegs4sh7j30pn0co404.jpg)

## 总结

- 使用位置：

1. 用在接口或接口方法上，AOP必须是接口代理方式。不推荐	

2. 可以使用在类以及类方法上。推荐

3. 注解应该只被应用到 public 方法上。其它级别（ protected ，private无效）

4. 只有来自外部的方法调用，事务才生效。（不能由本地方法直接调用）

- 回滚控制：

1. 默认配置下，方法体只有在抛出RuntimeException或其子类时，才回滚事务。

2. 可以指定哪些异常回滚： @Transactional(rollbackFor=XxxException.class)

3. 可以指定哪些异常不回滚： @Transactional(noRollbackFor=XxxException.class)






