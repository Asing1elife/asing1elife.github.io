---
title: Java Thread 状态转换
date: 2017-04-18
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> 下图对 Thread 的所有状态进行流程描述  

![](http://asing1elife.com/sources/images/41D2A69F-6C73-43B0-9378-13244C732564.png)

## 分解
1. 初始 - 新创建一个线程对象
2. 就绪- 对象的 start() 被调用
	* 处于该状态的线程位于**可运行线程池**中，等待获取 CPU 使用权
3. 运行 - 执行程序代码
4. 阻塞 - 因为某种原因放弃 CPU 使用权，暂时停止运行
	* 处于该状态的线程直到重新进入就绪状态，才有机会回到运行状态
	* 该状态具体分为三种
		1. 等待 - 该线程执行 wait() ，JVM 将该线程放入等待池中，并释放持有的锁
		2. 同步 - 该线程在获取对象同步锁时，发现该同步锁被占用，JVM 则将该线程放入锁池中
		3. 其他 - 该线程执行 sleep() 或 join() 或发出 I/O 请求，JVM 将该线程改为阻塞状态
5. 死亡 - 线程执行结束或因抛出异常导致退出 run() 