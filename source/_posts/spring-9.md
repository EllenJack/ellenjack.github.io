---
title: Servlet3.0与SpringMvc那些事
date: 2019-07-01 01:16:48
tags:
- Spring
category:
- 框架
- Spring
--- 

1，以前来写web的三大组件：以前写servlet filter listener都需要在web.xml进行注册，包括springmvc的前端控制器DispactherServlet也需要在web.xml注册，现在可以通过注解的方式快速搭建我们的web应用

2，servlet3.0需要tomcat7以上版本进行支持

3，创建动态web工程（Dynamic Web Project），看一看原生版的servlet：

   步骤如下

3.1创建工程：

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jowb7l5xj30dd0jrn16.jpg)   ![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jowfa9coj30dk0k1tad.jpg)

 

  3.2新建jsp页面

   ![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowfrtbzj30hp0l7mzq.jpg)

 

  3.3打开页面，新增请求地址，请求地址为order

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jowgp0n6j30um0caq84.jpg) 

 

3.4那么我们建立一个原生的servlet来处理 order的请求

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joubc54fj30um07wadh.jpg) 

加入tomcat，启动后测试访问地址，结果如下：

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jouieozxj30um04zt9e.jpg) 

可以访问到路径地址。

当然这些注解不是我们讲的重点，原生的servlet开发还是很少的。

 

 

 

 

 

 

4，Shared libraries（共享库） and runtimes pluggability（运行时插件）的原理,在后面的框架整合里，用得比较多，来分析下它；

**ServletContainerInitializer初始化web容器：**

在web容器启动时为提供给第三方组件机会做一些初始化的工作，例如注册servlet或者filters等，servlet规范(JSR356)中通过ServletContainerInitializer实现此功能。

每个框架要使用ServletContainerInitializer就必须在对应的jar包的**META-INF/services** 目录创建一个名为javax.servlet.ServletContainerInitializer的文件，文件内容指定具体的ServletContainerInitializer实现类，那么，当web容器启动时就会运行这个初始化器做一些组件内的初始化工作。

 

**操作步骤：**

4.1创建META-INF/services目录

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowhl8j1j30j309itbj.jpg) 

4.2创建javax.servlet.ServletContainerInitializer文件

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowi1jjqj30fr02n0t5.jpg) 

 

4.3,新建JamesServletContainerInitializer实现ServletContainerInitializer接口

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jp06mvxlj30om0a9jvi.jpg)

4.4编辑javax.servlet.ServletContainerInitializer文件内容

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou92r6aj30nz02r3zk.jpg) 

 

4.5一般伴随着ServletContainerInitializer一起使用的还有HandlesTypes注解，通过HandlesTypes可以将感兴趣的一些类注入到ServletContainerInitializer的onStartup方法作为参数传入。

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jou40zfqj30ul0haqby.jpg) 

并新建JamesService接口的所有子类型

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jp20k6krj30a601yaa2.jpg)

JamesService接口：

分别新建JamesServiceOther,JamesServiceImpl,AbstractJamesService

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jouh2c50j30f901zwet.jpg) 

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jowddqrej30fx023aac.jpg) 

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jou3iyyzj30kv028t95.jpg) 

 

 

4.6启动tomcat测试，看打印日志，不难发现，都拿到了，可以根据需要来反射创建对象

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joujtmlgj30ll066gpd.jpg) 

这其实就是基于运行时插件的机制，启动并运行这个ServletContainerInitializer，在整合springmvc的时候会用到

 

4.7使用ServletContext注册web组件（其实就是Servlet,Filter,Listener三大组件），

对于我们自己写的JamesServlet，我们可以使用@WebServlet注解来加入JamesServlet组件，

但若是我们要导入第三方阿里的连接池或filter，以前的web.xml方式就可通过配置加载就可以了，但现在我们使用ServletContext注入进来；

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jowiik05j30ul05mq5i.jpg) 

操作步骤：新建以下三个组件

A，新建OrderFilter.java过滤器

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jou88pv1j30k20ca0vh.jpg) 

B，新建OrderListener.java监听类

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jouf3m1xj30kj0f9wj5.jpg) 

 

C，新建OrderServlet.java类

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4joucvlgsj30oa06c0uq.jpg) 

D，使用ServletContext来注册以上三个组件

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou9k6iij30um0dfdn3.jpg) 

 

注意：在运行的过程中，是不可以注册组件， 和IOC道理一样，出于安全考虑

 

 

 

 

 

 

 

 

 

5，利用以上机制来整合springmvc;创建一个新的maven工程,springmvc注解版

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joweiw9cj30el0dgta8.jpg) ![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jowcgnxsj30eo0dfmz2.jpg)

 

 

5.1，建立完工程后，pom.xml会报错，老铁们，怎么办？？？不要慌，哈哈看下面吧

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jowbzomcj30my082n0v.jpg) 

