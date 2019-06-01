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

---
