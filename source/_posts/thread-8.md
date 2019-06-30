---
title: Java8新增的并发
date: 2019-06-20 23:21:00
tags:
- 多线程
category:
- 并发编程
- 多线程
---

## 原子操作CAS

### LongAdder

JDK1.8时，java.util.concurrent.atomic包中提供了一个新的原子类：LongAdder。
 根据Oracle官方文档的介绍，LongAdder在高并发的场景下会比它的前辈————AtomicLong 具有更好的性能，代价是消耗更多的内存空间。

**AtomicLong**是利用了底层的CAS操作来提供并发性的，调用了**Unsafe**类的**getAndAddLong**方法，该方法是个**native**方法，它的逻辑是采用自旋的方式不断更新目标值，直到更新成功。

在并发量较低的环境下，线程冲突的概率比较小，自旋的次数不会很多。但是，高并发环境下，N个线程同时进行自旋操作，会出现大量失败并不断自旋的情况，此时**AtomicLong**的自旋会成为瓶颈。

这就是**LongAdder**引入的初衷——解决高并发环境下**AtomicLong**的自旋瓶颈问题。

**AtomicLong**中有个内部变量**value**保存着实际的long值，所有的操作都是针对该变量进行。也就是说，高并发环境下，value变量其实是一个热点，也就是N个线程竞争一个热点。

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnenhvsrj309m00v3ya.jpg)

**LongAdder**的基本思路就是**分散热点**，将value值分散到一个数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样热点就被分散了，冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回。

这种做法和ConcurrentHashMap中的“分段锁”其实就是类似的思路。

**LongAdder**提供的API和**AtomicLong**比较接近，两者都能以原子的方式对long型变量进行增减。

但是**AtomicLong**提供的功能其实更丰富，尤其是**addAndGet**、**decrementAndGet**、**compareAndSet**这些方法。

**addAndGet**、**decrementAndGet**除了单纯的做自增自减外，还可以立即获取增减后的值，而**LongAdder**则需要做同步控制才能精确获取增减后的值。如果业务需求需要精确的控制计数，做计数比较，**AtomicLong**也更合适。

另外，从空间方面考虑，**LongAdder**其实是一种“空间换时间”的思想，从这一点来讲**AtomicLong**更适合。

总之，低并发、一般的业务场景下AtomicLong是足够了。如果并发量很多，存在大量写多读少的情况，那LongAdder可能更合适。适合的才是最好的，如果真出现了需要考虑到底用AtomicLong好还是LongAdder的业务场景，那么这样的讨论是没有意义的，因为这种情况下要么进行性能测试，以准确评估在当前业务场景下两者的性能，要么换个思路寻求其它解决方案。

对于**LongAdder**来说，内部有一个base变量，一个Cell[]数组。

base变量：非竞态条件下，直接累加到该变量上。

Cell[]数组：竞态条件下，累加个各个线程自己的槽Cell[i]中。

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnenf1v8j30bq00pa9u.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnencgzfj30aa00q3ya.jpg)

所以，最终结果的计算应该是

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jnf5lhhsj30fd0790sv.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnf5jahqj306201j743.jpg)

在实际运用的时候，只有从未出现过并发冲突的时候，base基数才会使用到，一旦出现了并发冲突，之后所有的操作都只针对Cell[]数组中的单元Cell。

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnf5g0c9j30lv06h3yw.jpg)

而LongAdder最终结果的求和，并没有使用全局锁，返回值不是绝对准确的，因为调用这个方法时还有其他线程可能正在进行计数累加，所以只能得到某个时刻的近似值，这也就是**LongAdder**并不能完全替代**LongAtomic**的原因之一。

而且从测试情况来看，线程数越多，并发操作数越大，LongAdder的优势越大，线程数较小时，AtomicLong的性能还超过了LongAdder。

### 其他新增

除了新引入LongAdder外，还有引入了它的三个兄弟类：**LongAccumulator** **、** **DoubleAdder** **、** **DoubleAccumulator**。

LongAccumulator是LongAdder的增强版。LongAdder只能针对数值的进行加减运算，而LongAccumulator提供了自定义的函数操作。

