---
title: Java Thread 调度
date: 2017-04-21
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> Thread 调整线程优先级，称为线程调度  
> 线程优先级越高，则会获得更多的运行机会  

## 级别
1. 优先级的取值范围是 1-10 ，但只提供 3 个常量
	* Thread.MAX_PRIORITY - 最高优先级，取值 10 
	* Thread.MIN_PRIORITY - 最低优先级，取值 1
	* Thread.NORM_PRIORITY - 默认优先级，取值 5 
2. 虽然 JVM 提供了 10 个优先级，但不推荐使用常量以外的其他优先级，因为其一致性不佳
3. 线程的优先级拥有继承关系，子类默认和父类拥有相同优先级

## 方法
1. setPriority() 设置优先级
2. getPriority() 获取优先级