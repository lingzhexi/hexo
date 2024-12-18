---
title: 浅谈SpringMVC 框架
date: 2022-04-28 14:42:42
tags: SpringMVC 
categories: SpringMVC
description:  介绍SpringMVC框架开发的原理以及开发流程的配置相关学习
---

<meta name="referrer" content="no-referrer"/>

<!--more-->

> 本文介绍SpringMVC 框架的作用、原理和开发流程

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xiaoxi/p/6164383.html)

# SpringMVC 原理

SpringMVC 的工作原理图：

![SpringMVC工作原理](https://gcore.jsdelivr.net/gh/lingzhexi/blogImage/img/2022/03/202204281522150.jpeg)

## SpringMVC 流程

1、  用户发送请求至前端控制器 DispatcherServlet。

2、  DispatcherServlet 收到请求调用 HandlerMapping 处理器映射器。

3、  处理器映射器找到具体的处理器 (可以根据 xml 配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成) 一并返回给 DispatcherServlet。

4、  DispatcherServlet 调用 HandlerAdapter 处理器适配器。

5、  HandlerAdapter 经过适配调用具体的处理器 (Controller，也叫后端控制器)。

6、  Controller 执行完成返回 ModelAndView。

7、  HandlerAdapter 将 controller 执行结果 ModelAndView 返回给 DispatcherServlet。

8、  DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器。

9、  ViewReslover 解析后返回具体 View。

10、DispatcherServlet 根据 View 进行渲染视图（即将模型数据填充至视图中）。

11、 DispatcherServlet 响应用户。

## 组件说明

以下组件通常使用框架提供实现：

- DispatcherServlet：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。

- HandlerMapping：通过扩展处理器映射器实现不同的映射方式，例如：**配置文件方式，实现接口方式，注解方式** 等。 

- HandlAdapter：通过扩展处理器适配器，支持更多类型的处理器。

- ViewResolver：通过扩展视图解析器，支持更多类型的视图解析，例如：**jsp、freemarker、pdf、excel** 等。

### 组件  

#### **1、前端控制器 DispatcherServlet（不需要工程师开发）, 由框架提供**  

作用：接收请求，响应结果，相当于转发器，中央处理器。有了 dispatcherServlet **减少**了其它组件之间的**耦合度**。  
用户请求到达前端控制器，它就相当于 mvc 模式中的 c，dispatcherServlet 是整个流程控制的中心，由它**调用其它组件处理用户的请求**，dispatcherServlet 的存在降低了组件之间的耦合性。

#### **2、处理器映射器 HandlerMapping(不需要工程师开发), 由框架提供**  

作用：根据请求的 url 查找 Handler  
HandlerMapping 负责根据用户请求找到 Handler 即处理器，springmvc 提供了不同的映射器实现不同的映射方式，例如：**配置文件方式，实现接口方式，注解方式**等。

#### **3、处理器适配器 HandlerAdapter**  

作用：按照特定规则（HandlerAdapter 要求的规则）去执行 Handler  
通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

#### **4、处理器 Handler(需要工程师开发)**  

**注意：编写 Handler 时按照 HandlerAdapter 的要求去做，这样适配器才可以去正确执行 Handler**  
Handler 是继 DispatcherServlet 前端控制器的后端控制器，在 DispatcherServlet 的控制下 Handler 对具体的用户请求进行处理。  
由于 Handler 涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发 Handler。

#### **5、视图解析器 View resolver(不需要工程师开发), 由框架提供**  

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）  
View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。 springmvc 框架提供了很多的 View 视图类型，包括：jstlView、freemarkerView、pdfView 等。  
一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

#### **6、视图 View(需要工程师开发 jsp...)**  

View 是一个接口，实现类支持不同的 View 类型（jsp、freemarker、pdf...）