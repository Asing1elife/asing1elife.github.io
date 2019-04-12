---
title: SpringBoot+JPA 问题汇总
date: 2019-02-12
author: asing1elife
categories:
- java
- springboot
- jpa
tags:
- java
- springboot
- jpa
---
> 以下会列出SpringBoot在继承JPA时通常会碰到一些配置问题  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 实体类有继承父类，但父类没有单独标明注解

### 异常表现
```java
Caused by: org.hibernate.AnnotationException: No identifier specified for entity: com.xxx.ProjectDTO
```

### 解决方式
1. 可以看到*ProjectDTO*有继承一个*BaseDTO* ，那么在父类中肯定存在某些字段需要与数据库表字段对应
2. 因此父类需要使用 `@MappedSuperclass` 标注为映射的父类，即可解决上述问题

```java
@Entity
@Table(name = "al_project")
public class ProjectDTO extends BaseDTO { ... }

@MappedSuperclass
public class BaseDTO { ... }
```

## 自定义接口实现了JpaRepository ，但没有单独标明注解

### 异常表现
```java
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'baseJpaRepository': Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: Not a managed type: class java.lang.Object
...
Caused by: java.lang.IllegalArgumentException: Not a managed type: class java.lang.Object
```

### 解决方式
1. 上述报错中提到*BaseJpaRepository*创建失败，其实是因为该接口继承*JpaRepository* ，继承 *CrudRepository* 同理
2. 导致SpringBoot把该类认为是Jpa的某一个Repository ，所以需要添加 `@NoRepositoryBean` 告知SpringBoot该类不是一个Repository

```java
@NoRepositoryBean
public interface BaseJpaRepository<T, ID> extends JpaRepository<T, ID> { ... }
```

## 实体类创建后没有单独标明注解@Entity

### 异常表现
1. 需要注意的是这个错误可能不完全以下内容

```java
***************************
APPLICATION FAILED TO START
***************************
...
The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)
...
````

### 解决方式
1. 检查所有的实体类，并全部添加上 `@Entity` 注解
2. 需要注意的是以下实体类*ConfigDTO*继承了*BaseDTO* ，而*BaseDTO* 作为父类不应该使用 `@Entity` 注解
3. 而应该使用上述问题中提到的 `@MappedSuperclass` 标明是映射父类

```java
@Entity
@Table(name = "al_config")
public class ConfigDTO extends BaseDTO { ... }
```

## 实体类中的某个字段是一个对象，却错误的使用了@Column注解

### 异常表现
```java
Caused by: org.hibernate.MappingException: Could not determine type for: com.asing1elife.teamnote.dto.ProjectGroupDTO, at table: al_project, for columns: [org.hibernate.mapping.Column(project_group)]
```

### 解决方式
1. 以下类中*ProjectGroupDTO*应该是一个对象，所以应该根据具体映射情况决定是使用 `@ManyToOne` 还是 `@OneToMay` ，或其他对象映射规则

```java
@Entity
@Table(name = "al_project")
public class ProjectDTO extends BaseDTO {
    @Column
    private ProjectGroupDTO projectGroup;
}
```

## 实体类ID使用了错误的生成规则

### 异常表现
```java
java.sql.SQLSyntaxErrorException: Table 'asl_station.hibernate_sequence' doesn't exist
```
```java
public class BaseDTO {
    @Id
    @GeneratedValue
    private Long id = 0L;
}
```

### 解决方式
1. 上述类中在*id*上通过 `@GeneratedValue` 直接指定生成规则，就会抛出上述异常
2. 因为默认的生成规则是 `GeneratedType.AUTO` 
3. 而我们在创建表时，通常使用的*id*自增规则都是 `auto_increment` ，这对应的应该是 `GenerationType.IDENTITY`
4. 所以应该使用如下方式指定*id*的自增规则

```java
public class BaseDTO {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id = 0L;
}
```

## 修改JPA的字段命名规则为驼峰式

### 异常表现
1. 在实体类和数据库中对应的字段都是*createTime* ，但是JPA在连接对应表时却抛出如下错误
2. 因为JPA默认的字段映射规则是下划线风格

```java
Caused by: java.sql.SQLSyntaxErrorException: Unknown column 'create_time' in 'field list'
```
```java
@Column
private Date createTime = DateUtil.getSysDate();
```

### 解决方式
1. 在配置文件中指定JPA的字段映射规则为驼峰式即可

```yaml
spring:
  jpa:
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

## 获取单个实体类JSON转换异常

### 异常表现
1. Hibernate获取单个实体类数据后，会为每个实体类添加一个 `hibernateLazyInitializer` 属性，改属性在进行JSON转换时抛出异常

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: com.asing1elife.teamnote.core.bean.ResponseData["data"]->com.asing1elife.teamnote.dto.OrganizationDTO$HibernateProxy$f3Tr4vBI["hibernateLazyInitializer"])
```

### 解决方式一
1. 在实体类顶部添加 `@JsonIgnoreProperties("hibernateLazyInitializer")` ，确保数据转换时会过滤掉Hibernate生成的属性

```java
@JsonIgnoreProperties("hibernateLazyInitializer")
public class BaseDTO { ... }
```

### 解决方式二
1. 根据上述报错 `disable SerializationFeature.FAIL_ON_EMPTY_BEANS` 可以尝试将JSON转换时的以下属性设置为 `false`

```yaml
spring:
  jackson:
    serialization:
      fail-on-empty-beans: false
```

## 前端数据传递到后端过程中，反序列失败

### 异常表现
1. *jackson* 反序列化时需要无参构造函数，因为数据对应的实体类某个对象属性没有无参构造函数，就会抛出以下异常

```java
(although at least one Creator exists): cannot deserialize from Object value (no delegate- or property-based Creator)
```

### 解决方式
1. 检查实体类自身以及所有对象属性，为每个对象添加对应的无参构造函数即可解决
2. 参考自 [博客园](https://www.cnblogs.com/yucfeng/p/8932089.html)