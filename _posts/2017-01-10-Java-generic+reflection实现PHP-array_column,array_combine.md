---
layout: post
title:  "Java generic+reflection实现PHP array_column,array_combine"
date:   2017-01-10 23:59:58 +0800
categories: my2829
---

### Generic + Reflection
```java
import org.apache.commons.lang3.reflect.FieldUtils;

import java.lang.reflect.InvocationTargetException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by fanglijun on 17/1/10.
 */
public class GenericReflection {

    // 同php array_column
    public static <T,V> List<V> listColumn(List<T> source, String columnName)
            throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        List<V> res = new ArrayList<>();
        if (source != null){
            for (T item: source){
                V temp = (V) FieldUtils.readField(item, columnName, true);
                if (temp != null)
                    res.add(temp);
            }
        }
        return  res;
    }

    // 类似php array_combine
    public static <T, V> Map<V, T> listCombine(List<T> list, String columnName)
            throws IllegalAccessException {
        Map<V, T> res = new HashMap<>();
        if (null != list) {
            for (T item : list) {
                V key = (V) FieldUtils.readField(item, columnName, true);
                if (null != key) {
                    res.put(key, item);
                }
            }
        }
        return res;
    }

    // 用多个key去Map里取值，返回第一个不为null的值
    // 如果都为null返回null
    public static <K, T> T tryGet(Map<K, T> map, K... keys) {
        T value = null;
        for (K key : keys) {
            value = map.get(key);
            if (null != value) {
                break;
            }
        }
        return value;
    }

    // 测试类
    public static class MyClass {
        public Integer id;
        public String name;
        private String mobile;

        public MyClass(Integer id, String name, String mobile) {
            this.id = id;
            this.name = name;
            this.setMobile(mobile);
        }

        public String getMobile() {
            return mobile;
        }

        public void setMobile(String mobile) {
            this.mobile = mobile;
        }
    }

    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        MyClass a1 = new MyClass(1, "xiaoming1", "13812345671");
        MyClass a2 = new MyClass(2, "xiaoming2", "13812345672");
        MyClass a3 = new MyClass(3, "xiaoming3", "13812345673");
        MyClass a4 = new MyClass(4, "xiaoming4", "13812345674");

        List<MyClass> myClasses = new ArrayList<>();
        myClasses.add(a1);
        myClasses.add(a2);
        myClasses.add(a3);
        myClasses.add(a4);

        // mobiles = ["13812345671", "13812345672", "13812345673", "13812345674"]
        List<String> mobiles = listColumn(myClasses, "mobile");

        // myClassMap = {
        //      "13812345671" => a1,
        //      "13812345672" => a2,
        //      "13812345673" => a3,
        //      "13812345674" => a4
        // }
        // Map里的顺序不保证
        Map<String, MyClass> myClassMap = listCombine(myClasses, "mobile");

        // 会取到 a4
        MyClass willBe_a4 = tryGet(myClassMap, "nonExistKey", "13812345674");

        System.out.println(mobiles);

        System.out.println(myClassMap);
    }
}
```

### Generic only
```java
public class Generic {

    // 同php array_column
    public static <T,V> List<V> listColumn(List<T> source,ColumnGetter<T,V> columnGetter){
        List<V> res = new ArrayList<>();
        if (source != null){
            for (T item: source){
                V temp = columnGetter.getColumn(item);
                if (temp != null)
                    res.add(temp);
            }
        }
        return  res;
    }

    // 类似php array_combine
    public static <T, V> Map<V, T> listCombine(List<T> list, ColumnGetter<T, V> columnGetter) {
        Map<V, T> res = new HashMap<>();
        if (null != list) {
            for (T item : list) {
                V key = columnGetter.getColumn(item);
                if (null != key) {
                    res.put(key, item);
                }
            }
        }
        return res;
    }

    // 用多个key去Map里取值，返回第一个不为null的值
    // 如果都为null返回null
    public static <K, T> T tryGet(Map<K, T> map, K... keys) {
        T value = null;
        for (K key : keys) {
            value = map.get(key);
            if (null != value) {
                break;
            }
        }
        return value;
    }

    // 用于listColumn和listCombine
    public interface ColumnGetter<T,V>{
        V getColumn(T v);
    }


    // 测试类
    public static class MyClass {
        public Integer id;
        public String name;
        public String mobile;

        public MyClass(Integer id, String name, String mobile) {
            this.id = id;
            this.name = name;
            this.mobile = mobile;
        }
    }

    public static void main(String[] args) {
        MyClass a1 = new MyClass(1, "xiaoming1", "13812345671");
        MyClass a2 = new MyClass(2, "xiaoming2", "13812345672");
        MyClass a3 = new MyClass(3, "xiaoming3", "13812345673");
        MyClass a4 = new MyClass(4, "xiaoming4", "13812345674");

        List<MyClass> myClasses = new ArrayList<>();
        myClasses.add(a1);
        myClasses.add(a2);
        myClasses.add(a3);
        myClasses.add(a4);

        // mobiles = ["13812345671", "13812345672", "13812345673", "13812345674"]
        List<String> mobiles = listColumn(myClasses, new ColumnGetter<MyClass, String>() {
            @Override
            public String getColumn(MyClass v) {
                return v.mobile;
            }
        });

        // myClassMap = {
        //      "13812345671" => a1,
        //      "13812345672" => a2,
        //      "13812345673" => a3,
        //      "13812345674" => a4
        // }
        // Map里的顺序不保证
        Map<String, MyClass> myClassMap = listCombine(myClasses, new ColumnGetter<MyClass, String>() {
            @Override
            public String getColumn(MyClass v) {
                return v.mobile;
            }
        });

        // 会取到 a4
        MyClass willBe_a4 = tryGet(myClassMap, "nonExistKey", "13812345674");

        System.out.println(mobiles);

        System.out.println(myClassMap);
    }
}
```
