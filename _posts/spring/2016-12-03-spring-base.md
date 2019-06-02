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

  分析源码 `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`，内部有一个 `singletonsCurrentlyInCreation` 集合保存了正在创建过程中的 bean 们。同时内部定义了 `singletonObjects`、`earlySingletonObjects`、`singletonFactories` 三个缓存（一二三级缓存）。

  - A 创建过程中需要 B，于是 A 将自己放到 singletonFactories，去实例化 B
  - B 首先从 singletonObjects 中获取 A
  - 找不到说明正在创建，继续从 earlySingletonObjects 中获取 A
  - 找不到就从 singletonFactories 里获取 A
    - 找到后则从 singletonFactories 中移除 A，放到 earlySingletonObjects 中
    - B 创建完成，将自己放入 singletonObjects.
  - A 创建时可直接从 singletonFactories 获取 B

  这样就解决了循环依赖。

## 使用的设计模式

其实 spring 将常用设计模式都实现了。这个问题实际就是在问 java 的设计模式们。

- 工厂，单例，代理
- 适配器
- 装饰者
- 观察者
- 策略
- 模板方法
- builder 模式
- 责任链

更多设计模式，可访问：[java设计模式整理](/2017/03/29/design_paterns) 查看。

## Spring AOP

参考官方文档：[Aspect Oriented Programming with Spring](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/htmlsingle/#aop)

面向切面编程就是将交叉业务逻辑封装成切面，利用 AOP 的功能将切面织入到主业务逻辑中。交叉业务逻辑是指通用的、与主业务逻辑无关的代码，如安全检查、事务、日志等，若不使用 AOP 则会出现交叉业务逻辑与主业务逻辑混合在一起，会使主业务逻辑变的混杂不清。

Spring 默认采用 JDK 动态代理机制实现 AOP，当 JDK 动态代理不可用时（代理类无接口）会使用 CGlib 机制。但 Spring 的 AOP 有一定的缺点，第一只能对方法进行切入，不能对接口，字段，静态代码块进行切入（切入接口的某个方法，则该接口下所有实现类的该方法将被切入）；第二同类中的互相调用方法将不会使用代理类。因为要使用代理类必须从 Spring 容器中获取 Bean。第三性能不是最好的。

主要概念和使用注解如下：

- Aspect
  - 切面，共有功能的实现，横切多个对象
- Join point
  - 拦截点，程序在运行过程中能够插入切面的地点。例如，方法调用、异常抛出或字段修改等。
- Advice
  - 切面的具体实现，以目标方法为参照点，根据放置的地方不同，可分为 5 种：
  - @Before
    - 前置增强：在目标方法执行前，增加执行逻辑。
  - @AfterReturning
    - 后置增强：在目标方法执行后，增加执行逻辑（若目标方法抛出异常，则不执行增强逻辑）。
  - @AfterThrowing
    - 后置增强：和 @AfterReturning 增强相对应，只会在目标方法抛出异常时执行。
  - @After
    - 后置增强：增强始终会被执行（不管目标方法是否抛出异常）。
  - @Around
    - 环绕增强：在目标方法执行前和执行后，增加执行逻辑。
- @Pointcut
  - 切点：简单理解就是一个匹配规则，与切点函数组合使用。
  - 一个 Pointcut 对应多个 Join point。
- Weaving
  - 将切面应用到目标对象从而创建一个新的代理对象的过程。
  - 这个过程可以发生在编译期、类装载期及运行期。

大致使用流程如下：

- 使用 @Aspect 注解一个 bean

  ```java
  @Component
  @Aspect
  @Order(0) //设置切面的优先级  0-2147483647 数字越小优先级越高
  public class MyAopBean{
  ```

- 指定切点

  ```java
  @Pointcut("execution(* com.demo.service..*.*(..))")
  public void myPointcut(){}
  ```

- 配置 advice

  ```java
  @Before("myPointcut()")
  public void doBeforeAdvice(JoinPoint joinPoint){
    // 具体的逻辑，比如安全检查、日志记录、耗时检测等等
  }
  ```

- spring 初始化完切面之后，IOC 容器就会为切面相匹配的 bean 创建代理。
- 当运行对应的切面业务运行时就会被织入对应逻辑


