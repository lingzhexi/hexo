---
title: SpringBoot 常见面试题 30道
date: 2022-02-22 16:12:10
tags:
categories:
description:
---
> 简介：主要对于SpringBoot是什么，如何使用

## 1.什么是 SpringBoot 
Spring组件一站式解决方案，主要简化了 Spring 难度，简省了繁重的配置，提供了各种的启动器，是开发上手快

## 2.Spring Boot 优点

1. 开箱即用，原理繁琐配置
2. 内嵌服务器、安全管理、运行数据监控、运行状态检查、外部化配置
3. 易上手开发效率高，有完善的第三方start库和官网 starter

总结：**编码、配置、部署、监控** 简单
<!--more-->
## 2.SpringBoot 启动类注解？它是由那些注解组成？
@SpringBootApplication
- @SpringBootConfiguration：组合了@Configuration注解，实现配置文件的功能。
- @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项
- @SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
- @ComponentScan：Spring组件扫描

## 3.yaml是什么
用来表达数据序列化的参数

## 4.SpringBoot启动方式
1. main方法
2. 命令行 java -jar
3. mvn/gradle

## 5.SpringBoot 需要独立的容器独立？
内置了Tomcat/Jetty

## 6.SpringBoot自动配置原理
	在sprinBoot启动时由 @SpringBootApplication 注解会自动去maven中读取每个 starter 中的 spring.factories文件,该文件里配置了所有需要被创建spring容器中的bean，并且进行自动配置把 bean注入SpringContext中 //（SpringContext是Spring的配置文件）

## 7.SpringBoot监视器是什么，如何配置监控？

	Spring boot actuator 是 spring 启动框架中的重要功能之一。
	Spring boot 监视器可帮助您访问生产环境中 **正在运行的应用程序的当前状态** 。
	有几个指标必须在生产环境中进行检查和监控。即使一 些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接 作为 HTTP URL 访问的REST 端点来检查状态。

pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

yml

```yaml
management:
    endpoint:
        health: ## 开启健康监控端点
            enabled: true
        beans: ## 开启Bean实例监控端点
            enabled: true
```

健康监控开启标志，启动了两个端点，默认之开启 health 和 info 端口

![image-20220223093051031](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223093051031.png) 

yml

```yaml
management:
	endpoints:
		web:
			exposure:
				include: "*" ## 开启所有端点暴露
```



## 8.查看各个监控信息？

/actuator 端点 ，默认开启了两个端点，health 和 info

```json
{
  "_links": {
    "self": {
      "href": "http://localhost:9999/actuator",
      "templated": false
    },
    "health": {
      "href": "http://localhost:9999/actuator/health", 查看当前服务的是否上线
      "templated": false
    },
    "health-path": {
      "href": "http://localhost:9999/actuator/health/{*path}",
      "templated": true
    },
    "info": {
      "href": "http://localhost:9999/actuator/info",
      "templated": false
    }
  }
}
```

如果将所有端点暴露

访问路径： ip:port/actuator/xx

![image-20220223094652254](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223094652254.png) 

### loggers 端点

访问 `http://localhost:8080/actuator/loggers` 可以查看当前应用的日志级别等信息：

![](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223095239970.png) 

这里面本身并不特别，但是有一个功能却非常有用，比如我们生产环境日志级别一般都是 info，但是现在有一个 bug 通过 info 级别无法排查，那么我们就可以临时修改 log 级别。

比如上图中的 ROOT 节点是 info 级别，那么我们可以通过 postman 等工具来发一个 post 请求修改日志级别。 

![image-20220223095330889](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223095330889.png) 

修改之后就会发现，日志由原来的 info 变成了 debug： 

![image-20220223095351178](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223095351178.png) 

### metrics 端点

非常重要的监控端点，监控内容包含 JVM内存、堆、类加载、处理器和 tomcat 容器等重要的指标

![image-20220223095921378](https://gitee.com/lingzhexi/blogImage/raw/master/img/2021/12/image-20220223095921378.png) 

### 自定义监控端点 

> 这一部分之后再详介绍

## 9.SpringBoot 中的starter?

可以理解成对依赖的一种合成，starter会把一个或者一套功能相关的依赖包含进来，避免造轮子。



## 10.什么是SpringProfiles

开发到生产，经过开发（dev）、测试（test）、上线（prod），主要针对不同的的配置。Spring Profiles允许用户根据配置文件（dev/test/prod）来注册bean。

## 11.激活不同环境配置

yml:

```yaml
spring:
	profiles:
		active: dev
```

   命令行：

```shell
java -jar xx.jar --spring.profiles.active=dev
```

## 12.SpringBoot异常处理相关注解？

@ControllerAdvice

@ExceptinoHandler

## 13.SpringBoot1.1 和 2.x的区别？

1. SpringBoot 2基于Spring5和JDK8，Spring 1.x用的最低版本
2. 配置变更，参数名
3. SpringBoot 2相关插件最低版本很多都要比原来高
4. 2.x配置中的中文可以直接读取

## 14. Spring Boot 中如何解决跨域问题 ? 

	后端通过 （CORS，Cross origin resource sharing） 来解决跨域问题。这种解决方案并非 Spring Boot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置 CORS ， 现在可以通过实现 **WebMvcConfigurer** 接口然后重写 **addCorsMappings** 方法解决跨域问题。

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        .allowedOrigins("*")
        .allowCredentials(true)
        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
        .maxAge(3600);
    }
}
```



## 参考：

https://mp.weixin.qq.com/s/kisvBJABJ27rb6JNZdCyqA