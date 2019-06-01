---
layout: post
title: Spring 简介
categories: Spring
description: Spring 简介
keywords: Spring, java
---

整理 Spring 框架原理以及常见使用问题。

## 框架组成结构

![image](/images/posts/springmodules.png)

## 容器类型

- BeanFactory
  - 基础容器
  - 默认采用延迟加载
  - 启动快
- ApplicationContext
  - 高级容器，继承 `ListableBeanFactory` 接口
  - 提供各种高级特性
    - 通用方式加载资源 ResourceLoader
    - 事件发布 ApplicationEventPublisher
    - 国际化信息支持 MessageSource
    - 继承实现的上下文环境隔离

## Bean 的生命周期

- 实例化 bean 对象
- 填充属性，注入依赖
- 调用 xxxAware 接口
  - 调用 `BeanNameAware#setBeanName(String name)`
  - 调用 `BeanFactoryAware#setBeanDactory(BeanFactory beanFactory)`
  - 调用 `ApplicationContextAware()#setApplicationContext(ApplicationContext ctx)`
    - 将上下文环境传入 bean
- 调用 `BeanPostProcessor#postProcessBeforeInitialization(Object bean, String beanName)`
  - 创建成功后进行**前置增强处理**，如修改 bean，增加某些方法
- 调用 `@PostConstruct` 标注的方法
- 调用 `InitializingBean#afterPropertiesSet()`
  - 对应 `init-method`，全部属性设置成功后调用此方法，执行自定义初始化流程
- 调用 `BeanPostProcess#postProcessAfterInitialization(Object bean, String beanName)`
  - 类似第6步，只是在全部初始化完成后
- working...
- 调用 `@PreDestroy` 标注的方法
- 调用 `DisposableBean#destroy()`
  - 对应 `destroy-method`，销毁 bean

大意如图所示：

![image](/images/posts/spring_bean_lifecycle.png)

以上就是主要的 bean 生命周期，实际上还有很多的细节没有列出，比如 BeanClassLoaderAware、EnvironmentAware、EmbeddedValueResolverAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ServletContextAware、DestructionAwareBeanPostProcessor 等等特别多的过程，某种程度上实现了 spring 框架的良好可拓展性。

### 循环依赖解决

spring 源码中搜索 `circular references`.

- 循环依赖的检测

  初始化 bean 时设置一个标志位，在系统加载过程中发现此对象被标记为创建中，则说明是循环依赖。

- 循环依赖的解决

  循环依赖的解决基于引用传递，也就是初始化之后，属性设置可以延后。因此：

  属性注入的循环依赖可以解决，构造器注入的循环依赖不可以解决，即构造器初始化不可延后，因为无法创建对象。prototype 类型的循环依赖无法解决。

  分析源码 `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`，内部有一个 `singletonsCurrentlyInCreation` 集合保存了正在创建过程中的 bean 们。同时内部定义了 `singletonObjects`、`earlySingletonObjects`、`singletonFactories` 三个缓存。

  - A 创建过程中需要 B，于是 A 将自己放到 singletonFactories，去实例化 B
  - B 首先从 singletonObjects 中获取 A
  - 找不到说明正在创建，继续从 earlySingletonObjects 中获取 A
  - 找不到就从 singletonFactories 里获取 A
    - 找到后则从 singletonFactories 中移除 A，放到 earlySingletonObjects 中
    - B 创建完成，将自己放入 singletonObjects.
  - A 创建时可直接从 singletonFactories 获取 B

  这样就解决了循环依赖。

singletonFactories、earlySingletonObjects、singletonFactories 又被称为一二三级缓存。
