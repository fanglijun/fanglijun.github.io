---
layout: post
title:  "Spring Core - IoC Container"
date:   2017-01-18 23:56:58 +0800
categories: my2829
---

### ApplicationContext  => Spring's IoC Container
- read configuration metadata from:
    - XML配置
    - Java annotations
    - Java code
- instantiate, configure, and assemble the beans


### 使用XML配置
`beans.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
`App.java`
```java
ApplicationContext context =
    new ClassPathXmlApplicationContext("beans.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

### Bean overview
#### bean id/name and alias
- id 指定唯一的ID
- name 以逗号、分号、空格分隔，指定多个 alias
- `<alias name="fromName" alias="toName"/>` 给名字叫`fromName`的bean 加一个alia `toName`

#### bean 实例化
- 用 class 指定一个类e.g. SomeClass，相当于直接调用 new SomeClass()
- 如果是Inner Class，使用`$`分隔，e.g. `com.example.Foo$Bar`
- 使用静态工厂方法实例化：`<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>`，ClientService必须有个静态方法`createInstance`
- 使用对象的工厂方法：`<bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>` 其中`serviceLocator`是一个bean，他的`createClientServiceInstance`方法是一个工厂方法，用来创建其他的bean

