---
title: Lombok 自动生成实体类 getter/setter 方法
date: 2019-01-03
author: asing1elife
categories:
- java
tags:
- java
- lombok
---
> Lombok 是一个通过注解帮助实体类自动生成 getter/setter 方法的依赖  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 官网
[lombok](https://mvnrepository.com/artifact/org.projectlombok/lombok)

## 使用方式

### IDEA下载插件
1. 下载插件后可以提供更友好的语法提示
![](http://asing1elife.com/sources/images/C80A5017-0C9D-48F5-A382-5ED89C7F6447.png)

### POM引入依赖
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```

### 为实体类添加注解
1. 为实体类添加 `@Data` 注解，该实体类在编译时就会自动添加 *getter/setter* 函数，以及 *toString* 函数
2. 对于需要自定义的 *getter/setter* 函数，也可以自行生成

```java
@Data
public class ProjectModel {

    private String name;

    @JsonIgnore
    public String getName() {
        return name;
    }
}
```

## 解除 @EqualsAndHashCode(callSuper=false) 警告

### 异常表现
1. 在IDE编译的时候，经常会抛出以下警告
2. 具体描述是因为被添加 `@Data` 注解的是一个派生类，所以会给出以下警告，告知子类的equals/hashCode方法不会继承父类的属性
3. 其实按照IDE提示，在对应子类添加 `@EqualsAndHashCode(callSuper=false)` 即可解决
4. 但如果子类比较多的话，这个过程就会很繁琐，我们使用lombok的初衷就是想要避免繁琐，所以这并不是一个好的解决办法

```java
Warning:(11, 1) java: Generating equals/hashCode implementation but without a call to superclass, even though this class does not extend java.lang.Object. If this is intentional, add '@EqualsAndHashCode(callSuper=false)' to your type.
```

### 解决方式
1. 在实体类所在的目录创建 *lombok.config* 文件，并添加以下内容，之后IDE编译时就不会抛出警告了
2. 参考 [Stack Overflow](https://stackoverflow.com/questions/38572566/warning-equals-hashcode-on-data-annotation-lombok-with-inheritance) ，以及 [CSDN博客](https://blog.csdn.net/feinifi/article/details/85275280)

```java
config.stopBubbling=true
lombok.equalsAndHashCode.callSuper=call
```