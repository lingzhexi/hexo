---
title: 数据库系列（二） | Mybatis Plus 3.0+快速入门
tags:
  - 数据库
  - Mybatis Plus
categories:
  - 数据库
  - Mybatis Plus
cover: https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/image-20240927101721314.png
abbrlink: 60908
date: 2024-12-09 14:22:03
---

# Mybatis Plus 3.0+快速入门

> 官网：[https://baomidou.com/](https://baomidou.com/)
>
> 参考教程：[快速开始](https://baomidou.com/getting-started/)
>
> 最新版本：[3.5.7](https://baomidou.com/getting-started/#spring-boot3) 
>
> 对应版本参考：JDK 8+ 、Maven or Gradle

## 1、简介

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus) （简称 MP）是一个 [MyBatis](https://www.mybatis.org/mybatis-3/) 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

![Mybatis 最好的搭档](https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/image-20240927101721314.png) 

## 2、创建并初始化数据库

### 2.1、创建数据库

`mybatis_plus`

### 2.2、创建 `User` 表

其表结构如下：

| id   | name   | age  | email              |
| ---- | ------ | ---- | ------------------ |
| 1    | Jone   | 18   | test1@baomidou.com |
| 2    | Jack   | 20   | test2@baomidou.com |
| 3    | Tom    | 28   | test3@baomidou.com |
| 4    | Sandy  | 21   | test4@baomidou.com |
| 5    | Billie | 24   | test5@baomidou.com |

其对应的数据库 Schema 脚本如下：

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT(11) NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
```

其对应的数据库 Data 脚本如下：

```sql
DELETE FROM user;
INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

## 3、创建项目

### 3.1、创建maven项目

`mybatis_plus_demo`

`GroupId：com.stormling`

![创建Mybatis-Plus项目](https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/image-20240927102444438.png) 

### 3.2、SpringBoot版本

```xml
<version>3.0.5</version>
```

### 3.3、添加依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.5</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>
    
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
    
    <!--lombok用来简化实体类-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```



### 3.4、Idea中安装lombok插件

使用的是Idea 23.3.4，使用 lombok 是为了简化实体类的代码编程，不了解的同学可以出门左转看下[lombok特性](https://projectlombok.org/features/)，后续会专门写一篇lombok相关使用（挖一个坑）

![安装lombok插件](https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/image-20240927102655255.png)

## 4、编写代码

### 4.1、配置文件

 `application.properties` 配置文件

```properties
#mysql数据库连接
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
#mybatis日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```



### 4.2、实体类

```java
package com.stormling.mp.entity;
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```



### 4.3、mapper

```java
package com.stormling.mp.mapper;
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```



## 5、开始使用

添加测试类，进行功能测试：

```java
package com.stormling.mp;

@SpringBootTest
public class CRUDTests {
    @Resource
    private UserMapper userMapper;

    @Test
    public void testSelectList() {
        List<User> userList = userMapper.selectList(null);
        userList.forEach(System.out::println);
    }
}
```

**通过以上几个简单的步骤，我们就实现了 User 表的 CRUD 功能，甚至连 XML 文件都不用编写！**

## 6、主键策略

```java
@Test
public void testInsert(){
    User user = new User();
    user.setName("Stormling");
    user.setAge(22);
    user.setEmail("501344802@qq.com");
    int result = userMapper.insert(user);
    System.out.println(result); //影响的行数
    System.out.println(user); //id自动回填
}
```



**注意：**数据库插入id值默认为：全局唯一id

![新增一条数据](https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/image-20240927110351744.png) 



### 6.1、`ASSIGN_ID` 全局唯一`ID`（默认）

> 不了解雪花算法的童鞋，后续更新一篇手写一个雪花算法（**挖坑**）

```java
@TableId(type = IdType.ASSIGN_ID)
```

**雪花算法**是由 `Twitter` 公布的分布式主键生成算法，它能够保证不同进程主键的**不重复性**，以及相同进程主键的**有序性**。

**实现原理**

在同一个进程中，它首先是通过**时间位保证不重复**，如果时间相同则是通过**序列位保证**。 同时由于时间位是**单调递增**的，且各个服务器如果大体做了时间同步，那么生成的主键在分布式环境可以认为是**总体有序**的，这就保证了对**索引字段的插入的高效性**。 例如 `MySQL` 的 `Innodb` 存储引擎的主键。

使用雪花算法生成的主键，二进制表示形式包含 `4` 部分，从高位到低位分表为：`1bit` **符号位**、`41bit` **时间戳位**、`10bit` **工作进程位**以及 `12bit` **序列号位**。

- 符号位（`1bit`）

预留的符号位，恒为零。

- 时间戳位（`41bit`）

`41` 位的时间戳可以容纳的毫秒数是 `2` 的 `41` 次幂，一年所使用的毫秒数是：`365 * 24 * 60 * 60 * 1000`。 通过计算可知：

```java
Math.pow(2, 41) / (365 * 24 * 60 * 60 * 1000L);
```

结果约等于 `69.73` 年，相信能满足绝大部分系统的要求。

- 工作进程位（`10bit`）

该标志在 `Java` 进程内是唯一的，如果是分布式应用部署应保证每个工作进程的 `id` 是不同的。 该值默认为 0，可通过属性设置。可以支持 `1024`(2 的 10 次幂)个工作进程

- 序列号位（`12bit`）

该序列是用来在同一个毫秒内生成不同的 `ID`。如果在这个毫秒内生成的数量超过 `4096` (2 的 12 次幂)，那么生成器会等待到下个毫秒继续生成。

雪花算法主键的详细结构见下图。

![雪花算法](https://cdn.jsdelivr.net/gh/lingzhexi/blogImage/post/202409240943887.png)

### 6.2、`ASSIGN_UUID`

* 生成唯一的值，包含数字和字母，表对应字段类型`varchar`类型

```java
@TableId(type = IdType.AUTO)
private Long id;
```

### 6.3、`AUTO` 自增策略

- 要想主键自增需要配置如下主键策略

  - 需要在创建数据表的时候设置主键自增

  - 实体字段中配置 `@TableId(type = IdType.AUTO)`

```java
@TableId(type = IdType.AUTO)
private Long id;
```

###  6.4、`INPUT` 手动设置

- 需要手动设置`id`值

## 7、分页插件

> 参考教程：[分页插件](https://baomidou.com/plugins/pagination/)

`MyBatis Plus`自带分页插件，只要简单的配置即可实现分页功能

### 7.1、配置分页插件

创建`MybatisPlusConfig`

```java
package com.stormling.mp.config;


@Configuration
public class MybatisPlusConfig {
    /**
     * 分页插件
     */
    @Bean
    public MybatisPlusInterceptor optimisticLockerInnerInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //向Mybatis过滤器链中添加分页拦截器
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

属性介绍

`PaginationInnerInterceptor` 提供了以下属性来定制分页行为：

|  属性名  |   类型   | 默认值 |           描述           |
| :------: | :------: | :----: | :----------------------: |
| overflow | boolean  | false  | 溢出总页数后是否进行处理 |
| maxLimit |   Long   |        |     单页分页条数限制     |
|  dbType  |  DbType  |        |        数据库类型        |
| dialect  | IDialect |        |        方言实现类        |

> 建议单一数据库类型的均设置 dbType

 `Page` 类

`Page` 类继承了 `IPage` 类，实现了简单分页模型。如果你需要实现自己的分页模型，可以继承 `Page` 类或实现 `IPage` 类。

|         属性名         |      类型       |  默认值   |                    描述                     |
| :--------------------: | :-------------: | :-------: | :-----------------------------------------: |
|        records         |     List<T>     | emptyList |                查询数据列表                 |
|         total          |      Long       |     0     |              查询列表总记录数               |
|          size          |      Long       |    10     |            每页显示条数，默认 10            |
|        current         |      Long       |     1     |                   当前页                    |
|         orders         | List<OrderItem> | emptyList |                排序字段信息                 |
|    optimizeCountSql    |     boolean     |   true    |             自动优化 COUNT SQL              |
| optimizeJoinOfCountSql |     boolean     |   true    | 自动优化 COUNT SQL 是否把 join 查询部分移除 |
|      searchCount       |     boolean     |   true    |             是否进行 count 查询             |
|        maxLimit        |      Long       |           |              单页分页条数限制               |
|        countId         |     String      |           |     XML 自定义 count 查询的 statementId     |

通过这些配置和使用方法，你可以轻松地在 MyBatis-Plus 中实现分页查询，提高应用的性能和用户体验。



### 7.2、测试selectPage分页

```java
@Test
public void testSelectPage() {
    Page<User> page = new Page<>(1,5);
    userMapper.selectPage(page, null);
    page.getRecords().forEach(System.out::println);
    System.out.println(page.getCurrent()); // 当前页
    System.out.println(page.getPages());   // 查询列表分页数
    System.out.println(page.getSize());    // 每页显示条数
    System.out.println(page.getTotal());   // 查询列表总记录数
    System.out.println(page.hasNext());	   // 是否有下一页	
    System.out.println(page.hasPrevious());// 是否有上一页
}

```

 输出结果

```shell
==>  Preparing: SELECT COUNT(*) AS total FROM user
==> Parameters: 
<==    Columns: total
<==        Row: 6
<==      Total: 1
==>  Preparing: SELECT id,name,age,email FROM user LIMIT ?
==> Parameters: 5(Long)
<==    Columns: id, name, age, email
<==        Row: 1, Jone, 18, test1@baomidou.com
<==        Row: 2, Jack, 20, test2@baomidou.com
<==        Row: 3, Tom, 28, test3@baomidou.com
<==        Row: 4, Sandy, 21, test4@baomidou.com
<==        Row: 5, Billie, 24, test5@baomidou.com
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@9e30f9a]
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
1
2
5
6
true
false
```



## 8、逻辑删除

- 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除数据
- 逻辑删除：假删除，将对应数据中代表是否被删除的字段状态修改为“`被删除状态`”，之后在数据库中仍旧能看到此条数据记录

### 8.1、数据库中添加 deleted字段

```sql
ALTER TABLE `user` ADD COLUMN `deleted` BOOLEAN DEFAULT 0 COMMENT '逻辑删除字段';
```



## 8.2、实体类添加deleted 字段

并加上 `@TableLogic` 注解

```java
@TableLogic
private Integer deleted;
//private Boolean deleted;
```



### 8.3、application.properties 配置

此为默认值，如果你的默认值和mp默认的一样，则不需要该配置

```properties
#默认配置
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

如果使用了@TableLogic注解，则不需要该配置

```properties
mybatis-plus.global-config.db-config.logic-delete-field=deleted
```



### 8.4、测试逻辑删除

- 测试后发现，数据并没有被删除，deleted字段的值由0变成了1
- 测试后分析打印的sql语句，是一条update

```java
/**
 * 测试 逻辑删除
 */
@Test
public void testLogicDelete() {
    int result = userMapper.deleteById(1L);
    System.out.println(result);
}
```

输出结果

```shell
==>  Preparing: UPDATE user SET deleted=1 WHERE id=? AND deleted=0
==>  Parameters: 1(Long)
<==    Updates: 1
```



### 8.5、测试逻辑删除后的查询

MyBatis Plus中查询操作也会自动添加逻辑删除字段的判断`（deleted = 0）`

```java
/**
 * 测试 查询操作
 */
@Test
public void findAll() {
    List<User> users = userMapper.selectList(null);
    users.forEach(System.out::println);
}
```

输出结果

```shell
==>  Preparing: SELECT id,name,age,email,deleted FROM user WHERE deleted=0
==>  Parameters: 
<==    Columns: id, name, age, email, deleted
<==        Row: 2, Jack, 20, test2@baomidou.com, 0
<==        Row: 3, Tom, 28, test3@baomidou.com, 0
<==        Row: 4, Sandy, 21, test4@baomidou.com, 0
<==        Row: 5, Billie, 24, test5@baomidou.com, 0
<==        Row: 1839500916857540609, Stormling, 28, 501344802@qq.com, 0
<==        Row: 1844981342959046657, Stormling, 28, 501344802@qq.com, 0
<==      Total: 6
```