通过LongBinaryOperator，可以自定义对入参的任意操作，并返回结果（LongBinaryOperator接收2个long作为参数，并返回1个long）。

LongAccumulator内部原理和LongAdder几乎完全一样。

DoubleAdder和DoubleAccumulator用于操作double原始类型。

## StampLock

StampedLock是Java8引入的一种新的所机制,简单的理解,可以认为它是读写锁的一个改进版本,读写锁虽然分离了读和写的功能,使得读与读之间可以完全并发,但是读和写之间依然是冲突的,读锁会完全阻塞写锁,它使用的依然是悲观的锁策略.如果有大量的读线程,他也有可能引起写线程的饥饿。

而StampedLock则提供了一种乐观的读策略,这种乐观策略的锁非常类似于无锁的操作,使得乐观锁完全不会阻塞写线程。

它的思想是读写锁中读不仅不阻塞读，同时也不应该阻塞写。

**读不阻塞写的实现思路：**

在读的时候如果发生了写，则应当重读而不是在读的时候直接阻塞写！即读写之间不会阻塞对方，但是写和写之间还是阻塞的！

StampedLock的内部实现是基于CLH的。

参考代码，参见cn.enjoyedu.cha. StampedLockDemo

## CompleteableFuture

### Future的不足 

Future是Java 5添加的类，用来描述一个异步计算的结果。你可以使用isDone方法检查计算是否完成，或者使用get阻塞住调用线程，直到计算完成返回结果，你也可以使用cancel方法停止任务的执行。

虽然Future以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的CPU资源，而且也不能及时地得到计算结果，为什么不能用观察者设计模式当计算结果完成及时通知监听者呢？。

Java的一些框架，比如Netty，自己扩展了Java的 Future接口，提供了addListener等多个扩展方法，Google guava也提供了通用的扩展Future:ListenableFuture、SettableFuture 以及辅助类Futures等,方便异步编程。

同时Future接口很难直接表述多个Future 结果之间的依赖性。实际开发中，我们经常需要达成以下目的：

将两个异步计算合并为一个——这两个异步计算之间相互独立，同时第二个又依赖于第一个的结果。

等待 Future 集合中的所有任务都完成。

仅等待 Future集合中最快结束的任务完成（有可能因为它们试图通过不同的方式计算同一个值），并返回它的结果。

应对 Future 的完成事件（即当 Future 的完成事件发生时会收到通知，并能使用 Future 计算的结果进行下一步的操作，不只是简单地阻塞等待操作的结果）

### CompleteableFuture 

JDK1.8才新加入的一个实现类CompletableFuture，实现了Future<T>， CompletionStage<T>两个接口。实现了Future接口，意味着可以像以前一样通过阻塞或者轮询的方式获得结果。

#### 创建

除了直接new出一个CompletableFuture的实例，还可以通过工厂方法创建CompletableFuture的实例

**工厂方法：**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnfxjkocj30da01jt8l.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnfxgt95j30dy01g0sm.jpg)

Asynsc表示异步,而supplyAsync与runAsync不同在与前者异步返回一个结果,后者是void.第二个函数第二个参数表示是用我们自己创建的线程池,否则采用默认的ForkJoinPool.commonPool()作为它的线程池。

**获得结果的方法**

public T get()

public T get(long timeout, TimeUnit unit)

public T getNow(T valueIfAbsent)

public T join()

getNow有点特殊，如果结果已经计算完则返回结果或者抛出异常，否则返回给定的valueIfAbsent值。

join返回计算的结果或者抛出一个unchecked异常(CompletionException)，它和get对抛出的异常的处理有些细微的区别。

参见cn.enjoyedu.cha.cfdemo下CFDemo和JoinAndGet

**辅助方法**

public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)

public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)

allOf方法是当所有的CompletableFuture都执行完后执行计算。

anyOf方法是当任意一个CompletableFuture执行完后就会执行计算，计算的结果相同。

参见cn.enjoyedu.cha.cfdemo下AllofAnyOf

 

