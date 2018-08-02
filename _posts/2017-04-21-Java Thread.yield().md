---
title: Java Thread.yield()
date: 2017-04-21
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> yield() 是 Thread 提供的一个**静态方法**，可直接通过 Thread 调用  
> yield() 是提醒线程暂停  

## 功能
1. 暂停当前正在执行的线程对象，并执行其他线程

## 特点
1. 让当前运行线程回到**可运行**状态，从而让拥有相同优先级的线程获取运行机会
2. 其目的是让拥有相同优先级的线程之前能适当的轮转运行
3. 实际上 yeld() 的可行性无法得到保证，因为回到可运行状态的线程依旧有可能有限被调度程序选中
4. yeld() 不会导致线程回到**等待/睡眠/阻塞**状态，所以对线程执行该方法后可能没有效果

## 对比 
1. sleep() 放线程进行停滞状态，导致线程在指定时间内停止运行
2. yield() 只是让线程回到可运行状态，但重新获取运行权的可能还是这个线程
3. sleep() 允许优先级较低线程在当前线程停滞后获取运行权
4. yield() 不可能让比当前线程优先级低的线程获取运行权

## 实现
```java
public YieldTest extends Thread {
	public YieldTest (String name) {
		super(name);
	}

	public void run() {
		for (int i = 1; i <= 5; i++) {
			System.out.println(this.getName() + " 运行 " + i + " 次");

			if (i == 3) {
				this.yield();
			}
		}
	}

	public static void main(String[] args) {
		YieldTest tom = new YieldTest("tom");
		YieldTest jerry = new YieldTest("jerry");

		tom.start();
		jerry.start();
	}
}
```