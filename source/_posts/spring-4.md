---
title: Spring的BeanPostProcessor分析及组件使用4
date: 2019-02-25 21:10:29
tags:
- Spring
category:
- 框架
- Spring
---
 
## CAP7 章节 Spring 底层对 BeanPostProcessor 的使用

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jevtebwyj30h00cf0yu.jpg)

**1,** **ApplicationContextAwareProcessor** **实现分析** **:**

此类帮我们组建IOC容器,跟进ApplicationContextAwareProcessor我们发现, 这个后置处理器其实就是判断我们的bean有没有实现ApplicationContextAware 接口,并处理相应的逻辑,其实所有的后置处理器原理均如此.

那么怎么组建呢? 只需要实现 ApplicationContextAware 接口

步骤:

​    1>, 新建Plane.java(将Jeep.java复制一份即可)

​         **class** Plane **implements** ApplicationContextAware

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jevt81hvj30hp0ecjub.jpg)

分析一下ApplicationContextAwareProcessor类的方法

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jevt3ccrj30im09umzo.jpg)

a,在创建Plane对象,还没初始化之前, 先判断是不是实现了ApplicationContextAware接口,

如果是的话就调用invokeAwareInterfaces方法, 并给里面注入值;

b,进入invokeAwareInterfaces()方法,判断是哪个aware, 如果是ApplicationContextAware, 就将当前的bean转成ApplicationContextAware类型, 调用setApplicationContext(), 把IOC容器注入到Plane里去;

c,用debug调用; 测试用例打断点测试

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jewc18u8j30jm04owg4.jpg)

d,也可看debug调用栈来分析;

**注**: debug可以打在ApplicationContextAwareProcessor处理器类的applyBeanPostProcessorsBeforeInitialization()方法里, 方便调试, 当bean为Plane类型时,F5跟进看, 最终在 InvokeAwareInterfaces()方法里返回我们的IOC容器applicationContext.

 

**2,BeanValidationPostProcess** **分析:数据校验,**

   **看BeanPostProcessor接口实现CTRL+T**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jewbwop2j30j10etn5j.jpg)
 

了解即可,处理器的原理和其它处理器一致.

**当对象创建完,给bean赋值后,在WEB用得特别多;把页面提交的值进行校验**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jewbpuy8j30o00aadi7.jpg)
 

 

 

**3,InitDestroyAnnotationBeanPostProcessor**

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jewpxzauj30mz0ajafa.jpg)
 

此处理器用来处理@PostConstruct, @PreDestroy, 怎么知道这两注解是前后开始调用的呢, 就是 **InitDestroyAnnotationBeanPostProcessor** **这个处理的**

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jewpqv55j30ia0acgng.jpg)

以@PostConstruct为例, 为什么声明这个注解后就能找到初始化init方法呢?

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jewpmmyrj30hx0cqjvm.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jexeqcotj30o00afn0r.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jexekl5bj306l032q36.jpg)
 
![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jexehewhj308g02yglm.jpg)


 

总结: Spring底层对BeanPostProcessor的使用, 包括bean的赋值, 注入其它组件, 生命周期注解功能,@Async, 等等

 

 

 

 

## CAP8 章节 @Value 赋值

新建cap8目录;

1, 新建Bird.java类

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jexr0a8ej30990b4gn2.jpg)

2,新建Cap8MainConfig.java配置类

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jexqwk96j30cq06tgmb.jpg)
 

3,新建测试用例Cap8Test.java, 从容器获取bean并打印

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jexqt5qgj30o008mjtd.jpg)
 

打印结果如下: 主要是没设值

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jey23n3mj30an01kjrg.jpg)
 

4,以前使用bean.xml配置文件进行赋值

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jey216m0j30fw02lt9n.jpg)
 

5,使用@Value赋值如何赋值呢?见下

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jey1tz2vj30o008kq4m.jpg)
 

6,从配置文件[properties]读取, 新建test.properties

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeyhsiifj309d01mwed.jpg)
 

7,test.properties内容为bird.color=red

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jeyhozywj305y0133yd.jpg)

8,在Bird类新增private String color及set和get方法;

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jeyhdcvfj30d602w74q.jpg)

9,将test.properties配置文件加载起来

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jeyu27ujj30g904c3z6.jpg)

10,再运行test01()用例, 打印出以下结果

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jeyty5rfj30f70113yj.jpg)
 

11,test.properties值是加在运行环境变量里:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jezkynvcj30ny09ygoj.jpg)
 

 

 

 

## CAP9 章节 @Autowired 自动装配

自动装配:spring利用依赖注入(DI), 完成对IOC容器中的各个组件的依赖关系赋值

 

1,新建TestController.java   TestService.java  TestDao; 分别建在指定的包内,可看步骤2.

这些所有JAVA 类的对象扫描后都是保存在IOC容器中管理的;

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jezkynvcj30ny09ygoj.jpg)
 

2,新建配置类Cap9MainConfig.java(),扫描并将以上bean都扫描并加载到容器

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jezug3z6j306602uaa3.jpg)

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jezudpwmj306f02hq32.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jezuaoppj3081029weo.jpg)
 

3, 针对以上基础类建立完成后, 可以先做个测试

