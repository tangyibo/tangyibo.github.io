---
layout: post
title: Java的多线程编程与线程同步:| java
category: java
tag: [java]
---

这篇文章主要介绍了Java的多线程编程与线程同步。
本文分为以下几个部分：
1. Java的多线程编程
2. Java的线程同步方法


# 1.Java的多线程编程

一个程序就是一个进程，而一个程序中的多个任务则被称为线程。

进程是表示资源分配的基本单位，线程是进程中执行运算的最小单位，亦是调度运行的基本单位。

在Java的JDK开发包中，已经自带了对多线程技术的支持，可以很方便地进行多线程编程。创建线程的方式如下:

（1）继承Thread类,并且重写run()方法.
```
class MyThread extends Thread{

            @Override
            public void run() {
                for (int i = 0;i < 20;i++){
                    System.out.println("MyThread," + i);
                }
            }
			
}    

		Thread t=new MyThread();
		t.start();
```
（2）实现Runnable接口,并实现run()方法.
```
        Runnable runnable = () -> {
            for (int i = 0;i < 20;i++){
                System.out.println("Thread_1," + i);
            }
        };
		
		Thread t=new Thread(runnable);
		t.start();
```
（3）实现Callable接口,并实现call()方法.
```
		Callable<Boolean> f=new Callable<Boolean>() {

			@Override
			public Boolean call() throws Exception {
				System.out.println("sub thread run");
				return Boolean.TRUE;
			}
			
		};
		
		FutureTask<Boolean> task=new FutureTask<Boolean>(f);
		Thread t=new Thread(task);
		t.start();
```
使用继承 Thread 类创建线程，最大的局限就是不能多继承，所以为了支持多继承，完全可以实现 Runnable 接口的方式。

# 3.Java的线程池

Java中创建线程池有以下的方式，

（1）使用ThreadPoolExecutor类

（2）使用Executors类

其实这两种方式在本质上是一种方式，都是通过ThreadPoolExecutor类的方式。

## 4.1 使用Executors类

使用方法：
```
ExecutorService pool = Executors.newFixedThreadPool(int nThreads);
pool.submit(new TestXThread());
```

或者:
```
ExecutorService pool = Executors.newFixedThreadPool(int nThreads, ThreadFactory threadFactory);
这里的ThreadFactory接口定义如下
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
```

示例如下：
```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class TestXThread implements Runnable {

	@Override
	public void run() {
		// do something
	}
	
}

public class ThreadPool01 {

	public static void main(String[] args) {
		ExecutorService pool = Executors.newFixedThreadPool(4);
		 for (int i = 0; i < 4; i++) {
	            pool.submit(new TestXThread());
	        }
		 
		 pool.shutdown();
		while (!pool.isTerminated()) {
			try {
				Thread.sleep(50);
			} catch (Exception e) {
			}
		}

	}

}
```

## 4.2 使用ThreadPoolExecutor类

示例代码如下：
```
package cn.weishao.test;

import java.io.IOException;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadPool02 {

	public static void main(String[] args) throws InterruptedException, IOException {
		int corePoolSize = 2;
		int maximumPoolSize = 4;
		long keepAliveTime = 10;
		TimeUnit unit = TimeUnit.SECONDS;
		BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(2);
		ThreadFactory threadFactory = new NameTreadFactory();
		RejectedExecutionHandler handler = new MyIgnorePolicy();
		
		ThreadPoolExecutor executor = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit,
				workQueue, threadFactory, handler);
		executor.prestartAllCoreThreads(); // 预启动所有核心线程

		for (int i = 1; i <= 10; i++) {
			MyTask task = new MyTask(String.valueOf(i));
			executor.execute(task);
		}

		System.in.read(); // 阻塞主线程
	}

	static class NameTreadFactory implements ThreadFactory {

		private final AtomicInteger mThreadNum = new AtomicInteger(1);

		@Override
		public Thread newThread(Runnable r) {
			Thread t = new Thread(r, "my-thread-" + mThreadNum.getAndIncrement());
			System.out.println(t.getName() + " has been created");
			return t;
		}
	}

	public static class MyIgnorePolicy implements RejectedExecutionHandler {

		public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
			doLog(r, e);
		}

		private void doLog(Runnable r, ThreadPoolExecutor e) {
			// 可做日志记录等
			System.err.println(r.toString() + " rejected");
			// System.out.println("completedTaskCount: " + e.getCompletedTaskCount());
		}
	}

	static class MyTask implements Runnable {
		private String name;

		public MyTask(String name) {
			this.name = name;
		}

		@Override
		public void run() {
			try {
				System.out.println(this.toString() + " is running!");
				Thread.sleep(3000); // 让任务执行慢点
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

		public String getName() {
			return name;
		}

		@Override
		public String toString() {
			return "MyTask [name=" + name + "]";
		}
	}
}

```

