---
title: JVM的内存布局和垃圾回收机制
date: 2020-04-08 16:24:49
tags:
- jvm
category:
- 随笔
---

![](https://tva1.sinaimg.cn/large/00831rSTly1gdmgb6g26ij30w80ojgvn.jpg)

**Java虚拟机栈：**
每个方法在被调用时就会创建一个栈帧，每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程

**Java堆：**
是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，对象实例在这里分配内存。是垃圾收集器（GC）管理的主要区域

**方法区：**
存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，运行时常量池（Runtime Constant Pool）是方法区的一部分。

**直接内存：**
直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域

![](https://tva1.sinaimg.cn/large/00831rSTly1gdmg05tihcj31r50u0e2r.jpg)

**标记-清除算法(Mark-Sweep)**
![](https://tva1.sinaimg.cn/large/00831rSTly1gdmg2ns7elj31en0u0qn2.jpg)

**复制算法（Copying)**
![](https://tva1.sinaimg.cn/large/00831rSTly1gdmg3vaqz5j31ca0u0k40.jpg)

**标记-整理算法（Mark-Compact）**
![](https://tva1.sinaimg.cn/large/00831rSTly1gdmg5u0m7ij31as0u0b29.jpg)

**算法都用上**
![](https://tva1.sinaimg.cn/large/00831rSTly1gdmg7n9yedj31l60tq1kx.jpg)
