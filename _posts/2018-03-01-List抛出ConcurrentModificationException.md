---
title: List抛出ConcurrentModificationException
date: 2018-03-01
author: asing1elife
categories:
 - java
 - list
 - error
tags:
 - java
 - list
 - error
---
> 在对 List 进行遍历的同时进行 remove 元素操作，可能会抛出 `java.util.ConcurrentModificationException` 异常  

## 错误示范
1. 在遍历 List 时获取元素实际上通过迭代器在进行，迭代器在获取下一个元素时会对 modCount 和 expectedCount 进行匹配
2. 遍历的同时直接对 List 进行 remove 操作，会导致只有 modCount 发生变化，而expectedCount 未发生变化
3. 所以迭代器在获取下一个元素会发现两个值不匹配则抛出 `java.util.ConcurrentModificationException` 异常

```java
for (CourseDTO course : courses) {
    for (MemberCourseDTO memberCourse : memberCourses) {
        if (course.getId().equals(memberCourse.getCourse().getId())) {
            courses.remove(course);
        }
    }
}
```

## 正确处理
1. 使用迭代器原生的 remove 方法去操作 List 元素即可

```java
for (Iterator<CourseDTO> it = courses.iterator(); it.hasNext(); ) {
    CourseDTO course = it.next();

    for (MemberCourseDTO memberCourse : memberCourses) {
        if (course.getId().equals(memberCourse.getCourse().getId())) {
            // 移除当前元素
            it.remove();
        }
    }
}
```

## Java 8 的处理方式
```java
question.getOptions().removeIf(option -> option.getContent().trim().equals(""));
```