# 4.Java的多线程同步

Java之多线程同步方法：

（1）synchronized    关键字

（2）Lock            接口类

## 4.1 synchronized关键字实现线程同步

### （1）synchronized关键词修饰同步代码块

使用方法：
```
synchronized(非匿名的任意对象){
    线程要操作的共享数据
}
```

示例代码如下：
```
class TicketConsumer1  implements Runnable{
	
	private int ticket;
	
	public TicketConsumer1(int t) {
		this.ticket=t;
	}

	@Override
	public void run() {

		while (true) {

			synchronized (this) {
				if (ticket > 0) {
					System.out.println("thread " + Thread.currentThread().getName() + " left ticket:" + ticket);
					ticket--;
				}else {
					break;
				}
			}

			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
	}
	
}

public class BasicTest04 {

	public static void main(String[] args) throws InterruptedException {
		Runnable runable=new TicketConsumer1(10);
		Thread t1=new Thread(runable);
		Thread t2=new Thread(runable);
		Thread t3=new Thread(runable);
		
		t1.start();
		t2.start();
		t3.start();
		
		t1.join();
		t2.join();
		t3.join();
		
		System.out.println("main thread over!");
	}

}
```

### （2）synchronized关键词修饰同步函数方法【推荐方式】

将线程共享数据，同步，抽取到一个方法中，在方法的声明处加入同步关键字,使用方法：
```
void synchronized shareFunction(){
    critical section
}
```

示例代码如下：
```
class TicketConsumer2  implements Runnable{
	
	private int ticket;
	
	public TicketConsumer2(int t) {
		this.ticket=t;
	}

	private synchronized boolean sale() {
		if(ticket > 0) {
				System.out.println("thread " + Thread.currentThread().getName() + " left ticket:" + ticket);
				ticket--;
				return true;
		}
		
		return false;
	}
	
	@Override
	public void run() {
			while (true) {
				if(!this.sale()) {
					break;
				}
				
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
	}
	
}


public class BasickTest05 {

	public static void main(String[] args) throws InterruptedException {
		Runnable runable=new TicketConsumer2(10);
		Thread t1=new Thread(runable);
		Thread t2=new Thread(runable);
		Thread t3=new Thread(runable);
		
		t1.start();
		t2.start();
		t3.start();
		
		t1.join();
		t2.join();
		t3.join();
		
		System.out.println("main thread over!");

	}

}
```

对synchronized的补充：

如果拿到synchronized的线程异常退出了，那么等待锁的线程是否会一直等待呢？

答案是否定的，当JVM发现有锁的线程异常了之后会将它的锁自动释放，再由其它等待的线程拿到锁。

**使用synchronized的缺陷：**

（1）如果获取锁的线程由于要等待IO或者其它原因（比如sleep）被阻塞了，但是又没有释放锁，其它线程便只能等待，这样非常影响程序执行的效率。因此就需要一种机制：可以不让线程一直无期限等待下去（比如只等待一定时间或者能够相应中断），通过Lock就可以办到。

（2）又假设当多个线程读写文件时，read-write会发生冲突现象，write-write会发生冲突，但是read-read不会发生冲突。如果采用synchronized，就不能让read-read同时进行，只要有一个线程read，其他想read的线程都只能等待，严重影响效率。这就需要一种机制：使得多个线程都只是read时，线程之间不会发生冲突，通过Lock就可以办到。

（3）另外通过Lock可以知道线程有没有成功获取到锁，这个是synchronized无法办到的。

**synchronized同步的原理:**

Synchronized进过编译，会在同步块的前后分别形成monitorenter和monitorexit这个两个字节码指令。

在执行monitorenter指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止。

## 4.2 Lock接口类实现线程同步

## 4.2.1 Lock接口

（1）Lock接口定义

