---
title: SpringBoot集成ActiveMQ抛出java.lang.NoClassDefFoundError异常
date: 2018-05-08
author: asing1elife
categories:
 - java
 - springboot
 - springmvc
 - maven
 - error
tags:
 - java
 - springboot
 - springmvc
 - maven
 - error
---
> SpringBoot 在集成 JMS 及 ActiveMQ 时抛出 java.lang.NoClassDefFoundError: javax/jms/JMSContext 异常  

## 出现问题的原因
1. **spring 5.0** 以上版本不会自动导入 **JMS 2.0** 的依赖
2. 但是 **activemq-core 5.7**  版本需要 **JMS 2.0** 的依赖

## 解决办法
1. 手动加入 **JMS 2.0** 依赖

```xml
<dependency>
    <groupId>javax.jms</groupId>
    <artifactId>javax.jms-api</artifactId>
    <version>2.0.1</version>
</dependency>
```

2. 在 **activemq-core 5.7** 中移除低版本的默认引入

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-core</artifactId>
    <version>5.7.0</version>
    <exclusions>
        <exclusion>
            <artifactId>spring-context</artifactId>
            <groupId>org.springframework</groupId>
        </exclusion>
        <exclusion>
            <groupId>org.apache.geronimo.specs</groupId>
            <artifactId>geronimo-jms_1.1_spec</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```