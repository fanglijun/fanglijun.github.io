---
layout: post
title:  "Spring Profiles and Properties"
date:   2017-02-21 23:43:58 +0800
categories: my2829
---

### Profiles

- A profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active.

#### 给Bean设置profile

- 使用注解，e.g. `@Profile("dev")` 、`@Profile("production")`
- 自定义注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}

@Configuration
@Production
public class AppConfig {

    @Bean
    public DataSource dataSource() throws Exception {
        ...
    }
}
```

- 使用XML配置

```xml
<beans ...>

    <!-- other bean definitions -->

    <beans profile="dev">
        <bean id="dataSource">
            ...
        </bean>
    </beans>

    <beans profile="production">
        <bean id="dataSource" .../>
    </beans>
</beans>
```

#### 激活某个Profile

- 使用代码

```java
AnnotationConfigApplicationContext ctx =
    new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("dev");
// ctx.getEnvironment().
//  setActiveProfiles("profile1", "profile2");
ctx.register(AppProductionConfig.class, AppDevConfig.class);
ctx.refresh();
```

- 使用JVM启动参数

```
-Dspring.profiles.active="profile1,profile2"
```

### Properties

#### 读取properties

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("'foo' property exists? " + containsFoo);
```

#### Properties读取优先级 （越往下优先级越低，被上面的覆盖）

- ServletConfig parameters (if applicable, e.g. in case of a DispatcherServlet context)
- ServletContext parameters (web.xml context-param entries)
- JNDI environment variables ("java:comp/env/" entries)
- JVM system properties ("-D" command-line arguments)
- JVM system environment (operating system environment variables)

#### 添加自己的PropertySource实现

```java
ConfigurableApplicationContext ctx =
    new GenericApplicationContext();
MutablePropertySources sources =
    ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

#### 使用@PropertySource注解从文件加载Properties

```java
@Configuration
// 路径中的placeholder会从之前加载的properties里读取
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {
    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

#### XML里的配置

```xml
<context:property-placeholder location="classpath:database.properties" ignore-unresolvable="true"/>

<bean id="dataSource">
    <property name="jdbcUrl" value="${db.url}"/>
    <property name="user" value="${db.username}"/>
    <property name="password" value="${db.password}"/>
</bean>
```
