---
title: Spring JPA 保存实体后无法拿到对应实体类的 ID
date: 2019-06-12
author: asing1elife
categories:
- springboot
- jpa
- java
tags:
- springboot
- jpa
- java
---
> 使用 JPA 进行对象持久化操作后，有时候希望能到新增的对象 ID ，以便进行下一步操作，如果出现拿不到 ID 的情况，解决方式如下  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 网上现存的一大堆错误的解决方式

### 将 ID 自增的注解标记在 GET 方法上
1. 网上比较多的说法是说 JPA 存在一个 BUG ，如果不把 ID 新增的注解标记在 GET 方法上，如以下代码
2. 那么 JPA 就无法识别到自增的标记，也就不会为新增的实体类赋值最新的 ID
3. 实测这么做不但无效，反而会带来另一个 BUG ： `org.hibernate.MappingException: Could not determine type for:`
	* 这个 BUG 的意思就是说注解要么全部标记在字段上，要么全部标记在 GET 方法上，不能混合标记

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
public Long getId() {
    return id;
}
```

### 将原生的 save() 方法，修改为 saveAndFlush() 方法
1. 实测无效，没有任何效果

## 正确的解决方式
1. [JPA保存记录无法获取保存后记录的ID - @洋 - ITeye博客](https://lpyyn.iteye.com/blog/2065949) 在这个博客中找到了具体的错误原因
2. 博客里的意思就是说，如果实体类的 ID 默认不是空值，JPA 在执行新增操作后，就不会为该实体类的 ID 重新赋值
3. 检查自己的代码，发现在基类的 ID 上，确实添加了 `private Long id = 0L;` 的默认值，删掉之后再次执行新增操作，实体类就会被赋值正确的新 ID，代码如下

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```