CompletionStage是一个接口，从命名上看得知是一个完成的阶段，它代表了一个特定的计算的阶段，可以同步或者异步的被完成。你可以把它看成一个计算流水线上的一个单元，并最终会产生一个最终结果，这意味着几个CompletionStage可以串联起来，一个完成的阶段可以触发下一阶段的执行，接着触发下一次，再接着触发下一次，……….。

总结CompletableFuture几个关键点：

1、计算可以由 Future ，Consumer 或者 Runnable 接口中的 apply，accept 或者 run等方法表示。

2、计算的执行主要有以下

a. 默认执行

b. 使用默认的CompletionStage的异步执行提供者异步执行。这些方法名使用someActionAsync这种格式表示。

c. 使用 Executor 提供者异步执行。这些方法同样也是someActionAsync这种格式，但是会增加一个Executor 参数。

CompletableFuture里大约有五十种方法，但是可以进行归类，

 

**变换类 thenApply：**

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jnfxe1cnj30cm0230so.jpg)

关键入参是函数式接口Function。它的入参是上一个阶段计算后的结果，返回值是经过转化后结果。

**消费类 thenAccept：**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jngmdny6j30cv021wef.jpg)

关键入参是函数式接口Consumer。它的入参是上一个阶段计算后的结果， 没有返回值。

 **执行操作类 thenRun：**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jngmasg1j30cr02ajrc.jpg)

对上一步的计算结果不关心，执行下一个操作，入参是一个Runnable的实例，表示上一步完成后执行的操作。

**结合转化类:**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jngm80mwj30pc02bmx7.jpg)

需要上一步的处理返回值，并且other代表的CompletionStage 有返回值之后，利用这两个返回值，进行转换后返回指定类型的值。

两个CompletionStage是并行执行的，它们之间并没有先后依赖顺序，other并不会等待先前的CompletableFuture执行完毕后再执行。

**结合转化类**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnhc7laaj30js0243yi.jpg)

对于Compose可以连接两个CompletableFuture，其内部处理逻辑是当第一个CompletableFuture处理没有完成时会合并成一个CompletableFuture,如果处理完成，第二个future会紧接上一个CompletableFuture进行处理。

第一个CompletableFuture 的处理结果是第二个future需要的输入参数。

**结合消费类:**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnhc534vj30og02cwej.jpg)

需要上一步的处理返回值，并且other代表的CompletionStage 有返回值之后，利用这两个返回值，进行消费

**运行后执行类：**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnhc2pauj30ju024gln.jpg)

不关心这两个CompletionStage的结果，只关心这两个CompletionStage都执行完毕，之后再进行操作（Runnable）。

**取最快转换类：**

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jnhuptjij30o902aq30.jpg)

两个CompletionStage，谁计算的快，我就用那个CompletionStage的结果进行下一步的转化操作。现实开发场景中，总会碰到有两种渠道完成同一个事情，所以就可以调用这个方法，找一个最快的结果进行处理。

**取最快消费类：**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnhumvxlj30mp025q2z.jpg)

两个CompletionStage，谁计算的快，我就用那个CompletionStage的结果进行下一步的消费操作。

**取最快运行后执行类：**

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnhuk0wfj30ju023dfv.jpg)

两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）。

**异常补偿类：**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnid9xe5j30em02v0so.jpg)

当运行时出现了异常，可以通过exceptionally进行补偿。

**运行后记录结果类：**

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnid3oewj30lv025jrf.jpg)

action执行完毕后它的结果返回原始的CompletableFuture的计算结果或者返回异常。所以不会对结果产生任何的作用。

**运行后处理结果类：**

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jnicviouj30l202274c.jpg)

运行完成时，对结果的处理。这里的完成时有两种情况，一种是正常执行，返回值。另外一种是遇到异常抛出造成程序的中断。

 

### 补充：Lambda速成

本补充章节仅为没接触过Lambda的同学快速入门和速查，更具体的Lamba的知识请自行查阅相关书籍和博客。相关代码放在cn.enjoyedu.cha.lambda下

现在我们有一个实体类，我们会对这个实体类进行操作。

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jnj3bu4hj308p05jweg.jpg)

#### 第一步

我们想从一批Circle中挑选出挑选出半径为2的圆，于是我们写了一个方法

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnj2znodj30ka06jjrn.jpg)

