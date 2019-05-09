---
layout: post
title: Java Annotation
categories: Java
description: Java Annotation
keywords: Java, java, annotation, Annotation
---

自 Java5.0 版本引入注解之后极大方便了应用开发，本文整理注解的基础知识以及使用方法。

> Annotation是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符。它是一种由 JSR-175 标准选择用来描述元数据的一种工具。

## 作用

> - 生成文档，如 `javadoc` 规范中的 `@param` `@return` 等
> - 实现代码依赖 如 `spring` 的注解
> - 编译检查格式 如 `@Override` 等

## `JDK` 四个元注解

可在 `java.lang.annotation` 包中查看

> - @Target     注解用于什么地方，定义此 Annotation 所修饰的范围
> - @Retention  什么时候使用该注解，定义此 Annotation 应该被保留的时间长短
> - @Documented 注解是否将包含在JavaDoc中，定义此 Annotation 会将此注解包含在 javadoc 中
> - @Inherited  是否允许子类继承该注解

## 说明

详情参见 `jdk`，常用整理如下：

### `@Target`

> If an @Target meta-annotation is not present on an annotation type T , then an annotation of type T may be written as a modifier **for any declaration except a type parameter declaration.**

|取值|说明|
|----------|--------|
|`ElementType.TYPE`|Class, interface (including annotation type), or enum declaration|
|`ElementType.METHOD`|Method declaration|
|`ElementType.FIELD`|Field declaration (includes enum constants)|
|`ElementType.PARAMETER`|Formal parameter declaration|
|...|...|

### `@Retention`

|取值|说明|
|----------|--------|
|`RetentionPolicy.RUNTIME`|Annotations are to be recorded in the class file by the compiler and retained by the VM at run time, so they may be read reflectively.|
|`RetentionPolicy.CLASS`|Annotations are to be recorded in the class file by the compiler but need not be retained by the VM at run time.  **This is the default behavior**.|
|`RetentionPolicy.SOURCE`|Annotations are to be discarded by the compiler.|

### `@Documented` & `@Inherited`

## 解析注解

可以关注下 `java.lang.reflect` 包的 `AnnotatedElement` 接口。

```java
default boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {...}
<T extends Annotation> T getAnnotation(Class<T> annotationClass);
Annotation[] getAnnotations();
default <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass) {...}
Annotation[] getDeclaredAnnotations();
...
```

## 测试

### 自定义注解类

```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 自定义注解
 */
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyAnnotation {
    public String name() default "fieldName";
}
```

### 测试类

```java
@MyAnnotation(name = "sss")
public class Demo {

    @MyAnnotation
    private String str;

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    public static void main(String[] args) {
        if (Demo.class.isAnnotationPresent(MyAnnotation.class)) {
            MyAnnotation m = Demo.class.getAnnotation(MyAnnotation.class);
            System.out.println(m);
            System.out.println(m.name());
        }
    }
}
```
