---
title: SpringBoot 集成 slf4j 日志
date: 2019-02-13
author: asing1elife
categories:
 - springboot
 - slf4j
tags:
 - springboot
 - slf4j
 - logback
---

> 集成日志时直接在项目的 **/resources** 目录引入以下文件即可  

## 添加依赖
1. 如果项目是基于SpringBoot开发的，在使用 **slf4j** 和 **logback** 时不需要引入各自的依赖
2. 因为SpringBoot默认已经集成了这两个依赖，位于 **spring-boot-starter-logging** 中
3. 具体的依赖路径为 `spring-boot-starter-web -> spring-boot-starter -> spring-boot-starter-logging`
4. 在 **spring-boot-starter-logging** 的 **pom.xml** 中，存在以下依赖
```xml
<dependencies>
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-to-slf4j</artifactId>
    <version>2.10.0</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.25</version>
    <scope>compile</scope>
  </dependency>
</dependencies>
```

## 日志配置内容
1. 日志文件的名称直接使用 **logback-spring.xml** ，这是SpringBoot能接受的默认日志文件名称
	* 放置在项目 **/resources** 根目录，可以直接被SpringBoot识别
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">

  <contextName>asl-teamnote</contextName>

  <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <!-- 典型的日志pattern -->
    <encoder>
      <pattern>%date{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <!-- 综合时间与大小的滚动策略，先按小时滚动，小时内的文件大于10mb时再按大小滚动 -->
  <appender name="defaultLogFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <!-- 过滤 Error -->
      <level>ERROR</level>
      <!-- 匹配到就禁止 -->
      <onMatch>DENY</onMatch>
      <!-- 没有匹配到就允许 -->
      <onMismatch>ACCEPT</onMismatch>
    </filter>

    <file>/opt/asl-logs/asl-teamnote.log</file>

    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>/opt/asl-teamnote-%d{yyyy-MM-dd_HH}.%i.zip</fileNamePattern>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <maxFileSize>10MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>

    <encoder>
      <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <!-- 主要记录业务异常日志，先按小时滚动，小时内的文件大于10mb时再按大小滚动 -->
  <appender name="simpleExceptionLogFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!-- 只获取ERROR级别 -->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>

    <file>/opt/asl-logs/asl-teamnote-error.log</file>

    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>/opt/asl-logs/asl-teamnote-error%d{yyyy-MM-dd_HH}.%i.zip</fileNamePattern>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <maxFileSize>10MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>

    <encoder>
      <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="simpleException" level="INFO" additivity="false">
    <appender-ref ref="console"/>
    <appender-ref ref="simpleExceptionLogFile"/>
  </logger>

  <!-- project default level -->
  <logger name="com.asing1elife.teamnote" level="DEBUG"/>

  <logger name="org.springframework" level="ERROR"/>

  <!-- root -->
  <root level="INFO">
    <appender-ref ref="console"/>
    <appender-ref ref="defaultLogFile"/>
    <appender-ref ref="simpleExceptionLogFile"/>
  </root>
</configuration>
```