这样，无疑很不优雅，如果我们想挑选半径为3的圆，难道还要再写一个方法？于是我们考虑将选择条件进行参数化，比如根据颜色挑选出圆或者根据半径挑选出圆

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jnj2hj5sj30r306rdg7.jpg)

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnjmweeqj30q906q74n.jpg)

但是，这种实现，还是有问题的，1、选择条件变化了，那么相应的方法也要变，比如我们想挑选半径大于3的圆，怎么办？如果我要根据多个条件选择，怎么办？难道把所有的条件都传入吗？于是，我们考虑定义一个挑选圆的接口，程序进化到了第二歩

#### 第二步

进行行为参数化，定义一个接口

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnjms56gj30cs02st8n.jpg)

 在进行圆的挑选的方法里，我们把这个接口作为参数进行传递

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnjmo7pmj30tg07faaf.jpg)

然后，我们只要按业务需求实现接口，并传入实现类的实例即可

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnk41d42j30nb08wmxo.jpg)

这种方式可以提高灵活性，但是业务上每增加一个挑选行为， 我们就需要显式声明一个接口ChoiceCircle的实现类，于是我们可以考虑使用内部匿名类，进入第三步。

#### 第三步

在实际使用时，我们不再声明一个接口ChoiceCircle的实现类

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnk3vxubj30p509x0td.jpg)

匿名内部类占用代码空间较多，而且存在着模版代码，这种情况下，Lambda表达式就可以派上用场了

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnk3ssnwj30kq03j74c.jpg)

所以可以把Lambda表达式看成匿名内部类的一个简洁写法

#### Lambda

在语法上，Lambda表达式包含三个部分，参数列表，箭头，主体，比如：

 **(parameters) -> expression**

或

 **(parameters) ->** **｛** **statements;** **｝**

Lambda表达式用在函数式接口上，所谓函数式接口，是只定义了一个抽象方法的接口（Interface），接口中是否有默认方法，不影响。

注解@FunctionalInterface可以帮助我们在设计函数式接口时防止出错。

我们常用的Runnable,Callable都是函数式接口，JDK8中新增了几个函数式接口：

**Predicate<T> :**

包含test方法，接受泛型的T，返回boolean，可以视为断言（检查）接口

**Consumer<T> :**

包含accept方法，接受泛型的T，无返回，可以视为数据消费接口

**Function<T** **，** **R> :**

包含apply方法，接受泛型的T，返回R，可以视为映射转换接口

**Supplier<T>**

包含get方法，无输入，返回T，可以视为创建一个新对象接口

**UnaryOperator<T>**

扩展至Function<T，T>，所以这个本质上也是一个映射转换接口，只不过映射转换后的类型保持不变

**BiFunction<T, U, R>**

包含apply方法，接受泛型的T、U，返回R，可以视为复合型映射转换接口

**BinaryOperator<T>**

扩展至Function BiFunction<T,T,T>，所以这个本质上也是一个复合型映射转换接口，只不过映射转换后的类型保持不变

**BiPredicate** **<T, U>** 

包含test方法，接受泛型的T，U，返回boolean，可以视为复合型断言（检查）接口

**BiConsumer<T** **，** **U>:**

包含accept方法，接受泛型的T，U，无返回，可以视为复合型数据消费接口

同时还提供了一些为了防止自动装箱机制，而特意声明的原始类型特化的函数式接口，比如，

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnknlotrj30b307daa7.jpg)

在意义上，和对应的Predicate接口并没有差别。

#### 函数描述符

函数式接口的抽象方法的签名基本上就是Lambda表达式的签名。我们将这种抽象方法叫作函数描述符。

Runnable接口可以看作一个什么也不接受什么也不返回（void）的函数的签名，因为它只有一个叫作run的抽象方法，这个方法什么也不接受，什么也不返回（void）。

我们可以用 () -> void代表参数列表为空，且返回void的函数。这正是Runnable接口所代表的。我们于是可以称() -> void是Runnable接口的函数描述符。

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4jnknitraj30bd09h3yr.jpg)

再考察Callable接口和Supplier接口

