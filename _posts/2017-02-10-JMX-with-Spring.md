---
layout: post
title:  "JMX with Spring"
date:   2017-02-10 23:56:58 +0800
categories: my2829
---

### 非侵入式手动配置

这种方式配置的MBean可以是任何Spring bean，它的所有public方法、属性都会被暴露给JMX。

```java
@Service
public class ConverterService {
    // ...
}
```

```xml
<bean id="mBeanExporter" class="org.springframework.jmx.export.MBeanExporter">
    <property name="beans">
        <map>
            <entry key="bean:name=testBean1" value-ref="converterService"/>
        </map>
    </property>
</bean>
```

### 使用注解配置

```xml
<context:mbean-export/>
```

- 使用`@ManagedResource`、 `@ManagedAttribute`和`@ManagedOperation`分别加在需要export的bean和它的属性、方法上
- 每个需要暴露的属性和方法都需要加注解


### JConsole远程连接配置

服务端JVM 启动时指定参数

- `-Dcom.sun.management.jmxremote.port=10017 `
- `-Dcom.sun.management.jmxremote.authenticate=false `
- `-Dcom.sun.management.jmxremote.ssl=false`

JConsole远程连接 `localhost:10017`


### 参考

[JMX with Spring](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/jmx.html)


