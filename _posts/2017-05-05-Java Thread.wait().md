---
title: Java Thread.wait()
date: 2017-05-05
author: asing1elife
categories:
 - java
 - multithreading
tags:
 - java
 - multithreading
---
> wait() 和 notify() 必须在 synchronized 语句块中使用  
> wait() 是强迫一个线程等待  
> notify() 是通知一个线程继续运行  

## 功能
1. wait() 是针对已经获取对象锁的线程进行操作
2. 当线程获取对象锁后，调用 wait() 主动释放对象锁，同时该线程休眠
3. 直到其他线程调用 notify() 唤醒该线程，才继续获取对象锁，并执行
4. 调用 notify() 唤醒线程并不是实时的，而是等相应的 synchronized 语句块执行结束，自动释放对象锁
5. 再由 JVM 选取休眠的线程赋予对象锁，并执行，从而实现线程间同步、唤醒的操作

## 对比
1. wait() 和 sleep() 都可以通过 interrupt() 打断线程的暂停状态，从而使线程立刻抛出 **InterruptedException**
	1. 通过 interrupt() 打断线程时，只需在 catch() 中捕获到异常即可安全结束线程
	2. InterruptedException 是线程内部抛出，并不是 interrupt() 抛出
	3. 当线程执行普通代码时调用 interrupt() 并不会抛出异常，只有当线程进入 wait() / sleep() / join() 后才会立即抛出
2. wait() 和 sleep() 都可以暂定当前线程，其区别在于 wait() 在暂定的同时释放了对象锁
3. sleep() 是 Thread 的静态方法，wait() 是 Object 的一般方法

## 实现
```java
public class WaitTest extends Thread {

    private final Object self;

    private final Object last;

    public WaitTest(String name, Object self, Object last) {
        super(name);
        this.self = self;
        this.last = last;
    }

    public void run() {
        for (int i = 0; i < 10; i++) {
            // 锁住下一个对象
            synchronized (last) {
                // 锁住当前对象
                synchronized (self) {
                    if (super.getName().equals("A")) {
                        System.out.println("第 " + (i + 1) + " 次运行！");
                    }

                    System.out.println(super.getName());

                    // 等待一轮循环结束后唤醒当前线程
                    self.notify();
                }

                try {
                    // 释放下一个线程的对象锁
                    last.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();

        WaitTest waitA = new WaitTest("A", a, b);
        WaitTest waitB = new WaitTest("B", b, c);
        WaitTest waitC = new WaitTest("C", c, a);

        waitA.start();
        waitB.start();
        waitC.start();
    }
}
```