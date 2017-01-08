---
layout: post
title:  "Spring Cloud Config 统一配置管理中心"
date:   2017-01-08 23:53:58 +0800
categories: my2829
---

### Intro
Spring Cloud Config provides server and client-side support for externalized configuration in a distributed system. With the Config Server you have a central place to manage external properties for applications across all environments.
统一配置管理中心，可管理多个Application的多个Profile的多个版本(Label)的配置。

### Server搭建
#### 配置文件的代码仓库
新建一个git repository，用于存放配置文件，假设路径是 /Users/abc/config/ （也可以是远程git仓库）
![](https://mmbiz.qlogo.cn/mmbiz_png/SHQtibmBWibdyZvN1hSc4B1IIDYrZ99WnGB6DCFTdr0qUJxO5qIJHPJtG0N8nfCgvJpdSr0M8ooBSkNyjODvd9jA/0?wx_fmt=png)

#### pom.xml
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
#### application.properties
```
server.port: 8888
spring.cloud.config.server.git.uri: /Users/abc/config/
```
#### App.java
```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            ConfigServiceApplication.class, args
        );
    }
}
```

### Client 配置
#### pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
#### bootstrap.properties
```
spring.application.name=app1
spring.cloud.config.uri=http://localhost:8888
spring.profiles.active=dev
```
#### App.java
```
@SpringBootApplication
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}


@RefreshScope
@RestController
class MessageRestController {

    // 这里会注入配置中的值
    @Value("${db.driver:defaultValue}")
    private String message;

    @RequestMapping("/message")
    String getMessage() {
        return this.message;
    }
}
```

#### Client不重启更新配置的方法
```
给需要更新的bean加上@RefreshScope注解，然后发送POST请求
curl -X POST http://localhost:8080/refresh
POST 请求会返回所有有变动的配置，但是只有加了注解的bean的属性才会更新（配置已取到最新的，但是不会更新到bean注入的属性里）
```


### 参考
https://spring.io/guides/gs/centralized-configuration/

http://cloud.spring.io/spring-cloud-config/spring-cloud-config.html