```
public interface Lock {

    /**
     * 获取普通锁，如果锁已被获取，则只能等待，功能等同于synchronized关键字。但不同的是Lock后必须unLock锁，一般来说，Lock必须在try{}catch{}中进行，并
	 * 且将释放锁的操作放在finally块中进行，以保证锁一定被释放，防止死锁的发生。
     */
    void lock();

    /**
     * 当2个线程同时通过lock.lockInterruptibly()想获取某个锁时，假设A线程获取到了锁，那B线程只能等待，那么对B线程调用threadB.interrupt()方法能够终
	 * 端B线程的等待过程。注意，当A线程获取了锁后，是不会被interrupt()方法中断的；因此通过lockInterruptibly()方法获取锁时，如果没有获取到，只能等待，只
	 * 有等待状态下的线程才是可以响应中断的！
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 尝试获取锁，如果获取成功返回true，反之返回false。也就是说，这个方法无论如何都不会阻塞等待获取锁
     */
    boolean tryLock();

    /**
     *  等待time时间，如果在time时间内获取到锁返回true，如果阻塞等待time时间内没有获取到锁返回false
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 业务处理完毕，释放锁
     */
    void unlock();

    /**
     * 返回一个新的Condition实例
     */
    Condition newCondition();
}
```

（2）Lock接口的实现

```
reentrant	英[riːˈɛntrənt] 美[ˌriˈɛntrənt]
解释：可重入; 可重入的; 重入; 可再入的; 重进入;
```

- ReentrantLock类

使用方法：
```
 Lock lock=new ReentrantLock();
 
 lock.lock();
 try {
 	//do something
 } finally {
 	lock.unlock();
 }
```

ReenTrantLock的实现是一种自旋锁，通过循环**调用CAS操作来实现加锁**。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。

深入分析参考：http://www.blogjava.net/zhanglongsr/articles/356782.html

使用示例如下：
```

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class TicketConsumer3  implements Runnable{
	
	private int ticket;
	private Lock lock;
	
	public TicketConsumer3(int t) {
		this.ticket=t;
		lock=new ReentrantLock();
	}

	private boolean sale() {
		lock.lock();

		try {
			if (ticket > 0) {
				System.out.println("thread " + Thread.currentThread().getName() + " left ticket:" + ticket);
				ticket--;
				return true;
			}

			return false;
		} finally {
			lock.unlock();
		}

	}
	
	@Override
	public void run() {
			while (true) {
				if(!this.sale()) {
					break;
				}
				
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
	}
	
}


public class BasicTest06 {

	public static void main(String[] args) throws InterruptedException {
		Runnable runable=new TicketConsumer3(10);
		Thread t1=new Thread(runable);
		Thread t2=new Thread(runable);
		Thread t3=new Thread(runable);
		
		t1.start();
		t2.start();
		t3.start();
		
		t1.join();
		t2.join();
		t3.join();
		
		System.out.println("main thread over!");

	}

}
```

- ReadLock类

 （略）
 
- WriteLock类

 （略）

## 4.2.2 ReadWriteLock接口

（1）ReadWriteLock接口定义

```
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

（2）ReadWriteLock接口的实现

- ReentrantReadWriteLock类

示例请参考地址：https://www.iteye.com/blog/cuisuqiang-1458806

# 5.Java的线程间通信

### synchronized加Object类的wait/notify方式

### ReentrantLock加条件变量Condition方式

### 管道通信PipedOutputStream/PipedInputStream

管道流主要用来实现两个线程之间的二进制数据的传播，下面以PipedInputStream类和PipedOutputStream类为例，实现生产者-消费者：

```
import java.io.IOException;
import java.io.PipedInputStream;
import java.io.PipedOutputStream;

class MyProducer extends Thread {
	private PipedOutputStream outputStream;

	public MyProducer(PipedOutputStream outputStream) {
		this.outputStream = outputStream;
	}

	@Override
	public void run() {
		int index = 0;
		
		while (true) {
			try {
				for (int i = 0; i < 5; i++) {
					outputStream.write(index++);
				}
			} catch (IOException e) {
				e.printStackTrace();
			}

			try {
				Thread.sleep(5000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

class MyConsumer extends Thread {

	private PipedInputStream inputStream;

	public MyConsumer(PipedInputStream inputStream) {
		this.inputStream = inputStream;
	}

	@Override
	public void run() {
		while (true) {
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			try {
				int count = inputStream.available();
				if (count > 0) {
					System.out.println("rest product count: " + count);
					System.out.println("get product: " + inputStream.read());
				}
			} catch (IOException e1) {
				e1.printStackTrace();
			}
		}
	}
}

public class ThreadCommunicate01 {

	public static void main(String[] args) {
		PipedOutputStream pos = new PipedOutputStream();
		PipedInputStream pis = new PipedInputStream();
		try {
			pis.connect(pos);
		} catch (IOException e) {
			e.printStackTrace();
		}

		new MyProducer(pos).start();
		new MyConsumer(pis).start();
	}

}
```
