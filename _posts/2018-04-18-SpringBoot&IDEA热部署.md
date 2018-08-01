---
title: SpringBoot&IDEA热部署
date: 2018-04-18
author: asing1elife
categories:
 - java
 - springboot
 - software
tags:
 - java
 - springboot
 - intellij
 - idea
 - software
---
> SpringBoot 自身有提供插件可实现代码热部署  

## IDEA 相关配置
1. 开启项目自动构建
![](http://asing1elife.com/sources/images/F0C28075-FA9E-471B-803A-F4377F7AEAE9.png)

## 代码相关配置
1. pom 中加入以下依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

2. 扩展 `spring-boot-maven-plugin` 的插件配置项

```xml
<build>
    <finalName>mop</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!-- 如果没有该项配置devtools不会起作用，即应用不会restart -->
                <fork>true</fork>
                <!-- 支持静态文件热部署 -->
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```
