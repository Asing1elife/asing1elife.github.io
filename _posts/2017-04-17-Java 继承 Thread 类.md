---
title: Java 继承 Thread 类
date: 2017-04-17
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> 如果只是启动一个线程，则可通过继承 Thread 类来实现  
> 但一般推荐使用 Runnable - [Java 实现 Runnable 接口](http://asing1elife.com/java/multithreading/2017/04/17/Java-实现-Runnable-接口/)  
> Thread 类实际上也是实现了 Runnable 接口  

## 注意
1. 程序运行时，Java 虚拟机启动一个进程，主线程在 main() 调用时被创建
2. 在 **main()** 中调用 ThreadTest 的两个对象，则启动两个线程
3. **start()** 的调用不会立即执行多线程代码，而是促使该线程变为可运行态，具体运行时间由操作系统决定
4. 多线程是乱序执行的，每次执行的结果都不确定
5. 在多线程中调用 **sleep()** 的目的是不让当前线程占用全部系统资源
6. 如果一个对象的 start() 被重复调用，则会抛出 **java.lang.IllegalThreadStateException** 异常

## 实现
```java
public class ThreadTest extends Thread {
	private String name;

	public ThreadTest (String name) {
		this.name = name;
	}

	public void run() {
		for (int i = 1; i <= 5; i++) {
			System.out.println(name + " 运行 " + i + " 次");

			try {	
				// 单位毫秒
				sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		ThreadTest tom = new ThreadTest("tom");
		ThreadTest jerry = new ThreadTest("jerry");

		tom.start();
		jerry.start();
	}
}
```