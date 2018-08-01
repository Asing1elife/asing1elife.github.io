---
title: Maven针对SpringBoot的多项目依赖配置
date: 2017-12-16
author: asing1elife
categories:
 - java
 - maven
 - springboot
tags:
 - java
 - maven
 - springboot
---
> 在Spring Boot中对于多项目依赖配置，可通过Maven实现  

## 创建一个父级的pom文件
1. 在该pom中指定其 `<groupId/>`、`<artifactId/>` 、`<version/>`
2. `<packaging/>` 必须是pom
3. `<parent/>` 需要引入 `spring-boot-starter-parent` 表示继承Spring Boot的父级配置
4. `<modules/>` 中依次引入后续需要相互依赖的项目
5. `<dependencyManagement/>` 中对上述引入的项目进行完成依赖配置

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.innovaee.hts.parent</groupId>
    <artifactId>innovaee-hts-parent</artifactId>
    <version>1.0.0.RELEASE</version>
    <packaging>pom</packaging>

    <name>innovaee-hts-parent</name>
    <url>http://maven.apache.org</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <modules>
        <module>hts-admin-backend</module>
  ...
    </modules>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.innovaee.hts</groupId>
                <artifactId>hts-admin-backend</artifactId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>
  ...
        </dependencies>
    </dependencyManagement>

</project>
```

## 在各个子项目的pom中对依赖关系进行配置
* 父级不再指向Spring Boot的父级，而是上述自定义父级

```xml
<parent>
    <groupId>com.innovaee.hts.parent</groupId>
    <artifactId>innovaee-hts-parent</artifactId>
    <version>1.0.0.RELEASE</version>
</parent>
```

* 从父级中获取需要的依赖配置

```xml
<dependency>
    <groupId>org.tshark.core</groupId>
    <artifactId>tshark-core</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  ...
</dependency>
```

## 在需要被打包成jar包的项目中引入Spring Boot的编译插件

```xml
<build>
    <finalName>hts-admin</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## 在父级pom所在的目录中执行打包命令对项目进行打包

```shell
mvn clean package -Dmaven.test.skip=true
```