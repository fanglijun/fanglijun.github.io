---
layout: post
title:  "Spring Java-based container configuration"
date:   2017-02-20 23:43:58 +0800
categories: my2829
---

### container configuration

- Spring里Bean的配置可以完全使用Java代码，而不使用XML配置
- 也可以使用Java代码和xml结合的方式

### 使用Java代码配置

```java
// 使用@Configuration注解的类被当做配置类
@Configuration
public class AppConfig {

    // 使用@Bean注解的方法被当做一个bean，name是方法名
    // @Bean也可以用在使用@Component注解的类中，
    // 也可以声明bean，但是无法定义bean之间的依赖
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

public static void main(String[] args) {
    // AnnotationConfigApplicationContext构造函数可以接受多个参数
    // 每个参数可以是一个@Configuration注解的配置类
    // 也可以是一个@Component注解的Bean
    // 即，可以完全不使用@Configuration注解，
    // 直接把所有的bean交给AnnotationConfigApplicationContext
    ApplicationContext ctx =
        new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}

// 也可以使用逐个register
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx =
        new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#### Component Scan

- 使用@ComponentScan注解
```java
@Configuration
// 所有com.acme package下的Bean
// 都会被扫描并注册到spring container
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

- 使用scan方法

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx =
        new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
// @Configuration本身被@Component注解(meta-annotation)
// 所以配置类也是一个Bean，也会被扫描到
// 所以它及里面的@Bean都会被注册
```

#### Bean之间的依赖

```java
@Configuration
public class AppConfig {
    // 类似constructor-based dependency injection
    @Bean
    public TransferService transferService(
        AccountRepository accountRepository)
    {
        return new TransferServiceImpl(accountRepository);
    }
}
```

#### Receiving lifecycle callbacks

```java
public class Foo {
    public void init() {
        // initialization logic
    }
}

public class Bar {
    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {
    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }
    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}

// 默认情况下有close或shutdown方法的Bean，
// 它的close或shutdown方法会被自动调用，在container关闭时
// 可以使用 @Bean(destroyMethod="") 避免这种inferred mode
```

#### 其他bean配置

```java
@Configuration
public class MyConfiguration {
    // bean name
    @Bean(name = "beanName")
    // bean alias
    // @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    // bean description
    @Description("Provides a basic example of a bean")
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

#### 使用 @Import 组合多个配置类

```java
@Configuration
public class ConfigA {
    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {
    @Bean
    public B b() {
        return new B();
    }
}
```

- 组合后的配置类里，bean之间的相互依赖可以使用函数的参数注入的方式
- 也可以使用@Autowired把其他配置类注入为属性，然后调用它的方法，因为配置类本身也是一个bean

#### Conditionally include @Configuration classes or @Bean methods

- [http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-java-conditional](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-java-conditional)

### Java配置类与xml配置的结合

- 以xml为中心，引入Java配置类

```xml
<beans>
    <!--开启注解配置允许@Autowired和@Configuration-->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <!-- 引入Java配置类 -->
    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

- 如果已经开启 component scan，则会自动开启`<context:annotation-config/>`

```xml
<beans>
    <!--会自动开启annotation-config-->
    <!--并且如果配置类在com.acme包下，也会被自动扫描并解析-->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

- 以Java配置类为中心，引入xml配置，使用@ImportResource注解

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {
    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

```xml
properties-config.xml
<beans>
    <context:property-placeholder
        location="classpath:/com/acme/jdbc.properties"/>
</beans>
```


