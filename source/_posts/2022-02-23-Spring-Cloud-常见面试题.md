---
title: Spring Cloud 常见面试题
date: 2022-02-05 11:41:46
tags: 面试
categories: 面试
summary:  微服务架构的组成
---
<meta name="referrer" content="no-referrer"/>

![](https://raw.githubusercontent.com/lingzhexi/blogImage/master/img/2022/03/202203021655862.jpg)

# 微服务基础

## 1.什么是微服务架构

​	微服务架构就是将**单体的应用程序分成多个应用程序**，这多个应用程序就成为微服务，每个微服务 运行在自己的进程中，并使用轻量级的机制通信。这些服务围绕业务能力来划分，并通过自动化部 署机制来独立部署。这些服务可以使用不同的编程语言，不同数据库，以保证最低限度的集中式管理。

## 2.为什么需要学习Spring Cloud

- 首先Spring Cloud基于Spring Boot的优雅简洁，可还记得我们被无数xml支配的恐惧？可还记得 Spring MVC ，Mybatis 错综复杂的配置，有了Spring Boot，这些东西都不需要了，Spring Boot好处不再赘诉，Spring Cloud就基于Spring Boot把市场上优秀的服务框架组合起来，通过Spring Boot风 格进行再封装屏蔽掉了复杂的配置和实现原理 
- 什么叫做开箱即用？即使是当年的黄金搭档 Dubbo + ZooKeeper下载配置起来也是颇费心神的！而 Spring Cloud完成这些只需要一个jar的依赖就可以了！ 
- Spring Cloud大多数子模块都是直击痛点，像 Zuul 解决的跨域，Fegin 解决的负载均衡，Hystrix的熔 断机制等等等等

## 3. Spring Cloud 是什么

- Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如 **服务发现注册**、**配置中心**、**智能路由**、**消息总线**、**负载均衡**、**断路器**、**数据监控**等，都可以用Spring Boot的开发风格做到一键启动和部署。

-  Spring Cloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框 架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留 出了一套简单易懂、易部署和易维护的分布式系统开发工具包

## 4. Spring Cloud的优缺点

优点

1. 耦合低
2. 配置简单
3. 跨平台
4. 可配置独立的数据库
5. 可以组件间之间通讯

缺点：

1. 部署麻烦
2. 数据管理麻烦
3. 系统集成测试
4. 性能监控复杂

## 5. Spring Boot 和 Spring Cloud 区别？

- Spring Boot 专注单体快速开发

- Spring Cloud 关注全局微服务协调整理治理框架，将Spring Boot 单体微服务整个管理

- 各个微服务之间提供，配置管理，服务发现，断路器，路由，微代理，消息总线，全局锁，决策竞选，分布式会话等

## 6.Spring Cloud 和 Spring Boot 版本对应关系?

| Release Train       | Boot Version                          |
| :------------------ | :------------------------------------ |
| 2020.0.x aka Ilford | 2.4.x, 2.5.x (Starting with 2020.0.3) |
| Hoxton              | 2.2.x, 2.3.x (Starting with SR5)      |
| Greenwich           | 2.1.x                                 |
| Finchley            | 2.0.x                                 |
| Edgware             | 1.5.x                                 |
| Dalston             | 1.5.x                                 |

## 7.SpringCloud由什么组成

这里列举几个主要的组件

- Eureka 服务注册和发现
- Zuul 网关
- Ribbon 负载均衡
- Feign 声明式Web服务客户端
- Hystrix 断路器
- Config 分布式统一配置管理

# Eureka 篇

## 8.服务注册和发现是什么意思？Spring Cloud 如何实现的?

​	一般Spring Cloud 项目由多个模块服务组成，通常在属性文件中进行配置，随着越来越多的服务开发和部署，添加修改属性变得复杂。由于所有服务都通过Eureka 服务注册统一由Eureka服务管理并通过Eureka完成查找，这样就无需知道服务的地的任何修改了。

## 9.什么是Eureka

​	Eureka 作为 Spring Cloud 服务注册中心，系统中的服务使用 Eureka 客户端将其连接 Eureka Service 中并保持心跳，可以通过 Eureka 服务来监控其他微服务是否正常运行

## 10.Eureka 如何实现高可用

​	通过集群注册多台 Eureka，将各个 微服务 相互注册

## 11.Eureka 的自我保护机制

​	默认情况下，如果 Eureka 服务一定时间没有收到某个微服务的心跳，那个Eureka 服务会进入自我保护模式，在该模式下 Eureka 服务会保护注册表中的信息，不删除注册表中的数据，当网路恢复后，自动退出保护模式。

## 12.DiscoveryClient 作用

​	可以从注册中心中的服务别名来注册服务器信息

## 13.Eureka 和ZooKeeper 的区别

1. ZooKeeper 的节点服务挂了要选举，选举期间的注册服务瘫痪
2. Eureka的各个节点平等，服务器挂了没关系，只要由一台可以保证服务即可，如果数据不是最新的，可能是启动了自我保护机制导致的。

3. Eureka 本质是工程，ZooKeeper 是进程
4. ZooKeeper 保证CP，Eureka 保证 AP

> CAP: C:一致性、A:可用性、P:分区容错性

# Zuul 篇

## 14.什么是网关

- 网关相当与网络服务框架的入口，所有网络请求都必须通过网关才能转发到具体的服务

## 15.作用是什么

- 统一管理微服务请求，权限控制、负载均衡、路由转发、监控、安全控制黑名单和白名单

## 16.什么是Spring Cloud Zuul (服务网关)

- Spring Cloud 的一套路由方案，会根据请求路径不同，网关会定位到指定的微服务，并代理请求到不同的微服务接口，对外隐蔽了微服务的真正接口地址，
  三个重要概念
  - 动态路由表：Zuul 支持 Eureka 路由，手动配置路由
  - 路由定位： 根据请求路径，Zuul 有自己一套定位服务规则以及路由表达式匹配
  - 反向代理：客户端请求到路由网关，网关手里后，目标发送请求，拿到相应后给到客户端
- 应用场景：
  - 对外暴露
  - 权限校验
  - 服务聚合
  - 日志审计

## 17.网关和过滤器有什么区别

- 网关对所有的服务请求进行分析过滤，过滤器是对于单个服务而言的

## 18.常用的网关框架

- Nginxx、Zuul、Gateway

## 19.Zuul 和 Nginx区别

- Zuul java 实现，主要是网关服务
- Nginx C 实现，性能高于Zuul，可以做Zuul集群

## 20.如何设计一套API接口

- API分类：开发API接口 和 内网API接口
  - 内网：局域网，为内部服务考虑
  - 外网：外部单位提供接口调用，遵循Oauth2.0权限
- 考虑安全和幂等性

## 21.ZuulFilter 常用那些方法

- Run()：过滤器的具体业务逻辑 
- shouldFilter()：判断过滤器是否有效 
- filterOrder()：过滤器执行顺序 
- filterType()：过滤器拦截位置

## 22.实现动态Zuul网关路由转发

- 通过path配置拦截请求，通过ServicerId配置中心转发道服务列表，内部使用Ribbon实现本地负载均衡和转发

# Ribbon

## 23.负载均衡的意义
