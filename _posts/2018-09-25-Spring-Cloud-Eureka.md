### Server

#### pom.xml
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  
</dependency>
```

#### RegistryApplication .java
```java
@EnableEurekaServer  
@SpringBootApplication  
public class RegistryApplication {
   public static void main(String[] args) {
      SpringApplication.run(RegistryApplication.class, args);
  }
}
```

#### application.properties
```properties
server.port=2002  

# 为了避免server把自己当成client来注册
eureka.client.register-with-eureka=false  
eureka.client.fetch-registry=false
```

### Client

#### pom.xml
```xml
<dependency>  
 <groupId>org.springframework.cloud</groupId>  
 <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>

<!-- Feign Client -->
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-openfeign</artifactId>  
</dependency>
```

#### UserApplication.java
```java
@EnableDiscoveryClient
@EnableFeignClients
@SpringBootApplication
public class UserApplication {
   public static void main(String[] args) {
      SpringApplication.run(UserApplication.class, args);
  }
}
```

#### application.properties
```properties
server.port=2003  
server.servlet.path=/api/user

eureka.client.register-with-eureka=true  
eureka.client.fetch-registry=true  
eureka.client.service-url.defaultZone=http://localhost:2002/eureka
```

#### UserClient.java
```java
@FeignClient("user")  
public interface UserClient {  
  
    @RequestMapping(method = RequestMethod.GET, value = "/api/user/test/echo")  
    Map<String, String[]> echo(@RequestParam("s") String s);  
  
    @RequestMapping(method = RequestMethod.GET, value = "/api/user/test/service-instances/{applicationName}")  
    List<Object> serviceInstancesByApplicationName(@PathVariable("applicationName") String applicationName);  
}
```

#### TestController.java
```java
@RestController  
@RequestMapping(path = "test")  
public class TestController {  
  
    @Resource  
    private UserClient userClient;  
  
    @RequestMapping(path = "/", method = RequestMethod.GET)  
    public Object test() {  
        return userClient.serviceInstancesByApplicationName("user");  
    }  
}
```