做个设置即可

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joukpeufj30jj0btgpi.jpg) 

再右键工程名，update更新一下maven配置就不会有错啦 啦 ……

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joufkqyuj30lz07zac7.jpg) 

 

 

5.2，加入必要的jar包依赖

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowg89q1j30o909in0h.jpg) 

 

5.3，导入依赖包后，查看maven的一个spring-web.jar包

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou7rk3hj30k10kl0zr.jpg) 

 

打开ServletContainerInitializer这个文件看看

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jouagvv4j30mj045tal.jpg) 

 

 

5.4打开SpringServletContainerInitializer源码类

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jouay059j30uk0dk45m.jpg) 

 

5.5打开WebApplicationInitializer源码看看组件及实现（ctrl+t）

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jouiv5m6j30ul05j0w7.jpg) 

 

子类AbstractContextLoaderInitializer作用：

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jowcyd0cj30pl0fy11g.jpg) 

 

 

子类AbstractDispatcherServletInitializer的作用：从名字来看可知是DispatcherServlet初始化

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joud8x1vj30ul0lutnb.jpg) 

 

子类AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的dispatcherServlet初始化器

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joucduzyj30uk0fpn5e.jpg) 

 

root根容器与servlet容器的区别在哪呢？父子容器

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowjgabkj30lu0jd0wh.jpg) 

很明显，servlet的容器用来处理@Controller，视图解析，和web相关组件

而root根容器主要针对服务层，和数据源DAO层及事务控制相关处理（图源自spring官网）

https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/web.html#mvc-servlet-context-hierarchy

 

后面我们根据这些来配置操作一下。

 

 

 

 

 

6，与springmvc的整合流程。

操作步骤：

新建JamesWebInitializer继承AbstractAnnotationConfigDispatcherServletInitializer类

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jowj5algj30ul0hptev.jpg) 

 

新建两个配置类JamesRootConfig和JamesAppConfig，形成父子容器的效果

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jowh3eh6j30rt093adw.jpg) 

 

再建JamesAppConfig类

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou59z58j30um092djm.jpg) 

再建OrderController控制类来测试

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jou79h5aj30oj0bhdjr.jpg) 

 

再建OrderService服务层类

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joug1n5jj30hl07rdhs.jpg) 

 

OrderController来调用一下service组件

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joua0d2zj30i908nwgp.jpg) 

 

注意：JamesWebAppInitializer还需要指定配置类（配置文件）位置，修改以下返回

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4joulosimj30ul0gbjy1.jpg) 

 

重启tomcat，进行测试：

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jouk9pxqj30ul056gmr.jpg) 

这就使用注解的方式（配置类）来完成配置springmvc的整合

 

 

 

 

 

 

7，如何定制与接管springmvc

以前是通过配置的方式来完成相应的处理

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou5q71hj30uk0960wt.jpg) 

现在使用配置注解，定制我们的springmvc,可看官网描述，加入@EnableWebMvc,来定制配置功能。

我们直接在JamesAppConfig实现WebMvcConfigurer

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jouhjovvj30um0k87dv.jpg) 

 

点击进去WebMvcConfigurer， ctrl+t发现WebMvcConfigurerAdapter实现了WebMvcConfigurer接口，但方法都为空，我们只要继承一下这个类即可

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jougk068j30ul05kdih.jpg) 

 

开始继承WebMvcConfigurerAdapter这个类（可以挑选部分方法进行重写）。

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jou4ba7nj30um09xgpi.jpg) 

 

我们得新建这些目录

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jowbfy4jj30hy06d0tj.jpg) 

 

在pages下新建一个名为 ok.jsp的页面

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jou4t47gj30ee02egln.jpg) 

页面内容如下：

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jou8likfj30ul080tby.jpg) 

 

 

直接在OrderController控制类加一个解析器的定制页面返回

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jou66szoj30jp0dfq67.jpg) 

 

重启tomcat，测试一下，结果为如下：

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4joudua7zj30pq049my1.jpg) 

 

 

 

 

 

静态资源如何配置访问呢？

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jouhxcjrj30uk0an42k.jpg) 

xml配置文件有个<mvc:default-servlet-handler/>，效果一样。

比如新增一张图片和一个JSP页面

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joubtdpkj30in0793za.jpg) 

新增index.jsp修改一下：

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4joweusjbj30lk0633za.jpg) 

浏览器测试结果如下，能访问到页面

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jou6nv2tj30li06gaau.jpg) 

 

 

 

接下来我们玩一个稍微复杂点的，拦截器？

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jowdw20nj30oi0egte2.jpg) 

 

新建 JamesInterceptor拦截器，需实现HandlerInterceptor接口

之前在xml里有， <mvc:interceptors>,效果是一样的。

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4joue606tj30ul0cewj8.jpg) 

 

