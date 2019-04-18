---
title: Hibernate 单表多对多关联防止 JSON 死循环
date: 2019-04-18
author: asing1elife
categories:
- java
- jpa
- springboot
- hibernate
tags:
categories:
- java
- jpa
- springboot
- hibernate
---
> Hibernate 在做实体类映射时可能存在单表的多对多关联，就是实体类关联自身的场景  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [Hibernate 双向关联转换 JSON 防止死循环](http://asing1elife.com/java/hibernate/2018/05/17/Hibernate-双向关联转换-JSON-防止死循环/)
	* 这是之前写的一篇双向一对多关联防死循环的解决方案
2. [java  – 双向多对多关系中的循环引用 - 代码日志](https://codeday.me/bug/20190104/495141.html)

## 问题描述
1. 实际情况中，可能存在以下场景，课程本身存在关联课程，在 A 课程的关联列表中可以看到 B 课程，同样的在 B 课程的关联列表中也可以看到 A 课程
2. 所以这在数据库就形成了 **A-B-A** 的关联关系
3. 因为在 **Course.java** 的中存在 `relevantCourses` 的多对多关联，所以在通过 **JPA** 获取 Course 的数据（列表或单个对象）时，就会出现 **死循环**
	* 例如当获取到 A 课程时，A 课程的 `relevantCourses` 中存在 B 课程
	* 那么 B 课程的 `relevantCourses` 又存在 A 课程，这个时候数据获取就会再次尝试第一步，就导致了死循环

```java
@Entity
@Table(name = "ht_course")
public class Course {
	@ManyToMany(cascade = {CascadeType.PERSIST}, fetch = FetchType.LAZY)
  @JoinTable(name = "ht_course_relevant", joinColumns = @JoinColumn(name = "courseId", referencedColumnName = "id"), inverseJoinColumns = 
  @JoinColumn(name = "relevantId", referencedColumnName = "id"))
  private Set<Course> relevantCourses = Sets.newHashSet();

	// getter / setter ...
}
```

## 解决方案
1. 在 [Hibernate 双向关联转换 JSON 防止死循环](http://asing1elife.com/java/hibernate/2018/05/17/Hibernate-双向关联转换-JSON-防止死循环/) 中对于双向关联是使用 `@JsonManagedReference` 和 `@JsonBackReference`  来处理父子级之间的关系
2. 但需要主要的是 `@JsonManagedReference` 可以用于列表，`@JsonBackReference` 却不行
3. 所以面对上述情况时，就算把 `@JsonManagedReference` 添加到 `relevantCourses` 上，因为没有对应的父级单个对象，`@JsonBackReference` 也无处可放

### 重写指定对象的序列化
1. 首先可以确定出现死循环的原因是因为实体类某个字段出现了循环调用
2. 那么在进行 JSON 序列化时，只需要排除这个字段即可
	* 通常使用 `@JsonIgnore`  即可，但该注解会直接在序列化时将指定字段忽略，之后不论是实体类的获取还是注入，都不会将该字段纳入范围
	* 导致的问题就是无法直接使用 JPA 本身的 **CRUD** 功能，而需要为指定字段单独编写 CRUD ，这就有点本末倒置，不可取
3. 想要排除指定字段，还可以按照以下方式，重写指定实体类的序列化过程

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.google.common.collect.Sets;
import com.innovaee.hts.admin.model.Course;

import java.io.IOException;
import java.util.Set;

public class CourseSerializer extends JsonSerializer<Set<Course>> {

    /**
     * 重写序列化
     */
    @Override
    public void serialize(Set<Course> courses, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        // 构建简版Course的集合，用于接收处于后的数据
        Set<SimpleCourse> simpleCourses = Sets.newHashSet();

        // 将传入的原始课程内容在循环中精简后加入简版课程集合
        courses.forEach(course -> simpleCourses.add(new SimpleCourse(course.getId(), course.getName())));

        // 将过滤后的数据进行回传
        jsonGenerator.writeObject(simpleCourses);
    }

    /**
     * 构建一个简版的Course实体类
     * 只包含关键数据
     * 其实想要包含其他数据也行，只需要省略掉会出现循环调用的 relevantCourses 即可
     */
    static class SimpleCourse {
        private Long id;

        private String name;

        // constructor / getter / setter ...
    }
}
```

### 在具体字段指定序列化规则
1. 在之前出现循环调用的 `relevantCourses` 字段上添加 `@JsonSerialize(using = CourseSerializer.class)` 注解即可让该字段在序列化时使用自定义的规则

```java
@ManyToMany(cascade = {CascadeType.PERSIST}, fetch = FetchType.LAZY)
@JoinTable(name = "ht_course_relevant", joinColumns = @JoinColumn(name = "courseId", referencedColumnName = "id"), inverseJoinColumns = @JoinColumn(name = "relevantId", referencedColumnName = "id"))
@JsonSerialize(using = CourseSerializer.class)
private Set<Course> relevantCourses = Sets.newHashSet();
```