在TestService.java, 使用Autowired注入,并把testDao打印出来(在测试时方便对比)

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf076ecuj30ny02y3yx.jpg)
 

4, 新建Cap9Test.java测试用例,比较TestService拿到testDao与直接从容器中拿到的testDao是否为同一个?

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jf071fr2j309u04zjrk.jpg)

结果很明显是同一个testDao,地址一样

 

**小结**:

@Autowired表示默认优先按**类型**去容器中找对应的组件,相当于anno.getBean(TestDao.class)去容器获取id为testDao的bean, 并注入到TestService的bean中;

使用方式如下:

   TestService{

​        @Autowired

​        private TestDao testDao;//默认去容器中找id为”testDao”的bean

   }

 

 

5, 注意事项

5.1如果容器中找到多个testDao, 会加载哪个testDao呢?

   操作步骤:

在Cap9MainConfig.java声明@Bean(“testDao2”)

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf06y6tmj30ny06tq4q.jpg)

并将TestDao加入flag属性和set, get及toString方法,用来分辨加载了哪个bean.

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf0qqphaj30gd05yq3x.jpg)

如何区分TestService是使用了(**@Reponstry** **的testDao** **的flag=1**)的bean还是(**testDao2** **的flag=2**)的bean?

测试步骤如下:

**1,**直接使用@Autowired, 将testDao注入到TestService

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jf0qipq8j30dn09djsj.jpg)

测试结果

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf0qfdvej30fo04v750.jpg)
 

**2,**如果一定要使用容器中的testDao2呢?操作如下:

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf133zmwj30bo00pwef.jpg)

测试结果

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf131ghzj30i205owfa.jpg)
 

**3,**虽然以上定义了private TestDao testDao2, 但还是想加载bean id为testDao(flag=1)的bean,怎么办?此时可以使用@Autowired和@Qualifier结合来指定注入哪一个bean,

操作如下:

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf12b7hij30bx00pmx3.jpg)
测试结果

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jf1facwgj30ep053755.jpg)
 

**4,**如果容器中没有任何一个testDao, 会出现什么状况呢?

操作如下: 注释掉@Repository和@Bean("testDao2")

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf1f6ipsj309y00ka9y.jpg) ![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jf1f3nn6j308706x0tf.jpg)
此时容器启动时这两个bean都不会加载(因为注解被注释啦.......)

测试结果如下:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf1ubu90j30al06rwfe.jpg)
很明显报错了, [因为@Autowired注解里的属性默认required=true.必须找到bean](mailto:因为@Autowired注解里的属性默认required=true.必须找到bean)

 

那怎么解决呢?

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf1u8x5qj30hr03gq4c.jpg)

测试结果如下:

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf1u4qrwj30h605sq3v.jpg)
 

 

**5,**@Primary注解指定bean如何加载呢?

(**注**:将以上原注释掉的@Repository和@Bean("testDao2") 恢复,见下图)

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf27zilej308i04gaag.jpg)
 

 

**重要:** **为了验证@Qualifier** **与@Primary** **两注解的加载顺序,** **测试如下**

当对于testDao在容器中同时存在多个时, 且@Qualifier与@Primary注解同时存在,会发生什么呢?

见下操作:  打开@Qualifier与@Primary注解.

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf27wr85j30av04sjs5.jpg) ![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf27tn2kj30bx048t9e.jpg)
 

测试结果:

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jf2n4oiij30a204174w.jpg)

此时只能说明一点: @Qualifier是根据bean id指定获取testDao, 不受@Primary影响.

 

 

那么@Primary的功能在哪呢?继续测试.....

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf2n1rgfj30ny011aal.jpg)

测试结果:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf2mye52j30hx06dmyj.jpg)
 

 

 

5.2除了@Autowired, 是不是还用过@Resource(JSR250)  和@Inject(JSR330) 

将Qualifier和Autowired注释掉(注意: 此时@Primary 还没注释......)

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jf357k7kj30o002wt9z.jpg)

 

测试结果:

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf350nsrj30el08ejsx.jpg)

效果也是一样的, 但它不先优先装配@Primary的bean

 

**小结:@Resource** **和Autowired** **的区别如下:**

@Resource和Autowired一样可以装配bean

@Resource缺点: 不能支持@Primary功能

​               不能支持@Autowired(required = false)的功能

 

当然也可以在TestService里按以下方式指定要注入的Bean

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf34v2lbj30ee03tdgk.jpg)

测试结果: 

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf3hg6jbj30be01gt8t.jpg)
 

 

5.3 @Inject自动装配的使用:

**注:@Inject** **与@Autowired** **的区别如下:**

@Inject和Autowired一样可以装配bean, 并支持@Primary功能, 可用于非spring框架.

@Inject缺点: 但不能支持@Autowired(required = false)的功能,需要引入第三方包javax.inject 

 
操作步骤:

1,pom.xml导入javax.inject包

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jf3hcuu5j30ch02yq3c.jpg)
 

2,使用@Inject注解

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf3h75nwj30l40a8wh7.jpg)
 

 

结论:@Inject不支持required=false,  但支持primary

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jf3rd9jqj30jg03jwf8.jpg)

Autowired属于spring的, 不能脱离spring,  @Resource和@Inject都是JAVA规范

推荐大家使用@Autowired

 

 
