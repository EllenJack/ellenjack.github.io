---
title: 手写SpringMVC笔记
date: 2019-07-01 01:39:49
tags:
- Spring
category:
- 框架
- Spring
---
 
1,新建项目


![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpl96v9qj30d50cjta8.jpg)  ![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpla0465j30e50ciabb.jpg)

 

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpl5r9dkj30da0c8gna.jpg)  ![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpl9ihcdj30fl09oacy.jpg)

此时工程创建完毕.

 

2,新建注解及控制类,服务类及DispatcherServlet类(见源码)

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpl4tpeoj30al0fediw.jpg) 

内容分别如下:

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpl4d9o9j30my06fmzv.jpg) 

 

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jplaysorj30lf0600ub.jpg) 

 

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpl7p3r4j30um082goc.jpg) 

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpl33kvoj30pc069gnb.jpg) 

 

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpl3guuoj30oe060q4o.jpg) 

声明注解到控制层(此时注解待解析)

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpl6bt96j30um0jkgse.jpg) 

声明service服务类注解EnjoyServiceImpl

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpl8kpxbj30kd0fyjvq.jpg) 

接口类如下

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpl5bjv3j30gy08nwgg.jpg) 

 

创建一个新的DispatherServlet类,用来初始化bean和拦截请求

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpl84c1kj30he045dgn.jpg) 

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jplah0gyj30lc0fm7b1.jpg) 

它的子方法请下载云盘里的源码跟进查看,每一行代码都写了注解.

 

 

3,将DispatcherServlet配置到web.xml(在项目启动时会加载这个DispatcherServlet的init方法)

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpl3wdfsj30uk0dmai5.jpg) 

 

4,DispatcherServlet的doPost方法里对请求路径进行拦截,并根据路径到找对应要执行的方法

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpl78tszj30qx0db7ao.jpg) 

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpl6r8l9j30eq07dmyo.jpg) 

使用处理器解析后的参数放到args数组, 直接使用method.invoke(instance, args)完成请求调用;

内容较大,文字不好描述, 关于处理器这块,请看视频.

 

 

 

 

 

 

 

 

 

