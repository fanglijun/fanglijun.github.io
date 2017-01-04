---
layout: post
title:  "What is JMX?"
date:   2017-01-04 23:29:58 +0800
categories: my2829
---

### 一、JMX（Java Management Extensions）
你按照规范实现MBean（Standard MBean or Dynamic MBean or MXBean），JMX允许你通过远程的Client（JConsole or HTTP Client or SNMP client）来查看（查看属性值、方法定义）、操作（修改属性值、远程调用方法）你的Bean。

![](https://upload.wikimedia.org/wikipedia/en/d/db/Jmxarchitecture.png)

### 二、 Standard MBean 示例
#### 定义接口和Bean
```java
// 首先有个interface
// 你只能查看、操作这个interface里定义的属性和方法
// interface 只有一个要求：名字以MBean结尾
public interface HelloMBean {
    void setName(String name);
    String getName();
    void sayHello();
}

// 定义一个类实现HelloMBean接口
public class Hello implements HelloMBean {
    private String name;

    @Override
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String getName() {
        return this.name;
    }

    @Override
    public void sayHello() {
        System.out.println("Hello, " + this.name + "!");
    }

    // 可以有其他属性、方法，会被忽略
}
```

#### 创建Bean并注册到Server
```java
public static void main(String[] args) {
       // 获取MBeanServer
       MBeanServer mbs = ManagementFactory.getPlatformMBeanServer();

       // 创建一个名字，包含package和类名
       // 这将是你在JConsole里看到的package和类名
       ObjectName name = new ObjectName("com.whatever.domain:type=WhateverName");

       // 创建你的Bean
       Hello mbean = new Hello();

       // 注册Bean到MBeanServer
       mbs.registerMBean(mbean, name);

       // Wait forever
       System.out.println("Waiting forever...");
       Thread.sleep(Long.MAX_VALUE);
   }
```

#### 打开JConsole连接到Server
1. 启动上面的程序
2. 使用`jconsole`命令打开JConsole的GUI
3. 找到上面的程序，连接
4. .......

### 三、与Spring集成
继续用上面的Hello bean，在Spring配置文件里加入:
```
<bean id="testBean" class="tests.Hello">
    <property name="name" value="Xiaoming"/>
</bean>
<!-- this bean must not be lazily initialized if the exporting is to happen -->
<bean id="mBeanExporter" class="org.springframework.jmx.export.MBeanExporter" lazy-init="false">
    <property name="beans">
        <map>
            <entry key="com.whatever.domain:name=WhateverClass" value-ref="testBean"/>
        </map>
    </property>
</bean>
```
然后运行程序，打开JConsole，连接，就可以看到MBeans.



[JMX with Spring 参考](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/jmx.html)
