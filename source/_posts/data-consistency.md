---
title: A系统给B系统转100块钱，如何实现？
date: 2019-02-12 00:21:55
tags: 数据一致性
category:
- 专题
- 数据一致性
---

## A系统给B系统转100块钱，如何实现？

> 数据如何保证一致性，性能优化（编程式事务），CAS锁

1. 数据保证一致性（CAS锁）幂等性

```
/**
 * 悲观锁
 * select * from xxx where id = xxx for update
 * 
 * 乐观锁
 * select version from xxx where id = xxx
 * update xxx set xxx = xxx where id = xxx and version = 'version'
 * 
 * 基于状态机的乐观锁
 * int i = update xxx set status = 4 where id=xxx and version =1
 * i==1
 */
@Override
public String updateByTemplateThread(Order order) {
    String orderid = order.getOrderid();
    logger.info("order =======================", order);
    // spring的编程式事务, 这是编程式事务的写法出现了很大的变形
    Boolean lockStatus = template
            .execute(new TransactionCallback<Boolean>() {
                @Override
                public Boolean doInTransaction(TransactionStatus status) {
                    Order orderProcess = new Order();
                    orderProcess.setOrderid(orderid);
                    orderProcess.setOrderstatus("4");// 订单处理中的状态
                    orderProcess.setVersion(1);
                    return 1 == orderDao.updateByVersion(orderProcess);
                }
            });
    logger.info("-----------------lockStatus====="+lockStatus);
    if (lockStatus) {
        OrderVO orderVo = new OrderVO();
        BeanUtils.copyProperties(order, orderVo);
        String flag = transService.trans(url+orderid);

        template.execute(new TransactionCallback<Object>() {
            @Override
            public Object doInTransaction(TransactionStatus status) {
                Order orderFinished = new Order();
                orderFinished.setOrderid(orderid);
                orderFinished.setVersion(1);
                if (null != flag && flag == "0") {
                    orderFinished.setOrderstatus("0");// 失败
                } else {
                    orderFinished.setOrderstatus("2");// 成功
                }
                orderDao.updateByVersion(orderFinished);
                return null;
            }
        });
    } else {
        logger.error("========lockfail============" + order.getOrderid());
    }
        return "0";
}

```

 
2. 性能优化 （推荐编程式事务），声明式事务一般不推荐

```
/**
 * 事务中包括远程调用，本地连接占用的时间会因为远程调用的时间而不可控，被消耗干净 导致其它乃至数据库的服务不可用
 * 
 * 连接解决后，被调用的服务可能会出现失败，需要引入出款请求模型
 * 
 * 使用TransactionTemplate，改善连接
 */
@Autowired
private TransactionTemplate template;

// 使用propagation 指定事务的传播行为，即当前的事务方法被另外一个事务方法调用时如何使用事务。
// NEVER表示在方法里没有事务，每一段的template.execute存在事务，调远程接口时没有事务，不占用连接
@Transactional(propagation = Propagation.NEVER)
public String updateByTemplate(Order order) {
    String orderid = order.getOrderid();
    logger.info("order =======================", order);
    // spring的编程式事务, 这是编程式事务的写法出现了很大的变形
    template.execute(new TransactionCallback<Object>() {
        @Override
        public Object doInTransaction(TransactionStatus status) {
            Order orderProcess = new Order();
            orderProcess.setOrderid(orderid);
            orderProcess.setOrderstatus("4");// 订单处理中的状态
            orderDao.update(orderProcess);
            return null;
        } 
    });

    OrderVO orderVo = new OrderVO();
    BeanUtils.copyProperties(order, orderVo);
    String flag = transService.trans(url+orderid);

    template.execute(new TransactionCallback<Object>() {
        @Override
        public Object doInTransaction(TransactionStatus status) {
            Order orderFinished = new Order();
            orderFinished.setOrderid(orderid);
            if (null != flag && flag == "0") {
                orderFinished.setOrderstatus("0");// 失败
            } else {
                orderFinished.setOrderstatus("2");// 成功
            }
            orderDao.update(orderFinished);
            return null;
        }
    });

    return "0";
}
```
 
 

