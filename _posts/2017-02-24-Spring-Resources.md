---
layout: post
title:  "Spring Resources"
date:   2017-02-24 22:40:58 +0800
categories: my2829
---

### Resource interface

```java
public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isOpen();
    URL getURL() throws IOException;
    File getFile() throws IOException;
    Resource createRelative(String relativePath)
        throws IOException;
    String getFilename();
    String getDescription();
}

public interface InputStreamSource {
    InputStream getInputStream() throws IOException;
}
```

### Built-in Resource implementations

- UrlResource
    - e.g. 以`file:`, `http:`, `ftp:`开头
- ClassPathResource
    - 以 `classpath:` 开头
- FileSystemResource
    - This is a Resource implementation for java.io.File handles.
- ServletContextResource
    - This is a Resource implementation for ServletContext resources, interpreting relative paths within the relevant web application’s root directory.
- InputStreamResource
    - A Resource implementation for a given InputStream.
- ByteArrayResource
    - It creates a ByteArrayInputStream for the given byte array.


### ResourceLoader interface

```java
public interface ResourceLoader {
    Resource getResource(String location);
}
```

- 所有的ApplicationContext都实现了ResourceLoader，可以调用getResource方法获取一个Resource

#### ApplicationContext 对路径的解析

```java
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

对上面的代码：

- ClassPathXmlApplicationContext会把路径当做classpass来解析，返回一个ClassPathResource；
- FileSystemXmlApplicationContext会把路径当做文件路径解析，返回的是FileSystemResource；
- WebApplicationContext会返回ServletContextResource；

如果路径里指定了前缀（`file:`，`classpath:`，`http:`），则会按前缀指定的方式解析路径返回相应类型的Resource

### The ResourceLoaderAware interface

```java
// 实现此接口的Bean可在初始化时接收到ResourceLoader
public interface ResourceLoaderAware {
    void setResourceLoader(ResourceLoader resourceLoader);
}
```

- 也可以用`@Autowire`注解自动注入ResourceLoader，类似于注入当前ApplicationContext

### Resources as dependencies

```xml
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

- 如果myBean的template属性是一个Resource类型，则路径会被自动加载为一个Resource
- template的路径将会由ApplicationContext(同时是一个ResourceLoader)来加载，所以如果没指定前缀，则会受ApplicationContext的具体实现影响返回ClassPathResource或FileSystemResource等，可指定前缀强制路径解析方式

### FileSystemResource caveats

- A FileSystemResource that is not attached to a FileSystemApplicationContext (that is, a FileSystemApplicationContext is not the actual ResourceLoader) will treat absolute vs. relative paths as you would expect. Relative paths are relative to the current working directory, while absolute paths are relative to the root of the filesystem.

- For backwards compatibility (historical) reasons however, this changes when the FileSystemApplicationContext is the ResourceLoader. The FileSystemApplicationContext simply forces all attached FileSystemResource instances to treat all location paths as relative, whether they start with a leading slash or not.

- 所以如果想按照文件的绝对路径来加载Resource，最好是使用`file:`前缀强制获取一个UrlResource

```java
// actual context type doesn't matter,
// the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");

// force this FileSystemXmlApplicationContext
// to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```