把拦截器加到子容器配置类

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4joujb6f9j30pb0dd0xr.jpg) 

 

重启tomcat进行测试，输入网些访问ok方法

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jouemjufj30k8045dgr.jpg) 

同时eclipse打印内容如下，拦截成功。

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4joul7iwkj30mr04u40i.jpg) 


 

8，servlet3.0异步请求分析

 

8.1，什么是同步处理，请求发出后，等待服务端响应

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe5k2ftj30h40f50ts.jpg) 

8.2同步请求原理，从tomcat中获取连接线程进行处理，但tomcat的线程数有限，会造成线程资源的紧张。

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpe3k7t9j30jg0cqmym.jpg) 

8.3同步机制操作步骤：

a,在servlet工程目录下，修改JamesServlet类，把当前处理的线程也打印出来

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpe41wg7j30uk0ckjvt.jpg) 

 

重启tomcat，得到如下结果

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe1no3pj30sf03w0ur.jpg) 

从头到尾都是由同一个线程4进行处理的，同一线程处理

很明显，线程从头执行到尾，会造成资源占用不能释放

 

b,异步请求操作步骤：

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe9po0hj30wt0lek0v.jpg) 

从servlet3.0文档的9.6章节也可以看到，要声明的内容

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpe6cqs6j30qb05m0vu.jpg) 

 

重启tomcat,查看运行结果如下，主线程与副线程分别为不同的线程，主线程从开始到结束不等待副线程就返回了：

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpe57b3bj30um055tco.jpg) 

页面返回结果如下：等待3S后才返回，但线程资源其实早已释放

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe5xo0jj30um04bdgl.jpg) 

 

 

c,异步请求原理

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpe09ogjj30um0ehadp.jpg) 

 

当然怎么样定义servlet的异步处理线程池，不多讲，springmvc已集成了异步处理线程池

 

 

 

 

 

 

 

 

9，springmvc的异步请求

我们可以打官网

https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/web.html#mvc-servlet-context-hierarchy

第1.7.1章节，讲述得很清晰

springmvc异步机制是基于servlet3来做的封装处理，通过这两种返回值都可以完成异步

官网例子如下：

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpe36io1j30df05n3zd.jpg)或![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpe15fqcj30g005idh5.jpg)

我们接下来实现一下：

操作步骤：

a,新建AsyncOrderController测试类 

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe8a3ttj30ul0e7ag9.jpg) 

控制台打印结果如下：

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpean64dj30um03ctbi.jpg) 

 

页面运行结果如下：

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpeb1t65j30um03zq3q.jpg) 

 

Callable的原理是什么呢？还是一样，请打开官网哈…………第1.7.1章节

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe0pzq0j30qe06paec.jpg) 

 

思考：控制台打印结果为什么会两次进入拦截器preHandle呢？

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpe98vzwj30um080tdc.jpg) 

很明显可知：请求进入时拦截了一次，将Callable返回结果时，将请求重新派发给容器时又拦截了一次，所以进了两次拦截；

 

如何验证？只要在拦截器打印的地方加上getRequestURI()便知晓。

![img](http://ww1.sinaimg.cn/large/006tNc79ly1g4jpea6mv1j30ul050dil.jpg) 

 

重启tomcat，并测试结果如下：分析见箭头的中文描述：

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe6szs5j30ul0bndno.jpg) 

 

 

 

 

 

 

 

10，springmvc异步请求及返回实战

 

以上只是原理，但在开发的过程并不是像Callable简单

比如现在我们有以下需求：

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe7sb5tj30pf0bedi1.jpg) 

需求描述：以创建订单为例，tomcat启动线程1来完成一个请求，但实际上是订单服务才能创建订单，那么tomcat线程应该把请求转发给订单服务，使用消息中间件来处理，订单服务把处理结果也放到消息中间件，由tomcat的线程N拿到结果后，响应给客户端。

 

我们打开官网第1.7.1章节

![img](http://ww2.sinaimg.cn/large/006tNc79ly1g4jpe8r8q7j30um0arwjc.jpg) 

 

操作步骤：

很明显我们不会用到MQ等消息中间件，写一个队列为模拟消息中间件

a,新建JamesDefferdQueue消息队列类

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe2l2gzj30ul0augq3.jpg) 

 

 

b,在AsyncOrderController新增两方法（其实就是两个线程，线程1和线程N）

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe7b8jcj30um0f710h.jpg) 

 

测试结果为：

![img](http://ww4.sinaimg.cn/large/006tNc79ly1g4jpe23o7tj30px02lq3m.jpg) 

![img](http://ww3.sinaimg.cn/large/006tNc79ly1g4jpe00e6qj30pz03k3ze.jpg) 

通过create (tomcat线程N处理的订单结果)，异步返回给createOrder(tomcat线程1),两结果一致，异步返回了结果。

 

 

 

 

