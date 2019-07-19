---
layout: post
title: 微服务设计与实践原则
categories: SpringCloud
description: 微服务设计与实践原则
keywords: web, micro, design, service
---

整理微服务设计与实践历程，共享给大家。

## 微服务的描述

The description of microserivce by *Martin Fowler* :

- 根据业务模块划分服务种类。
- 每个服务可以独立部署并且互相隔离。
- 通过轻量的 API 调用服务。
- 服务需要保证良好的高可用性。

> 微服务架构是以专注与单一责任的小功能模块为基础、通过 API 相互通信的方式完成复杂业务系统搭建的一种设计思想。

## 演变过程

单体架构(Monolithic) -> 垂直架构 -> SOA 架构 -> 微服务架构

待解决的痛点：

DAO 等类似操作代码重复、缓存升级改造、分库分表、数据库复用与耦合、SQL无法集中管理、DB耦合等引发的无关服务被动更新问题；

## 准备工作

- 业务拆分

要合理规划服务之间的界限；

- 服务治理

方便实现服务的发现、治理、熔断、降级等机制；

- 自动测试

尽量做到更多的环节实现自动化。

- 自动运维

方便部署升级、高效自动化运维。

- 服务监控

包括硬件环境、服务状态、系统健康度、接口调用情况、异常的实时告警以及潜在问题的事先预警等等。一句话：没准备好监控，就不要搞微服务。

### 设计原则

- AKF 拆分原则(AKF 扩展立方体)

  - X 水平复制，即多实例集群负载
  - Y 按业务服务功能拆分
  - Z 数据分区

- 前后端分离

优化前后端框架、便于接口设计开发和维护

- 无状态服务

将需要被多个服务共享的数据存储到分布式缓存中，让业务服务变成“无状态的计算节点”，便于动态扩展节点。

- Restful 风格

目前较多的有：① Restful ② RPC(Thrift, Dubbo)

建议采用无状态的 HTTP 协议，扩展性强；JSON 报文业务成熟友好。

### 服务拆分原则

#### 强制条件

- 服务内部应稳定
- 服务应实现相关的方法、必须遵循共同封闭的原则
- 不同服务间应是松耦合的，服务内部改变不影响外部其他服务
- 服务应易于测试性

#### 拆分解决方案

- 根据业务拆分(横向拆分)

比如用户服务、订单服务、产品服务等等，基于现有系统的各个业务流程，抽象出各个逻辑上可以独立出的服务，类似拆分代码的服务接口；

- 基础服务下沉

把公共的组件拆分成独立的原子服务，下沉到底层，形成相对独立的原子服务层。

- 基于领域模型拆分

