---
title: SpringBoot启动抛出Unregistering JMX-exposed beans on shutdown异常
date: 2018-04-16
author: asing1elife
categories:
 - java
 - springboot
 - error
tags: 
 - java
 - springboot
 - error
---
> 在配置 SpringBoot 项目时可能会抛出 Unregistering JMX-exposed beans on shutdown异常 

## 抛出错误的原因
1. 是因为以下依赖的 scope 为 **provided** 导致的

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

## 解决的办法
1. 将 **provided** 改为 **compile** 即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>compile</scope>
</dependency>
```