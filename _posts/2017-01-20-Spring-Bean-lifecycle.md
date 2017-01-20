---
layout: post
title:  "Spring Bean lifecycle"
date:   2017-01-20 23:46:58 +0800
categories: my2829
---

### Bean Lifecycle callbacks

#### Initialization callbacks
- 在bean的属性被设置之后调用
- 三种设置方式，如果都设置了，按以下顺序执行（可能调多个方法，但每个方法最多调一次）
    1. 给一个方法加@PostConstruct注解
    2. 实现InitializingBean接口的afterPropertiesSet
    3. 自定义init方法
        - 单个bean `<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>`
        - 统一设置 `<beans default-init-method="init">`
        - 单个设置可以覆盖统一的设置

#### Destruction callbacks
- 当Container被destroy时调用
- 三种方式设置
    1. 给一个方法加 @PreDestroy 注解
    2. 实现DisposableBean接口的destroy方法
    3. 自定义destroy方法
        - 单个bean `destroy-method`属性
        - 统一设置`default-destroy-method`

### ApplicationContextAware and BeanNameAware
```java
public interface ApplicationContextAware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```
实现ApplicationContextAware接口的bean，他的setApplicationContext方法会在bean属性被设置后、Initialization callbacks调用前 被调用，可以让bean获取到当前执行的ApplicationContext

BeanNameAware 类似 ApplicationContextAware，可以让bean获取到xml配置里它的id（如果没设置id，就是name）