[https://www.zhihu.com/question/24013141](https://www.zhihu.com/question/24013141)

[https://blog.csdn.net/jek123456/article/details/54691456](https://blog.csdn.net/jek123456/article/details/54691456)

[https://blog.csdn.net/featuresoft/article/details/52496186](https://blog.csdn.net/featuresoft/article/details/52496186)

[https://blog.csdn.net/huangshulang1234/article/details/78848420](https://blog.csdn.net/huangshulang1234/article/details/78848420)

[https://blog.csdn.net/Gupaoxueyuan/article/details/79173292](https://blog.csdn.net/Gupaoxueyuan/article/details/79173292)

> 标准化的功能，由底层服务为产品层提供标准能力支撑，具有强烈的个别业务特性化的功能，由产品层直接调用更底层的能力自己封装实现。

## 注意问题

- 远程服务调用的性能损耗
- 强一致性VS最终一致性(分布式事务 or CAP 问题)
- 更为复杂的跨服务测试
- 更复杂的运维工作
- 更高要求的整理规划与设计

### 事务一致性问题

大部分业务可以通过 *最终一致性* 解决，极少部分需要采用 *强一致性*。具体策略如下：

#### 最终一致性

基于 MQ 等消息中间件实现。

#### 强一致性

使用 TCC 框架，需要嵌入分布式事务框架来支持分布式事务。

> 领域驱动设计早已阐明，具有强一致性要求的一组业务概念，属于同一个聚合，不建议拆到不同服务中，从而尽可能避免分布式强事务一致性的处理。
而可以拆分的服务边界，是在限界上下文或者聚合的粒度上。这样的事务一致性属于最终一致性，可以用成熟的工具或者算法处理。

## 实战方案

### Service Mesh

此部分详见 https://www.sdnlab.com/20363.html

#### 定义

> 服务网格是用于处理服务到服务通信的专用基础设施层。它负责通过复杂的服务拓扑来可靠地传递请求。实际上，服务网格通常被实现为与应用程序代码一起部署的轻量级网络代理矩阵，并且它不会被应用程序所感知。

可总结 Service Mesh 为：

1. 专用基础设施层：独立的运行单元。

2. 包括数据层和控制层：数据层负责交付应用请求，控制层控制服务如何通讯。

3. 轻量级透明代理：实现形式为轻量级网络代理。

4. 处理服务间通讯：主要目的是实现复杂网络中服务间通讯。

5. 可靠地交付服务请求：提供网络弹性机制，确保可靠交付请求。

6. 与服务部署一起，但服务无需感知：尽管跟应用部署在一起，但对应用是透明的。

#### 作用

Service Mesh 作为透明代理，它可以运行在任何基础设施环境，而且跟应用非常靠近，那么，Service Mesh 能做什么呢？

- 负载均衡

运行环境中微服务实例通常处于动态变化状态，而且经常可能出现个别实例不能正常提供服务、处理能力减弱、卡顿等现象。但由于所有请求对 Service Mesh 来说是可见的，因此可以通过提供高级负载均衡算法来实现更加智能、高效的流量分发，降低延时，提高可靠性。

- 服务发现

以微服务模式运行的应用变更非常频繁，应用实例的频繁增加减少带来的问题是如何精确地发现新增实例以及避免将请求发送给已不存在的实例变得更加复杂。Service Mesh 可以提供简单、统一、平台无关的多种服务发现机制，如基于 DNS，K/V 键值对存储的服务发现机制。

- 熔断

动态的环境中服务实例中断或者不健康导致服务中断可能会经常发生，这就要求应用或者其他工具具有快速监测并从负载均衡池中移除不提供服务实例的能力。这种能力也称熔断，以此使得应用无需消耗更多不必要的资源不断地尝试，而是快速失败或者降级，甚至这样可避免一些潜在的关联性错误。而 Service Mesh 可以很容易实现基于请求和连接级别的熔断机制。

- 动态路由

随着服务提供商以提供高稳定性、高可用性以及高 SLA服务为主要目标，为了实现所述目标，出现各种应用部署策略尽可能从技术手段达到无服务中断部署，以此避免变更导致服务的中断和稳定性降低。例如：Blue/Green部署、Canary部署，但是实现这些高级部署策略通常非常困难。关于应用部署策略，可参考 EtienneTremel的文章，他对各种部署策略做了详细的比较。而如果运维人员可以轻松的将应用流量从 staging 环境到产线环境，一个版本到另外一个版本，更或者从一个数据中心到另外一个数据中心进行动态切换，甚至可以通过一个中心控制层控制多少比例的流量被切换。那么 Service Mesh 提供的动态路由机制和特定的部署策略如 Blue/Green 部署结合起来，实现上述目标更加容易。

- 安全通讯

无论何时，安全在整个公司、业务系统中都有着举足轻重的位置，也是非常难以实现和控制的部分。而微服务环境中，不同的服务实例间通讯变得更加复杂，那么如何保证这些通讯是在安全、授权情况下进行非常重要。通过将安全机制如 TLS 加解密和授权实现在 Service Mesh 上，不仅可以避免在不同应用的重复实现，而且很容易在整个基础设施层更新安全机制，甚至无需对应用做任何操作。

- 多语言支持

由于 Service Mesh 作为独立运行的透明代理，很容易支持多语言。

- 多协议支持

同多语言支持一样，实现多协议支持也非常容易。

- 指标和分布式追踪

Service Mesh 对整个基础设施层的可见性使得它不仅可以暴露单个服务的运行指标，而且可以暴露整个集群的运行指标。

- 重试和最后期限

Service Mesh 的重试功能避免将其嵌入到业务代码，同时最后期限使得应用允许一个请求的最长生命周期，而不是无休止的重试。

#### 工业产品

当前，业界主要有以下相关产品：

Buoyant 的 linkerd，基于 Twitter 的 Fingle，长期的实际产线运行经验及验证，支持 Kubernetes，DC/OS 容器管理平台，CNCF 官方支持的项目之一。

Lyft 的 Envoy，7 层代理及通信总线，支持 7 层 HTTP 路由、TLS、gRPC、服务发现以及健康监测等，也是 CNCF 官方支持项目之一。

IBM、Google、Lyft 支持的 Istio，一个开源的微服务连接、管理平台以及给微服务提供安全管理，支持 Kubernetes、Mesos 等容器管理工具，其底层依赖于 Envoy。

#### 总结

Service Mesh 可以使得快速转向微服务或者云原生应用，以一种自然的机制扩展应用负载，解决分布式系统不可避免的部分失败，捕捉分布式系统动态变化，完全解耦于应用等等。

我相信 Service Mesh 在微服务或者云原生应用领域一定别有一番天地。

### Spring Cloud

这个大家肯定不陌生了，Spring 全家桶肯定是大家的常用工具了，非常适合无法投入大量精力去打造基础组件的中小企业，适合快速开发与业务实现，强大的社区支持力量，提供了很完整的微服务实践支持，是我上手的第一选择。

![image](/images/posts/spring-cloud-framework.png)

我这里选择 Spring Cloud 作为微服务实践的工具。但是对于 Google 等巨头的强大创造力我是相当期待的，希望早日可以诞生更加强大的产品。

#### 介绍

Spring Cloud 官网介绍如下：

> Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer's own laptop, bare metal data centres, and managed platforms such as Cloud Foundry.

包括但不限于以下功能：

- Distributed/versioned configuration
- Service registration and discovery
- Routing
- Service-to-service calls
- Load balancing
- Circuit Breakers
- Global locks
- Leadership election and cluster state
- Distributed messaging

#### 组成

主要组成模块：

- Spring Cloud Config
- Spring Cloud Netflix
- Spring Cloud Bus
- Spring Cloud Cluster
- Spring Cloud Consul
- Spring Cloud Security
- Spring Cloud Sleuth
- Spring Cloud CLI
- Spring Cloud Gateway
- Spring Cloud OpenFeign
....

配合上

- Spring Boot Actuator
- Spring Boot admin

即可满足日常业务所需的功能。我觉得 Spring 全家桶对企业开发支持的很全面，互联网企业开发中涉及到的很多功能都在这里进行了良好的支持。

---
