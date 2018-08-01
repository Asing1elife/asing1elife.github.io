---
title: SpringBoot集成Swagger
date: 2018-04-17
author: asing1elife
categories:
 - java
 - springboot
 - swagger
tags: 
 - java
 - springboot
 - swagger
---
> Swagger 是一款目前世界最流行的API管理工具  

## 官网
1. [Swagger](https://swagger.io)
2. [Swagger Annotation](https://www.jianshu.com/p/b0b19368e4a8)

## 集成步骤
1. 在项目 pom 中引入以下依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

2. 在项目中配置 Swagger

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
.apis(RequestHandlerSelectors.any())
.paths(or(regex("/api/.*"))).build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title("轻实训-移动端 API").version("1.0").build();
    }
}
```