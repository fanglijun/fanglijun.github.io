---
layout: post
title:  "Spring AMQP使用"
date:   2017-01-06 23:53:58 +0800
categories: my2829
---

### 安装
```xml
<!--貌似需要Spring 4.*版本，不然报这个类未找到 org.springframework.util.backoff-->
<!--Sprint AMQP-->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>1.6.6.RELEASE</version>
</dependency>
```

### 发消息
```xml
<!--默认使用org.springframework.amqp.rabbit.connection.CachingConnectionFactory，可以缓存channel连接-->
<rabbit:connection-factory
        id="rabbitConnectionFactory"
        host="${rabbitmq.host}"
        port="${rabbitmq.port}"
        virtual-host="/"
        username="${rabbitmq.username}" password="${rabbitmq.password}"/>

<!--rabbitTemplate用于同步操作（同步收发消息等）-->
<!--channel-transacted="true"使channel具备事务支持-->
<rabbit:template
        id="rabbitTemplate"
        connection-factory="rabbitConnectionFactory"
        exchange="${rabbitmq.exchange.default}"
        message-converter="jsonMessageConverter"
        channel-transacted="true"
/>

```

```java
// inject bean
@Resource
RabbitTemplate rabbitTemplate;

@Transactional
public void run() {
    rabbitTemplate.convertAndSend("routing.key", anyObject);
    // 如果这里出异常，会自动rollBack
    doYourBusiness();
    // 手动触发回滚
    TransactionInterceptor.currentTransactionStatus().setRollbackOnly();
}
```

### 消费消息
```xml
<!--container是个Listener（相当于消费者）容器,可设置自己的线程池-->
<rabbit:listener-container
            connection-factory="rabbitConnectionFactory"
            message-converter="jsonMessageConverter"
            concurrency="10"
            max-concurrency="10"
            prefetch="1"
            requeue-rejected="false"
            acknowledge="auto"
    >
        <rabbit:listener
                ref="loginTaskListener"
                method="onMessage"
                queue-names="${rabbitmq.queue.spider.resume.login}"/>
        <rabbit:listener
                ref="crawlTaskListener"
                method="onMessage"
                queue-names="${rabbitmq.queue.spider.resume.login}"/>
    </rabbit:listener-container>

    <bean id="loginTaskListener" class="com.qjyd.xrxs.listener.LoginTaskListener"/>
    <bean id="crawlTaskListener" class="com.qjyd.xrxs.listener.CrawlTaskListener"/>

<!--这个用于序列化消息-->
    <bean id="jsonMessageConverter" class="org.springframework.amqp.support.converter.Jackson2JsonMessageConverter"/>

```
```java
public class LoginTaskListener {
    // 这里消息题会被自动转为指定的Object
    // 如果消息格式错误，无法转为TaskMessageDO，则会被reject
    public void onMessage(TaskMessageDO taskMessageDO) {
        LOGGER.debug(taskMessageDO.getType());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("message done");

        // 抛异常消息会被自动reject
        throw new RuntimeException("test");
    }
}
```

