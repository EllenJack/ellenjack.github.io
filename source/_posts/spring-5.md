---
title: Spring组件及AOP基本使用操作
date: 2019-03-01 00:09:04
tags:
- Spring
category:
- 框架
- Spring
---

## @Autowired方法, 参数, 构造方法都可加载, 跟进源码看看

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jei1x6huj30ny03ct9o.jpg)
A>放到方法上的测试步骤:

​    新建Moon.java

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jei1uhn7j30e007v752.jpg)
 

 

   新建Sun.java

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jei1riisj30hf0anabp.jpg)

setMoon()方法使用的参数,自定义类型的值从IOC容器中获取, 方法里的moon会从容器中拿到

 

以下是验证方式

在配置类要加入扫描bean包

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jeiv0rwqj30k207ndhj.jpg)

测试:

  新建test02()测试方法

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jeiuwb6vj30o006xgn1.jpg)
 

结果如下:(为同一个bean)

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeiurwsjj30g7025aah.jpg)

方法使用的参数,自定义类型的值从IOC容器中获取, 方法里的moon会从容器中拿到

 

 

B> 将Autowired标记在有参构造器

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jej7vpbcj30ny0hpgqa.jpg)

构造函数的moon是从容器里拿到的

执行test02()测试;

 

 

 

 

 

同样,也可以放在构造器的参数位置也可以获取到IOC容器的bean.

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jej7prfqj30jw07n75w.jpg)
 

结论: 不管@Autowired是放到参数, 方法还是构造方法, 都是从容器里取到的bean...

 

 

 

 

## 自动装配:Aware注入spring底层组件原理

​    自定义组件想要使用Spring容器底层的组件(ApplicationContext, BeanFactory, ......)

​    思路: 自定义组件实现xxxAware, 在创建对象的时候, 会调用接口规定的方法注入到相关组件:Aware

​    之前讲过Plane.java就是使用这个

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jej7lt6ij30hd0etwi6.jpg)
 

 

CTRL+SHIFT+T  找到Aware

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jejy1axqj30ar03074a.jpg)

查看有哪些接口继承了Aware接口

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jejxx6mwj30fc08cjv1.jpg)
 

 

 

使用ApplicationContextAware接口为例, 实现接口

步骤:

1, 新建Light.java类, 实现ApplicaitonContextAware接口

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jejxrwqmj30i207gdhq.jpg)
 

 2,实现BeanNameAware接口

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jekhx9krj30m509qjug.jpg)
 

 

 

 

 2,实现EmbeddedValueResolverAware接口

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jekhqrwaj30m10bsdjn.jpg)
 

3, 将Light类加入@Component, 声明为被扫描的组件

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jekhl6t7j30lp03tjsi.jpg)
 

4,将test02()加入打印anno容器, 用来比如

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jeky3p6uj30ht03cmxj.jpg)
 

5,测试, 执行test()02;

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jekxo5k4j30o008maea.jpg)
 

 

**总结**:把Spring底层的组件可以注入到自定义的bean中,ApplicationContextAware是利用ApplicationContextAwareProcessor来处理的, 其它XXXAware也类似, 都有相关的Processor来处理, 其实就是后置处理器来处理;

XXXAware---->功能使用了XXXProcessor来处理的, 这就是后置处理器的作用;

   ApplicaitonContextAware--->ApplicationContextProcessor后置处理器来处理的

 

问题: Spring怎么把applicationContext容器注入进来的呢????

​    debug栈分析, 之前讲过的.

 

 

 

 

 

 

 

 

## CAP27  AOP 功能测试

AOP: 面向切面编程[底层就是动态代理]

指程序在运行期间动态的将某段代码切入到指定方法位置进行运行的编程方式

 

1,先建立MainConfigOfAOP配置类

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jekxisgdj30kr0870u5.jpg)

2,在POM.XML中导入spring-aspects依赖包

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeldkvodj30ev03laau.jpg)
 

3,新建立一个业务逻辑类Calculator.java

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jeldfoe8j309b048gm5.jpg)

在div()方法运行之前, 记录一下日志, 运行后也记录一下,运行出异常,也打印一下

有的同学可能会这么做, 在每个方法加入个打印.

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeldc0mvj30gu04zq3i.jpg)

但这样会有问题, 这种方式耦合了....

 

新建一个日志切面类....

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeltp4s4j30i80ac40m.jpg)

日志切面类的方法需要动态感知到div()方法运行到哪里了, 然后再执行, 如果除法开始, 就日志开始方法, 

   也叫通知方法, 分以下几种:

​      前置通知: logStart(),在目标方法(div)运行之前运行 (@Before)

​      后置通知:logEnd(), 在目标方法(div)运行结束之后运行,无论正常或异常结束 (@After)

​      返回通知:logReturn, 在目标方法(div)正常返回之后运行 (@AfterReturning)

​      异常通知:logException, 在目标方法(div)出现异常后运行(@AfterThrowing)

​      环绕通知:以上没写,动态代理, 手动执行目标方法运行joinPoint.procced(),最底层通知,手动指定执行目标方法(@Around), 执行之前相当于前置通知, 执行之后相当于返回通知

其实就是通过反射执行目标对象的连接点处的方法；

 

给日志切面类LogAspect的方法标注何时运行(即通知注解)

怎么加入 呢?

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jeltl88kj30ny07nwg4.jpg)

若不想区分切入了哪个方法及参数类型和个数,可以有如下指定方式:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jelthnl0j30o008in11.jpg)
 

不难发现问题:注解里的内容是冗余重复的, 公共的代码应该抽出来

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jem6mm2bj30j506pju7.jpg)

并加上Around环绕通知

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jem6iqisj30o004c0u1.jpg)

那么应该写一个公共切入点表达式.

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jem6esb6j30fu0bz0vc.jpg)
 

公共切入方法加要指定的方法

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jemkfqkxj30ny0i2q9s.jpg)
 

 

有了以上操作, 我们还需要将切面类和被切面的类, 都加入到容器中

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jemk770bj30cs0acmyp.jpg)
 

但这个时候还会有问题, Spring无法区别以上的两个bean哪个是切面类, 哪个是业务逻辑类

怎么区别呢????

只需要给切面类LogAspects上加一个注解@Aspect即可.

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jemjsrztj30ny05mq45.jpg)
 

是不是就完了呢?并没有

需要开启基于注解的AOP模式

给配置类中加@EnableAspectJAutoProxy[一定得加上,关键]

注意: 在spring以后会有很多@EnableXXXX, 表示开启某项功能, 取代XML配置

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jemzkb51j30cd09s75r.jpg)
 

测试: 新建一个测试类Cap10Test.java

同学们在测试的过程中, 应该怎么测? 很容易出问题.

大家可能会这么写:

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jemzgs9tj30fg04qwfe.jpg)

没用到容器, 肯定是不行的, 获取bean时使用IOC容器取出bean

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jemzcytjj30gs05hgmq.jpg)

 运行结果如下:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jenbhojxj309d07n0tq.jpg)
