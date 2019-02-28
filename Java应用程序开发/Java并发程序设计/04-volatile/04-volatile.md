# volatile 关键字

在讲解volatile关键字之前，需要一些预先说明。

## 工作内存和主存

我们知道，计算机的存储系统分为主存，多级高速缓存和寄存器。JVM抽象的虚拟机中，分为工作内存和主存，JVM主存相当于计算机的主存，工作内存相当于高速缓存。

Java中，执行代码例如`i = 1`，值先被写入工作内存，然后同步到主存。

## 并发编程中的三个概念

### 操作原子性

JVM中，例如`i = 1`是原子操作，`i++`却不是，因为它包含读取i到寄存器，i自增，写回i到主存三步操作。

### 数据可见性

由于工作内存的存在，并发程序中会产生可见性问题。当一个线程修改了一个变量，JVM还未将其同步到主存，那么另一个线程就无法读取当前真实的变量值，而工作内存写回主存的时机是不确定的。

volatile即主要是用来解决此问题的：被volatile修饰的变量每次访存都会强制从主存同步。

注：`synchronized`中的操作也具有volatile的特性。但是为什么用volatile呢？因为同步块会阻塞尚未获得锁的线程，而volatile不会，有些情况下，出于性能考虑，根本不需要同步阻塞。例如下面代码例子：访存一个状态变量。

### 操作有序性

指令重排序是一种针对指令执行速度的优化手段。也就是说，代码实际的执行顺序可能和编写的不同，当然，存在一些规则保证了单线程运行时，和未重排序的代码执行结果一致，但是多线程下就出问题了。JVM允许指令重排序，并发程序存在有序性问题。

volatile可在一定程度上避免有序性问题：对被volatile修饰的变量的访存操作，不会在其前面代码之前执行，也不会在其后面代码之后执行。

## 一个典型的可见性问题

MyThread.java
```java
public class MyThread implements Runnable
{

	private boolean flag = true;

	public void setFlag(boolean flag)
	{
		this.flag = flag;
	}

	@Override
	public void run()
	{
		while(true)
		{
			try
			{
				Thread.sleep(100);
				System.out.println("thread running");
			}
			catch(InterruptedException e)
			{
				e.printStackTrace();
			}
			if(!this.flag)
			{
				return;
			}
		}
	}
}
```

Main.java
```java
public class Main
{
	public static void main(String[] args) throws InterruptedException
	{
		MyThread myThread = new MyThread();
		Thread t1 = new Thread(myThread);
		t1.start();
		Thread.sleep(2000);
		myThread.setFlag(false);
	}
}
```

上面代码的意图显而易见，想用flag作为线程停止的标志位，主线程2000ms后设置该标志位，线程停止。实际上，虽然上面写法一般不会出现问题，线程能够正常停止，但是它确实理论上是不对的。因为JVM出于优化考虑，主线程的改动可能根本没有写入主存，子线程就永远不会停止。

这也引出了volatile最常用的一种方式：修改状态变量。

## 对于操作有序性问题

上文介绍了volatile保证了一定程度上的操作有序性，在某些情况下这十分有用。比如主线程中初始化了两个毫不相干的两个全局变量，这两个初始化操作就可能被重排序，但是子线程对这两个变量必须按顺序操作，那就可以用上volatile了。

## 对于操作原子性问题

volatile并不能保证操作原子性问题，举个最简单的例子，两个线程同时对全局变量自增，即使用volatile，结果也是错的。只要理解了上文volatile究竟做的什么，就很好理解了。对于这种操作原子性问题，可以通过`synchronized`同步块解决。
