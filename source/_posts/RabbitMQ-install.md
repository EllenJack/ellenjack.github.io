---
title: RabbitMQ 安装部署
date: 2018-12-08 22:01:01
tags: RabbitMQ
category: 
- 消息中间件2222222222222222
- RabbitMQ
common: on
---

这里注意获取镜像的时候要获取management版本的，不要获取last版本的，management版本的才带有管理界面。

# 获查询镜像

```
 docker search rabbitmq:management
```

可以看到如下结果：

```
[root@localhost ~]# docker search rabbitmq:management
INDEX       NAME                                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/macintoshplus/rabbitmq-management   Based on rabbitmq:management whit python a...   1                    [OK]
docker.io   docker.io/transmitsms/rabbitmq-sharded        Fork of rabbitmq:management with sharded_e...   0
[root@localhost ~]#
```

<!--more -->

# 获取镜像

```
docker pull rabbitmq:management
```

可以看到如下结果

```
[root@localhost ~]# docker pull rabbitmq:management
Trying to pull repository docker.io/library/rabbitmq ...
management: Pulling from docker.io/library/rabbitmq
e7bb522d92ff: Pull complete
ad90649c4d84: Pull complete
5a318b914d6c: Pull complete
cedd60f70052: Pull complete
f4ec28761801: Pull complete
b8fa44aa9074: Pull complete
e8002a209c24: Pull complete
cd1206edcd43: Pull complete
769be0727074: Pull complete
7308b93d35af: Pull complete
c4102ef22c29: Pull complete
fefc8e1aa4b5: Pull complete
a271d400045b: Pull complete
b0d4c40c62de: Pull complete
Digest: sha256:8761de2c22badfc86dfe89791dc9dbf122f67ff0f8981966573d267af421b97f
[root@localhost ~]#
```

# 运行镜像

```
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
```

看到如下结果，变成功了：

```
[root@localhost ~]# docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
e194a2dbeb52f2296dfb6d1c527cf052d82be5ed9a4c974d70dcd6af3da3eb7e
[root@localhost ~]#
```

# 访问管理界面

访问管理界面的地址就是 http://[宿主机IP]:15672，可以使用默认的账户登录，用户名和密码都guest，如：



![img](/hexo/images/RabbitMQ-1.png)
