---
layout: post
title:  "Java Enum and Annotation"
date:   2017-01-11 23:33:58 +0800
categories: my2829
---

### Java Enum

```java
import java.util.EnumSet;

public enum PlanetEnum {
    // 这里是调用构造函数，定义所有枚举类型
    MERCURY (3.303e+23, 2.4397e6, "水星"),
    VENUS   (4.869e+24, 6.0518e6, "金星"),
    EARTH   (5.976e+24, 6.37814e6, "地球"),
    MARS    (6.421e+23, 3.3972e6, "火星"),
    JUPITER (1.9e+27,   7.1492e7, "木星"),
    SATURN  (5.688e+26, 6.0268e7, "土星"),
    URANUS  (8.686e+25, 2.5559e7, "天王星"),
    NEPTUNE (1.024e+26, 2.4746e7, "海王星");

    // Enum也是一个类，可以有属性、方法
    // 设置为final的属性，必须在构造函数里初始化
    private final double mass;   // in kilograms
    private final double radius; // in meters
    // 不设置为final的属性，可以为其定义setter
    private String name;

    PlanetEnum(double mass, double radius, String name) {
        this.mass = mass;
        this.radius = radius;
        this.name = name;
    }

    // 可以给属性定义 getter
    public String getName() {
        return this.name;
    }
    public double getMass() {
        return this.mass;
    }
    public double getRadius() {
        return this.radius;
    }

    // 可以给属性定义 setter，但是属性声明时不能用final
    public void setName(String name) {
        this.name = name;
    }

    public static PlanetEnum findByName(String name) {
        switch (name) {
            case "水星":
                return MERCURY;
            case "金星":
                return VENUS;
            case "地球":
                return EARTH;
            case "火星":
                return MARS;
            case "木星":
                return JUPITER;
            case "土星":
                return SATURN;
            case "天王星":
                return URANUS;
            case "海王星":
                return NEPTUNE;
            default:
                return null;
        }
    }

    public static void main(String[] args) {

        // 循环
        for (PlanetEnum p : PlanetEnum.values()) {
            System.out.printf(
                    "%s的重量是%fkg, 半径是%f米%n",
                    p.getName(), p.getMass(), p.getRadius());
        }
        System.out.println("=====================================");

        // 取子范围，按定义时的顺序(从水星到地球)
        for (PlanetEnum p: EnumSet.range(PlanetEnum.MERCURY, PlanetEnum.EARTH)) {
            System.out.println(p.getName());
        }
        System.out.println("=====================================");

        // 自定义子集（地球和火星）
        for (PlanetEnum p: EnumSet.of(PlanetEnum.EARTH, PlanetEnum.MARS)) {
            System.out.println(p.getName());
        }
        System.out.println("=====================================");

        // 修改枚举
        PlanetEnum earth = PlanetEnum.findByName("地球");
        earth.setName("地球已被人类毁灭");
        System.out.println("=====================================");

        // 再次获取枚举
        PlanetEnum planetEarth = PlanetEnum.findByName("地球");
        if (null == planetEarth) {
            System.out.println("还能找到地球吗？");
        } else {
            // 地球的名字？
            System.out.println(planetEarth.getName());
            // 还是原来的地球吗（用==表示两个变量指向的实际上是同一个实例）
            if (planetEarth == earth) {
                System.out.println("还是原来的地球");
            }
        }
    }
}
```

### Java Annotation

```java

import java.lang.annotation.*;
import java.lang.reflect.Method;

public class AnnotationTest {

    // Retention表示注解保留阶段，可以是：
    // RetentionPolicy.SOURCE or RetentionPolicy.CLASS or RetentionPolicy.RUNTIME
    @Retention(value = RetentionPolicy.RUNTIME)

    // Documented 表示这个注解的属性需要在javadoc生成的文档里列出来
    @Documented

    // Target 表示这个注解应该被用在什么地方
    // ElementType.TYPE 表示用在类型定义上
    // ElementType.METHOD 表示用在方法定义上
    // 当注解只有一个属性时(Target注解只有一个属性value)，可以省略属性名
    @Target({ElementType.TYPE, ElementType.CONSTRUCTOR})
    @interface ClassPreamble {
        String author();
        String date();
        int currentRevision() default 1;
        String lastModified() default "N/A";
        String lastModifiedBy() default "N/A";
        // 使用数组
        String[] reviewers();
    }

    // 使用注解
    @ClassPreamble(
            author = "authorName",
            date = "2017-01-11",
            currentRevision = 7,
            reviewers = {"name1", "name2"}
    )
    public static class SomeClass {

        @Deprecated
        public void sayHello(String name) {
            System.out.println("hello " + name);
        }
    }


    public static void main(String[] args) throws NoSuchMethodException {
        SomeClass someClass = new SomeClass();

        // 运行时获取注解，需要注解的Retention设置为 RetentionPolicy.RUNTIME

        // 获取类的注解
        ClassPreamble classPreamble = someClass.getClass().getAnnotation(ClassPreamble.class);
        if (null != classPreamble) {
            System.out.println(classPreamble.author());
        } else {
            System.out.println("SomeClass doesn't have ClassPreamble annotation");
            System.out.println(" or the ClassPreamble annotation is not retained at runtime");
        }

        // 获取方法注解
        Method sayHelloMethod = someClass.getClass().getMethod("sayHello", String.class);
        Annotation[] annotations = sayHelloMethod.getAnnotations();
        for (Annotation annotation : annotations) {
            if (annotation instanceof Deprecated) {
                System.out.println("Method sayHello is deprecated!");
            }
        }
    }
}
```
