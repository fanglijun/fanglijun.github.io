---
layout: post
title:  "Spring Classpath scanning and managed components"
date:   2017-02-08 23:46:58 +0800
categories: my2829
---

### Enable component scan

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example,com.domain"/>

</beans>
```

> The use of `<context:component-scan>` implicitly enables the functionality of `<context:annotation-config>`. There is usually no need to include the `<context:annotation-config>` element when using `<context:component-scan>`.


### Using filters to customize scanning

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

#### Filter Types

- **annotation (default)**
    - `org.example.SomeAnnotation`
    - An annotation to be present at the type level in target components.
- **assignable**
    - `org.example.SomeClass`
    - A class (or interface) that the target components are assignable to (extend/implement).
- **aspectj**
    - `org.example..*Service+`
    - An AspectJ type expression to be matched by the target components.
- **regex**
    - `org\.example\.Default.*`
    - A regex expression to be matched by the target components class names.
- **custom**
    - `org.example.MyTypeFilter`
    - A custom implementation of the `org.springframework.core.type.TypeFilter` interface.

### Naming autodetected components
```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```
#### 使用NameGenerator （implements BeanNameGenerator）

```xml
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

### Providing a scope for autodetected components

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

#### 使用Scope resolver (implements ScopeMetadataResolver)

```xml
<beans>
    <context:component-scan base-package="org.example"
            scope-resolver="org.example.MyScopeResolver" />
</beans>
```
