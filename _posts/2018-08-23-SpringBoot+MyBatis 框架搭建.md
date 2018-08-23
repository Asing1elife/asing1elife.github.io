---
title: SpringBoot+MyBatis 框架搭建
date: 2018-08-23
author: asing1elife
categories:
 - java
 - springboot
 - mybatis
tags:
 - java
 - springboot
 - mybatis
---
> 基础框架使用 SpringBoot  
> 持久层使用 MyBatis  
> 数据源使用 Druid  
> 数据库使用 MySQL  

## 创建 Java Web 项目，并配置 POM
1. 该 POM 只包含最基础的配置，没有任何多余 JAR 包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!-- 项目基础信息 -->
  <groupId>com.asing1elife.newso</groupId>
  <artifactId>asl-newso</artifactId>
  <version>1.0-SNAPSHOT</version>

  <!-- SpringBoot 基础依赖 -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
  </parent>

  <!-- 指定编码格式和 JDK 版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- SpringBoot 相关依赖 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
      <version>2.0.2.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>2.0.2.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.0.2.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <version>2.0.2.RELEASE</version>
    </dependency>

    <!-- MyBatis 相关依赖 -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.2</version>
    </dependency>

    <!-- MySQL 相关依赖 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>6.0.6</version>
    </dependency>

    <!-- Druid 相关依赖 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.9</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.9</version>
    </dependency>

  </dependencies>

</project>
```

## 指定端口号
1. 在 **/resources** 根目录创建 **application.yml** 文件并添加如下内容
2. 表示端口号为 8080
3. 以及项目请求根路径为 `/newso`

```yaml
server:
  port: 8080
  servlet:
    context-path: /newso
```

## 指定数据源
1. 在 **application.yml** 中添加如下内容为数据源连接信息
2. 数据源使用过的是阿里的 Druid

```yaml
spring:
  datasource:
    name: newso_dev
    url: jdbc:mysql://127.0.0.1:3306/newso_dev?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    druid:
      filters: stat
      max-active: 20
      initial-size: 1
      max-wait: 60000
      min-idle: 1
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: select 'x'
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-open-prepared-statements: 20
```

## 指定 MyBatis 配置
1. 同样可在 **application.yml** 中指定 MyBatis 的配置信息
2. `mapper-locations` 是指定 mapper 文件所在位置
3. `type-aliases-package` 是指定对应实体类所在位置

```yml
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.asing1elife.newso.dto
```

## 作为前后分离项目必不可少的结果集对象
1. 前后分离项目都是采用前端发起请求，后端返回结果
2. 为了保证后端返回结果的统一性，所以需要如下的结果集对象
3. 只有这样前端在处理后端返回结果时，才能有规范的操作

```java
public class ResponseData {

    private static final String SUCCESS_CODE = "0";
    private static final String ERROR_CODE = "-1";

    private String status = SUCCESS_CODE;

    private Object data;

    ...

    public boolean isSuccess() {
        return this.status.equalsIgnoreCase(SUCCESS_CODE);
    }

    public void setError(Object data) {
        this.status = ERROR_CODE;
        this.data = data;
    }
}
```

## 控制层基础控制类
1. 基础控制类的作用在于可以在调用逻辑层代码时有一个统一的预处理操作或者后置操作

```java
public class SimpleActionHandler {

    private Logger log = LoggerFactory.getLogger(this.getClass());

    // 当前请求
    private String action;

    public SimpleActionHandler(HttpServletRequest request) {
        // 记录当前请求
        this.action = request.getRequestURI();
    }

    public void doAction(ResponseData responseData) {
        throw new RuntimeException("This method must be overide");
    }

    public ResponseData handle(HttpServletRequest... requests) {
        ResponseData responseData = new ResponseData();

        try {
            doAction(responseData);
        } catch (Exception e) {
            log.info(this.action, e);
            responseData.setError(e.getMessage());
        }

        return responseData;
    }
}
```

## 创建控制层接收请求信息
```java
@RestController
@RequestMapping("/api/homes")
public class HomeController extends AbstractBaseController {

    @GetMapping("")
    public ResponseData main() {
        return new SimpleActionHandler(request) {
            @Override
            public void doAction(ResponseData responseData) {
                System.out.println("welcome");
            }
        }.handle();
    }

}
```

## MyBatis 通过自定义接口规范 DAO 层
1. 通过 `@Mapper` 注解可以确保容器只扫描被注解的 DAO
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Component
@Mapper
public @interface MyBatisRepository {
}
```

## 创建 DAO 接口
1. 在 DAO 接口使用自定义的注解即可保证该接口被容器扫描到

```java
@MyBatisRepository
public interface NewsDaoMyBatis {
    List<NewsDTO> getNewss();
}
```

## 创建 Mapper 文件
1. Mapper 文件的 DTD 都是统一格式的，按照如下规范即可

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.asing1elife.newso.module.news.dao.NewsDaoMyBatis">

  <select id="getNewss" resultType="NewsDTO">
    SELECT
      ne.id,
      ne.title,
      ne.brief,
      ne.author_id AS 'author.id',
      ne.createTime
    FROM
      ns_news ne
  </select>

</mapper>
```