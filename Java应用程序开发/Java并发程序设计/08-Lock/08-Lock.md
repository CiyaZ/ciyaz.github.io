# Lock的使用

Java5提供了Lock（锁）对象，也能实现同步的效果，使用上更加方便。

## 可重入锁 ReentrantLock

`synchronized`能够实现线程同步，可重入锁也能实现，而且更加灵活。可重入锁的使用方法和Linux中互斥锁十分相似。但为什么叫可重入锁呢？因为和互斥锁不同的是，同一条线程下，可重入锁在外层函数锁定后，内层函数依然能再次锁定，这样能够避免一种死锁情况，实际上`synchronized`也是可重入的。下面直接看一个例子：

MyThread.java
```java
public class MyThread implements Runnable
{
	private Lock lock = new ReentrantLock();

	@Override
	public void run()
	{
		System.out.println("thread " + Thread.currentThread().getId() + " start");
		lock.lock();
		for(int i = 0; i < 10; i++)
		{
			try
			{
				Thread.sleep(10);
			}
			catch (InterruptedException e)
			{
				e.printStackTrace();
			}
			System.out.println("Thread " + Thread.currentThread().getId());
		}
		lock.unlock();
	}
}
```

Main.java
```java
public class Main
{
	public static void main(String[] args)
	{
		MyThread myThread = new MyThread();
		Thread th1 = new Thread(myThread);
		Thread th2 = new Thread(myThread);
		Thread th3 = new Thread(myThread);
		Thread th4 = new Thread(myThread);
		Thread th5 = new Thread(myThread);
		th1.start();
		th2.start();
		th3.start();
		th4.start();
		th5.start();
	}
}
```

我们创建了5条线程，他们持有同一个Lock，使用`lock()`函数和`unlock()`函数锁定和解锁Lock对象。Lock已经被一个线程锁定时，另一个线程也想获得锁，就只能阻塞等待了。

## 锁结合Condition实现等待/通知机制

一个Lock对象可以创建多个Condition实例，线程对象可以注册在指定Condition中，这样可以实现有选择性的线程通知，调度上更加灵活。而`notify`/`notifiAll`中，如果有多个对象正在等待，通知的对象是随机的，这不太好控制。

* `await()` 当前线程阻塞等待
* `signal()` 唤醒一个线程
* `signalAll()` 唤醒多个线程

使用Condition例子

MyService.java
```java
public class MyService
{
	private Lock lock = new ReentrantLock();
	private Condition condition1 = lock.newCondition();
	private Condition condition2 = lock.newCondition();
	public void awaitA()
	{
		lock.lock();
		try
		{
			System.out.println("awaitA");
			condition1.await();
			System.out.println("signalA");
		}
		catch (InterruptedException e)
		{
			e.printStackTrace();
		}
		finally
		{
			lock.unlock();
		}
	}
	public void awaitB()
	{
		lock.lock();
		try
		{
			System.out.println("awaitB");
			condition2.await();
			System.out.println("signalB");
		}
		catch (InterruptedException e)
		{
			e.printStackTrace();
		}
		finally
		{
			lock.unlock();
		}
	}
	public void signalA()
	{
		lock.lock();
		try
		{
			condition1.signalAll();
		}
		finally
		{
			lock.unlock();
		}
	}
	public void signalB()
	{
		lock.lock();
		try
		{
			condition2.signalAll();
		}
		finally
		{
			lock.unlock();
		}
	}
}
```

MyThreadA.java
```java
public class MyThreadA implements Runnable
{
	private MyService myService;

	public MyThreadA(MyService myService)
	{
		this.myService = myService;
	}

	@Override
	public void run()
	{
		myService.awaitA();
	}
}
```

MyThreadB.java
```java
public class MyThreadB implements Runnable
{
	private MyService myService;

	public MyThreadB(MyService myService)
	{
		this.myService = myService;
	}

	@Override
	public void run()
	{
		myService.awaitB();
	}
}
```

Main.java
```java
public class Main
{
	public static void main(String[] args) throws InterruptedException
	{
		MyService myService = new MyService();
		Thread th1 = new Thread(new MyThreadA(myService));
		Thread th2 = new Thread(new MyThreadB(myService));

		th1.start();
		th2.start();

		Thread.sleep(1000);
		myService.signalA();
		Thread.sleep(1000);
		myService.signalB();
	}
}
```

上述代码中，我们使用同一个Lock对象创建了两个Condition对象，在其上分别调用`signalAll()`（实际上可以看成两组线程），实现了同一个锁两组线程分别唤醒。

## 公平锁和非公平锁

`ReentrantLock`构造函数可以传入一个`boolean`类型参数，表示这个锁是公平锁还是非公平锁。公平锁表示线程获取锁的顺序是按照加锁的顺序分配的，非公平锁表示获取锁是抢占式的，根据线程优先级作为参考，实际上是随机的。

## 可重入锁其他方法

* `getHoldCount()` 查询当前线程保持此锁的个数（即调用lock的次数）
* `getQueueLength()` 返回等待此锁的线程估计数
* `getWaitQueueLength()` 返回等待此锁上某个条件的线程估计数

* `hasQueuedThread(Thread th)` 查询指定线程是否正在等待获取此锁
* `hasQueuedThreads()` 查询是否有线程正在等待获取此锁
* `hasWaiters(Condition con)` 查询指定条件下是否有线程正在等待

* `isFair()` 返回是不是公平锁
* `isHeldByCurrentThread()` 查询当前线程是否保持此锁
* `isLocked()` 查询该锁是否被任意线程锁定

* `lockInterruptibly()` 如果当前线程未被中断，则获取锁定，如果已被中断则出现运行时异常（同步块或获得可重入锁的代码是不能中断的）
* `tryLock()` 调用时，如果该锁已被其他线程持有，就不取获取继续执行下一条代码，未被其他线程持有，就获取该锁
* `tryLock(long timeout, TimeUnit unit)` 给定时间内，锁没有被另一个线程获取，且当前线程未中断，就获取该锁

# 可重入读写锁ReentrantReadWriteLock

可重入锁是完全互斥的，有时我们不需要读写都是互斥的，使用可重入读写锁能够提升线程的运行效率。读写锁分为共享锁（读锁），排它锁（写锁），共享锁之间不互斥，共享锁和排它锁互斥，排它锁和排它锁互斥。

锁定共享锁或排它锁使用如下语句：

```java
reentrantReadWriteLock.readLock().lock();
reentrantReadWriteLock.writeLock().lock();
```
