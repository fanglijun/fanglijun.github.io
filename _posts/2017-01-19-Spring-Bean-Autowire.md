---
layout: post
title:  "Spring Bean Autowire"
date:   2017-01-19 23:46:58 +0800
categories: my2829
---

### Bean Autowire

- 正常一个Bean需要引用其他Bean（作为构造函数的参数，或者作为属性）时，通过`constructor-arg`或`property`标签的 `ref` 属性指向目标Bean
- Autowire特性可以省掉手动指定目标Bean这一步，当给一个bean 设置`autowire`属性时，Spring会在当前container中查找符合条件的bean，自动注入进来
- `autowire` 属性有四种值：
    - **no**，(Default) No autowiring. Bean references must be defined via a ref element. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. To some extent, it documents the structure of a system.
    - **byName**，Autowiring by property name. Spring looks for a bean with the same name as the property that needs to be autowired. For example, if a bean definition is set to autowire by name, and it contains a master property (that is, it has a setMaster(..) method), Spring looks for a bean definition named master, and uses it to set the property.
    - **byType**，Allows a property to be autowired if exactly one bean of the property type exists in the container. If more than one exists, a fatal exception is thrown, which indicates that you may not use byType autowiring for that bean. If there are no matching beans, nothing happens; the property is not set.
    - **constructor**，Analogous to byType, but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised.

#### autowire时排除bean
- 如果不想让一个bean被autowire给其他的bean（设置了autowire属性的bean），可以给他设置`autowire-candidate=false`属性
- 也可以给beans标签设置`default-autowire-candidates`属性，指出哪些bean可以作为其他设置了autowire属性的bean的目标，e.g. 给beans标签设置`default-autowire-candidates=*Repository`，则只有名字以Repository结尾的bean才可以作为其他bean autowire 的目标；给单个bean设置`autowire-candidate=true|false`可以覆盖beans的`default-autowire-candidates`规则

