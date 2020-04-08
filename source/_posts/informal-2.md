---
title: 类加载机制和双亲委派模型
date: 2020-04-08 15:29:15
tags:
- 类加载
category:
- 随笔
---

## 类加载机制

![](https://tva1.sinaimg.cn/large/00831rSTly1gdmedewb6ij31hc0jw4nt.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gdmedei3xoj31k20u07o6.jpg)

## 双亲委派模型
![](https://tva1.sinaimg.cn/large/00831rSTly1gdmedeogskj30vt0l0aif.jpg)

**双亲委派模型过程**
某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

**双亲委派模型好处**
Java类随着它的类加载器一起具备了带有优先级的层次关系，保证java程序稳定运行

