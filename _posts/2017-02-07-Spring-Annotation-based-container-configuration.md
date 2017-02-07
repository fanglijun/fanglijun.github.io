---
layout: post
title:  "Spring Annotation-based container configuration"
date:   2017-02-07 23:56:58 +0800
categories: my2829
---


### 使用注解配置Bean
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

- [@Required](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-required-annotation)
    - 用在需要注入的属性的Setter方法上
    - 配置Bean时这个属性必须被设置

- [@Autowired](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-autowired-annotation)
    - 用在属性、setter、任意方法、构造函数上
    - 如果用在setter方法或任意方法上，这些方法会在创建bean时执行
    - autowire bean **`by Type`**

- [@Primary](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-autowired-annotation-primary)
    - 用在声明Bean时
    - 声明时带有`primary="true"`属性（或者有@Primary注解）的Bean优先被使用


- [@Qualifier](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-autowired-annotation-qualifiers)
    - 用来缩小`Autowire by Type`时的范围
    - 声明Bean时设置Qualifier `<qualifier value="main"/>`，注入时使用 `@Qualifier("main")`来匹配
    - Bean的name/id会被自动当做一个qualifier
    - [Generics 会被当成Qualifier](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-generics-as-qualifiers)

- [@Resource](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-resource-annotation)
    - 类似@Autowire用于属性、setter方法、任意方法，不可用于构造函数，`@Resource(name="myBeanNameOrId")`
    - autowire **`by Name`**
    - 如果未指定name，则把属性名字当做Bean Name，用于setter方法时从方法名字推断出属性名当做Bean name，用于任意方法时把参数名当做Bean name
    - 如果未找到对应名字的Bean，fallback to `by Type` match
    - 作用等于 @Autowire + @Qualifier("beanName")

- [@PostConstruct and @PreDestroy](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-postconstruct-and-predestroy-annotations)
    - 用于方法上
    - 分别表示初始化函数、析构函数

