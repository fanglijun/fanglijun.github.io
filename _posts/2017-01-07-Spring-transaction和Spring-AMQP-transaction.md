---
layout: post
title:  "Spring transaction 和 Spring AMQP transaction"
date:   2017-01-07 23:55:58 +0800
categories: my2829
---


### Spring transaction
![](https://mmbiz.qlogo.cn/mmbiz_png/SHQtibmBWibdxaHgqdaCjhgjZglyxwxBnj0CefOLiaMEJxloYGQjmc2H3BcZXTxtIEe1e1gyxLCs1kOAN94IsWKeg/0?wx_fmt=png)


- Spring通过AOP给需要事务的方法加了一个代理（Proxy），在调用方法前开启事务，调用方法之后commit或rollBack。
- 使用事务需要配一个PlatformTransactionManager的实现，比如DataSourceTransactionManager、HibernateTransactionManager
```java
// PlatformTransactionManager 接口
public interface PlatformTransactionManager {
    // 在执行有事务的方法之前，调用getTransaction，开启一个事务
    TransactionStatus getTransaction(TransactionDefinition var1) throws TransactionException;

    // 执行有事务的方法后，调用commit或rollback
    void commit(TransactionStatus var1) throws TransactionException;

    void rollback(TransactionStatus var1) throws TransactionException;
}
```
#### Transaction propagation
一个有事务的方法调用了另一个有事务的方法（事务出现嵌套），处理方式有三种：

- Required
    - 内层事务和外层事务分别得到一个逻辑上的事务，物理上还是一个事务
    - 内层事务如果回滚，外层事务再调用回滚会出现UnexpectedRollbackException，最终结果是，两个事务同时回滚或成功执行
- RequiresNew
    - 内层事务和外层事务分别得到一个物理上的事务
    - 两个事务完全独立，互不影响
- Nested
    - 内外层共享一个物理事务，但是有多个savepoints，所以可部分成功
    - 需要savepoints支持，only work with JDBC resource transactions.

默认是Required

### How Spring AMQP work with Spring Transaction

#### Spring 的Transaction天生就是可以被别人利用的
Spring框架提供了一个TransactionSynchronizationManager，它管理了一系列TransactionSynchronization；实现了TransactionSynchronization这个接口的类可以被注册到TransactionSynchronizationManager，当前事务状态改变时会调用TransactionSynchronization的相应方法
```java
public interface TransactionSynchronization extends Flushable {
    int STATUS_COMMITTED = 0;
    int STATUS_ROLLED_BACK = 1;
    int STATUS_UNKNOWN = 2;

    void suspend();

    void resume();

    void flush();

    void beforeCommit(boolean var1);

    void beforeCompletion();

    void afterCommit();

    void afterCompletion(int status);
}
```

#### Spring AMQP里的RabbitResourceSynchronization即实现了TransactionSynchronization，在afterCompletion()回调函数里根据status参数判断是commit还是rollback

