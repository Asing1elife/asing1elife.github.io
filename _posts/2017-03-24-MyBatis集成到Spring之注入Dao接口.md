---
title: MyBatis集成到Spring之注入Dao接口
date: 2017-03-24
author: asing1elife
categories:
 - java
 - springmvc
 - mybatis
tags:
 - java
 - springmvc
 - mybatis
---
> MyBatis 的 MapperScannerConfigurer 可以将 Dao 接口和 Mapper 文件注入到 Spring 容器中  

## 配置
```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
...
</bean>
```

## 属性
1. 指定 Session 工厂
	* 通常情况下系统只会存在一个 DataSource ，这时 MapperScannerConfigurer 会自动装配 Session 工厂，无需手动指定
	* 必须使用 value 注入 Bean 的名称，而不是使用 ref 对 Bean 进行引用
	* sqlSessionFactory 的实例在 [MyBatis 集成到 Spring 之生成 Session 工厂](http://asing1elife.com/java/springmvc/mybatis/2017/03/24/MyBatis集成到Spring之生成Session工厂/)
	* `<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />`
2. 指定 Dao 接口的扫描路径
	* 同时指定多个路径只需要通过逗号分隔即可
	* `<property name="basePackage" value="org.seckill.module" />`