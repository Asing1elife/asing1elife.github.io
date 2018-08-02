---
title: Java 实现 Runnable 接口
date: 2017-04-17
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> 与 [Java 继承 Thread 类](http://asing1elife.com/java/multithreading/2017/04/17/Java-继承-Thread-类/) 一样，实现 Runnable 接口也是启动 Java 线程的一种方式  
> 通过继承 Thread 类实现多线程的对象，不适合资源共享，而实现 Runnable 接口，则适合资源共享  
> 不论是通过继承 Thread 类或者实现 Runnable 接口来实现多线程，最终都是通过 Thread 的 API 控制线程  

## 注意
1. **run()** 是多线程程序的一个约定，所有的多线程代码都在其中执行
2. 在启动多线程时，首先需要通过 **Thread(Runnable target)** 构造出线程对象，再调用 **start()** 运行多线程
3. **所有的多线程代码都是通过 Thread 的 start() 来运行**

## 优势 - Runnable 相较于 Thread
1. 适合拥有多个相同程序代码的线程去处理同一资源
2. 可以避免 Java 中的单继承限制
3. 代码可以被多个线程共享
4. 代码和数据实现独立
5. 增加程序的健壮性
6. 线程池只能放入实现 Runnable 或 Callable 的类，不能直接放入继承 Thread 的类

## 实现
```java
public class RunnableTest implements Runnable {
	private String name;

	public RunnableTest(String name) {
		this.name = name;
	}

	public void run() {
		for (int i = 1; i <= 5; i++) {
			System.out.println(name + " 运行 " + i + " 次");

			try {
				Thread.sleep(1000);
			} catch (InerruptedException e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		RunnableTest tom = new RunnableTest("tom");
		RunnableTest jerry = new RunnableTest("jerry");

		new Thread(tom).start();
		new Thread(jerry).start();
	}
}
```