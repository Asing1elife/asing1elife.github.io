---
title: Java Thread join()
date: 2017-04-21
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> join() 是 Thread 提供的一个方法，线程启动后可直接调用  
> join() 是等待线程终止  

## 功能
1. 线程调用 join() 后，主线程必须等待该线程终止后，主线程才能终止

## 场景
1. 当主线程调用子线程后，子线程需要大量的耗时运算
2. 由于多线程的特性，往往导致主线程会在子线程之前结束
3. 那么主线程结束之前就无法获取到子线程的运算结果
4. 当子线程调用 join() 后，会强制主线程必须等待子线程终止后，主线程才能终止
5. 从而主线程就可以获取到子线程的运算结果

## 实现
1. 主线程的结束语句一定会等两个子线程都结束后才会运行

```java
public JoinTest implements Runnable {
	private String name;

	public JoinTest(String name) {
		this.name = name;
	}

	public void run() {
		System.out.println(name = " 线程开始运行！");

		for (int i = 1; i <= 5; i++) {
			System.out.println(name = " 线程运行 " + i + " 次！");

			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}		

		System.out.println(name = " 线程运行结束！");
	}

	public static void main(String[] args) {
		System.out.println("主线程开始运行！\n");

		JoinTest tom = new JoinTest("tom");
		JoinTest jerry = new JoinTest("jerry");

		Thread tomThread = new Thread(tom);
		Thread jerryThread = new Thread(jerry);

		tomThread.start();
		jerryThread.start();

		try {
			tomThread.join();
			jerryThread.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		System.out.println("\n主线程运行结束！");
	}
}
```