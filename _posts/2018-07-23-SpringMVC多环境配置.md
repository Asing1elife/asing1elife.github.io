---
title: SpringMVC多环境配置 
date: 2018-07-23
author: asing1elife
categories:
 - java
 - springmvc
tags:
 - java
 - springmvc
 - profile
 - spring
---
> SpringMVC 可以使用 Spring 本身提供的 profile 特性对多环境配置文件进行统一集成，自动切换  

## 存在的必要
1. 日常开发中，一般都存在多个环境，开发、测试、生产
2. 上述环境对应的数据库及配置文件都会存在不同，所以为项目集成多环境配置很有必要

## 实现方式
1. 集成方式有多种，网上介绍的大多是使用 `<beans profile="dev">` 去区分不同的 `*.properties` 文件
2. 还可以使用 `@Profile` 进行不同环境代码加载
3. 本文介绍的是使用 `<beans profile="dev">` 直接在同一个 xml 文件中区分不同环境需要的不同配置项

## XML配置
1. 注意如果使用了多环境的配置的 xml 文件中存在其他公有属性，这些属性需要放在最前面，否则会报错

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">
  <description>spring-data-redis-cluster</description>

  <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxIdle" value="${redis.pool.maxIdle}"/>
    <property name="maxTotal" value="${redis.pool.maxActive}"/>
    <property name="maxWaitMillis" value="${redis.pool.maxWait}"/>
    <property name="testOnBorrow" value="${redis.pool.testOnBorrow}"/>
  </bean>

  ...

  <beans profile="dev">
    <bean id="jedisPool" class="redis.clients.jedis.JedisPool">
      <constructor-arg index="0" ref="jedisPoolConfig"/>
      <constructor-arg index="1" value="${redis.server.url}"/>
      <constructor-arg index="2" value="${redis.server.port}" type="int"/>
      <constructor-arg index="3" value="${redis.server.timeout}" type="int"/>
      <constructor-arg index="4" value="${redis.server.password}"/>
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
      <property name="hostName" value="${redis.server.url}"/>
      <property name="port" value="${redis.server.port}"/>
      <property name="password" value="${redis.server.password}"/>
      <property name="poolConfig" ref="jedisPoolConfig"/>
    </bean>
  </beans>

  <beans profile="prod">
    <bean id="redisClusterConfiguration" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
      ...
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
      <constructor-arg ref="redisClusterConfiguration"/>
      <constructor-arg ref="jedisPoolConfig"/>
    </bean>
  </beans>
</beans>
```

## 开发环境启动方式
1. IDEA 需要在启动时添加脚本 `-Dspring.profiles.active="dev"` 以确保使用开发环境启动
![](http://asing1elife.com/sources/images/8855BF1F-4F40-4B99-8E97-044B6C875633.png)

## 生产环境启动方式
1. Linux 环境需要前往 Tomcat 的 /bin 目录下，需要 **./catalina.sh** 中的 `JAVA_OPTS` 内容如下
2. `JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=prod"`