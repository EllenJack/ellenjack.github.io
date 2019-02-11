---
title: 动态代理的几种实现方式及优缺点
date: 2019-02-12 02:19:26
tags: 动态代理
category:
- 专题
- 动态代理
---

#动态代理是什么？

> 动态代理：是使用反射和字节码的技术，在运行期创建指定接口或类的子类（动态代理）以及其实例对象的技术，通过这个技术可以无侵入行的为代码进行增强

![img](/hexo/images/dynamic-proxies-1.png)

## Java的动态代理技术实现主要有两种方式

- JDK原生动态代理

- CGLIB动态代理

### 1. JDK 原生动态代理

- Proxy : Proxy是所有动态代理的父类，它提供了一个静态方法来创建动态代理的class对象和实例

- InvocationHandler : 每个动态代理实例都有一个关联的InvocationHandler。 在代理实例上调用方法时，方法调用将被转发到InvocationHandler的invoke方法；

![img](/hexo/images/dynamic-proxies-2.png)

code 实例：

```
public UserServiceInterceptor(Object realObj) {
    super();
    this.realObj = realObj;
}


public Object invoke(Object proxy, Method method, Object[] args)throws Throwable {
    if(args!=null && args.length>0 && args[0] instanceof User){
        User user = (User) args[0];
        if(user.getName().trim().length()<=1){
            throw new RuntimeException("用户姓名输入长度需要大于1！");
        }
    }
    Object ret = method.invoke(realObj, args);
    logger.info("数据库操作成功！");
    return ret;
}

User user = new User();
user.setAddress("地址");
user.setAge(20);
user.setName("1ison");
UserService us = new UserServiceImpl();
UserServiceInterceptor usi = new UserServiceInterceptor(us);
UserService proxy = (UserService) Proxy.newProxyInstance(us.getClass().getClassLoader(), us.getClass().getInterfaces(), usi);
proxy.addUser(user);
```

### 2. CGLIB动态代理

> CGLIB (Code Generation Library) 是一个基于ASM的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB通过继承方式实现代理

- Enhancer : 来指定要代理的目标对象、实际处理代理逻辑的对象，最终通过调用 create() 方法得到代理对象，对这个对象所有非final方法的调用都会转发给MethodInterceptor;

- MethodInterceptor : 动态代理对象的方法调用都会转发到 intercept 方法进行增强；

![img](/hexo/images/dynamic-proxies-3.png)

code 实例：

```
public Object intercept(Object obj, Method method, Object[] args,MethodProxy proxy) throws Throwable {
    if(args!=null && args.length>0 && args[0] instanceof User){
        User user = (User) args[0];
        if(user.getName().trim().length() <= 1){
            throw new RuntimeException("用户姓名输入长度需要大于1！");
        }
    }
    Object ret = proxy.invokeSuper(obj, args);
    logger.info("数据库操作成功！");
    return ret;
}

User user = new User();
user.setAddress("地址");
user.setAge(20);
user.setName("lison");
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(UserServiceImpl.class);
enhancer.setCallback(new UserServiceInterceptor());
UserServiceImpl usi1 = (UserServiceImpl) enhancer.create();
usi1.addUser(user);  
```

## 总结

- JDK原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理；

- CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法处理final的情况。



