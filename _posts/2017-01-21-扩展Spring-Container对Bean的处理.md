---
layout: post
title:  "扩展Spring Container对Bean的处理"
date:   2017-01-21 23:46:58 +0800
categories: my2829
---

### Bean definition inheritance
- Bean的定义支持继承，给bean加上`parent`属性指向另一个bean，那么parent bean设置的属性它也有，如果它也设置了同样的属性，则会覆盖
- child bean 和 parent bean 所指向的 `class` 不一定有继承关系，但是要相互兼容，即**给parent bean设置的属性child bean指向的class也必须有**
- parent bean定义时，可以不指定`class`，这样的bean不会被实例化，仅作为模板用，需要指定`abstract="true"`属性
- parent bean如果指定了`class`，且`class`指向一个abstract类，那么必须设置`abstract="true"`属性，避免Spring container实例化它

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

### 扩展Container对Bean的处理
#### 扩展方式有三种
- 一、**实现BeanPostProcessor接口，在Bean创建之后修改Bean**
    - 实现方式：只需要配置一个实现了BeanPostProcessor接口的Bean，Spring Container在创建Bean之前会先扫描所有的BeanPostProcessor，并把他们注册到Container
    - BeanPostProcessor可以有多个，通过实现Ordered接口排序
    - 所有的实现了BeanPostProcessor接口的Bean，在其他Bean之前实例化，这样在实例化其他Bean的时候可以应用BeanPostProcessor，如果BeanPostProcessor本身引用了其他的Bean，那么被引用的Bean会在BeanPostProcessor之前实例化，于是他们不会被应用到BeanPostProcessor
    - Because AOP auto-proxying is implemented as a BeanPostProcessor itself, neither BeanPostProcessors nor the beans they reference directly are eligible for auto-proxying, and thus do not have aspects woven into them.

```java
public interface BeanPostProcessor {
    // 在Bean初始化之前调用
    // e.g. before InitializingBean::afterPropertiesSet
    Object postProcessBeforeInitialization(Object bean, String beanName);
    // 在Bean初始化之后调用
    // e.g. after InitializingBean::afterPropertiesSet
    Object postProcessAfterInitialization(Object bean, String beanName);
}
```

- 二、**实现BeanFactoryPostProcessor接口，在Bean创建之前修改BeanFactory，可以获取并修改所有Bean的配置（definition）**
    - 实现方式同BeanPostProcessor
    - PropertyPlaceholderConfigurer 即是实现了 BeanFactoryPostProcessor接口，在创建Bean之前，把Bean的属性配置里的placeholder替换为真实的值，e.g. `${db.password}` 会被替换为 `your_password`

```java
public interface BeanFactoryPostProcessor {
    /**
     * Modify the application context's internal bean factory after its standard
     * initialization. All bean definitions will have been loaded, but no beans
     * will have been instantiated yet. This allows for overriding or adding
     * properties even to eager-initializing beans.
     * @param beanFactory the bean factory used by the application context
     */
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory);
}
```

- **三、实现FactoryBean，自己实现Bean的创建过程**
    - 如果创建一个Bean的过程很复杂，用配置（BeanDefinition）无法实现，你可以实现一个FactoryBean，用它来创建Bean
    - 使用FactoryBean的方式也是定义一个实现了FactoryBean的bean，Spring Container 会自动检测出它是一个FactoryBean
    - 假设定义了一个FactoryBean，`<bean id="objWithComplicateInit" class="com.domain.TheComplicateObjFactory">`，调用`applicationContext.getBean("objWithComplicateInit")`时，Spring会调用`TheComplicateObjFactory`的`getObject`方法，把返回的Object作为Bean返回
    - 如果想要获取FactoryBean本身这个bean，需要调用`applicationContext.getBean("&objWithComplicateInit")`，注意`&`符号

```java
public interface FactoryBean<T> {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```