![](http://ww2.sinaimg.cn/large/006tNc79ly1g4jnknfe13j30b805jmx8.jpg)

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jnmm0x5zj30ab05g748.jpg)

从函数描述符来看，Callable接口和Supplier接口是一样的，都是

() -> X

所以同一个Lambda可以同时用在这两个函数式接口上，比如：

Callable<Integer> = () -> 33;

Supplier<><Integer> = () -> 33;

#  扩充知识点- Disruptor

## 应用背景和介绍

Disruptor是英国外汇交易公司LMAX开发的一个高性能队列，研发的初衷是解决内部的内存队列的延迟问题，而不是分布式队列。基于Disruptor开发的系统单线程能支撑每秒600万订单，2010年在QCon演讲后，获得了业界关注。

据目前资料显示：应用Disruptor的知名项目有如下的一些：Storm, Camel, Log4j2,还有目前的美团点评技术团队也有很多不少的应用，或者说有一些借鉴了它的设计机制。 

Disruptor是一个高性能的线程间异步通信的框架，即在同一个JVM进程中的多线程间消息传递。

## 传统队列问题

在JDK中，Java内部的队列BlockQueue的各种实现，仔细分析可以得知，队列的底层数据结构一般分成三种：数组、链表和堆，堆这里是为了实现带有优先级特性的队列暂且不考虑。 

在稳定性和性能要求特别高的系统中，为了防止生产者速度过快，导致内存溢出，只能选择有界队列；同时，为了减少Java的垃圾回收对系统性能的影响，会尽量选择 Array格式的数据结构。这样筛选下来，符合条件的队列就只有ArrayBlockingQueue。但是ArrayBlockingQueue是通过**加锁**的方式保证线程安全，而且ArrayBlockingQueue还存在**伪共享**问题，这两个问题严重影响了性能。

ArrayBlockingQueue的这个伪共享问题存在于哪里呢，分析下核心的部分源码，其中最核心的三个成员变量为

![](http://ww4.sinaimg.cn/large/006tNc79ly1g4jnmm0x5zj30ab05g748.jpg) 是在ArrayBlockingQueue的核心enqueue和dequeue方法中经常会用到的，这三个变量很容易放到同一个缓存行中，进而产生伪共享问题。

## 高性能的原理

引入环形的数组结构：数组元素不会被回收，避免频繁的GC，

无锁的设计：采用CAS无锁方式，保证线程的安全性

属性填充：通过添加额外的无用信息，避免伪共享问题

环形数组结构是整个Disruptor的核心所在。 

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4jnmlutzjj30hi09swff.jpg)

首先因为是数组，所以要比链表快，而且根据我们对上面缓存行的解释知道，数组中的一个元素加载，相邻的数组元素也是会被预加载的，因此在这样的结构中，cpu无需时不时去主存加载数组中的下一个元素。而且，你可以为数组预先分配内存，使得数组对象一直存在（除非程序终止）。这就意味着不需要花大量的时间用于垃圾回收。此外，不像链表那样，需要为每一个添加到其上面的对象创造节点对象—对应的，当删除节点时，需要执行相应的内存清理操作。环形数组中的元素采用覆盖方式，避免了jvm的GC。 

其次结构作为环形，数组的大小为2的n次方，这样元素定位可以通过位运算效率会更高，这个跟一致性哈希中的环形策略有点像。在disruptor中，这个牛逼的环形结构就是RingBuffer，既然是数组，那么就有大小，而且这个大小必须是2的n次方

其实质只是一个普通的数组，只是当放置数据填充满队列（即到达2^n-1位置）之后，再填充数据，就会从0开始，覆盖之前的数据，于是就相当于一个环。

每个生产者首先通过CAS竞争获取可以写的空间，然后再进行慢慢往里放数据，如果正好这个时候消费者要消费数据，那么每个消费者都需要获取最大可消费的下标。

同时，Disruptor 不像传统的队列，分为一个队头指针和一个队尾指针，而是只有一个角标（上图的seq），它属于一个volatile变量，同时也是我们能够不用锁操作就能实现Disruptor的原因之一，而且通过缓存行补充，避免伪共享问题。该指针是通过一直自增的方式来获取下一个可写或者可读数据。
