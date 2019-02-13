---
title: 你是怎么控制缓存的更新？（被动方式主动方式增量全量）？
date: 2019-02-13 22:24:51
tags: 
- 缓存
- 数据一致性
category:
- 专题
- 数据一致性
---

> 缓存信息的本质就是硬盘数据的副本，归根究底是一种用空间换时间的技术，数据一致性问题是不可避免的！

## 缓存在系统部署中的位置

![img](/images/refresh-policy-1.png)

## 解决缓存数据一致性问题方案

![img](/images/refresh-policy-2.png)

### 数据实时同步失效 （增量/主动）

- code 实例：

```
// 锁对象集，粒度到key值
private final ConcurrentHashMap<Object, ReentrantLock> locks = new ConcurrentHashMap<>();

private Lock getLockForKey(Object key) {
    ReentrantLock lock = new ReentrantLock();// 创建锁
    ReentrantLock previous = locks.putIfAbsent(key, lock);// 把新锁添加到locks集合中，如果添加成功使用新锁，如果添加失败则使用locks集合中的锁
    return previous == null ? lock : previous;
}

@Override
// 优先从缓存中加载收益总金额
public BigDecimal getProfitAmountByCache(String userCode) {
    // 1.从缓存中加载数据
    BigDecimal cacheResult = cs.cacheResult(userCode, CACHE_NAME);
    // 2.如果缓存中有数据直接返回给调用端
    if (cacheResult != null) {
        logger.info("======================get data from cache============================");
        return cacheResult;
    }

    BigDecimal profitAmount = null;
    acquireLock(userCode);//竞争锁
    try {
        cacheResult = cs.cacheResult(userCode, CACHE_NAME);
        // 2.如果缓存中有数据直接返回给调用端
        if (cacheResult != null) {
            logger.info("======================get data from cache============================");
            return cacheResult;
        }

        // 3.如果缓存中没有数据从数据库中加载数据
        logger.info("======================get data from db============================");
        profitAmount = mapper.getProfitAmount(userCode);
        // 4.从数据库查询的结果不为空，则把数据放入缓存中，方便下次查询
        if (profitAmount != null) {
            cs.cachePut(userCode, profitAmount, CACHE_NAME);
        }

    } catch (Exception e) {
        // TODO: handle exception
    } finally {
        releaseLock(userCode);//释放锁
    }

    return profitAmount;
}

public void releaseLock(String userCode) {
    ReentrantLock lock = (ReentrantLock) getLockForKey(userCode);//获得锁对象
    if(lock.isHeldByCurrentThread()){//判断锁对象是否由当前线程持有
        lock.unlock();
    }
}

public void acquireLock(String userCode) {
    Lock lock = getLockForKey(userCode);//获得锁对象
    lock.lock();
}

@Override
@Transactional
public ProfitDetail addProfitDetail(ProfitDetail detail) {
    mapper.insert(detail);//1.更新数据库
    cs.cacheRemove(detail.getUsercode(), CACHE_NAME);//2.淘汰缓存
    return detail;
}

```

### 数据准实时更新 （增量/被动）

![img](/images/refresh-policy-3.png)

- code 实例：

```
@Resource
private JmsTemplate jt;

@Override
public ProfitDetail addProfitDetail(final ProfitDetail detail) {
    mapper.insert(detail);//1.更新数据库
    jt.send(new MessageCreator() {
    
    @Override
    public Message createMessage(Session session) throws JMSException {
        return session.createTextMessage(detail.getUsercode());
    }
    });
    
    return detail;
}

@Service
public class CacheMesgListener implements MessageListener {

    @Resource
    private CacheService cs;
    
    @Resource
    private ProfitDetailMapper mapper;
    
    private static final String CACHE_NAME = "amount";

    @Override
    public void onMessage(Message message) {
        TextMessage tm = (TextMessage) message;
        try {
            String userCode = tm.getText();
            BigDecimal profitAmount = mapper.getProfitAmount(userCode);
            cs.cachePut(userCode, profitAmount,CACHE_NAME);
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}

```

### 任务调度更新 （全量/被动）

- ETL 工具应用（略）





