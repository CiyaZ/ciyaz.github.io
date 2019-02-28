# TimerTask 定时计划任务

在客户端编程中，定时计划任务非常常见，很多GUI框架都实现了“定时器”这种控件或是函数。比如：安卓手机的连按两次退出功能，网络流量监视器的定时刷新功能等。当然我们不借助TimerTask直接利用`Thread.sleep()`也能实现，但显然代码和逻辑都比较复杂，写到最后结果就是自己封装了一个TimerTask。所以直接使用JDK提供的TimerTask实现定时计划任务真的非常方便。

实现定时计划任务实际上需要两个类：Timer和TimerTask。

* Timer：定时器工具类。
* TimerTask：被定时器执行的计划任务。

## 间隔一段时间后执行

下面的例子展示了如何间隔一段时间后，执行某段代码。

```java
import java.util.Timer;
import java.util.TimerTask;

class MyTask extends TimerTask
{

	private Timer timer;

	MyTask(Timer timer)
	{
		this.timer = timer;
	}

	@Override
	public void run()
	{
		System.out.println("Hello");
		timer.cancel();
	}
}

public class Main
{
	public static void main(String[] args)
	{
		Timer timer = new Timer(false);
		timer.schedule(new MyTask(timer), 3000);
	}
}
```

我们继承TimerTask实现了自己的计划任务类，计划任务的代码写在了`run()`方法中。`timer.schedule()`的第二个参数是延迟时间。

这里说明几点问题：

1. Timer的构造函数中，有一个boolean参数，代表这条线程是否是守护线程。Java中守护线程指从应用启动到应用进程终止都一直循环运行的线程。上面例子中显然只是想间隔一段时间执行某段代码，因此不是守护线程。
2. 计划任务是在新线程中执行的，也就是说在delay过程中，即等待计划任务代码执行时，主线程是不阻塞的。
3. 计划任务中不要忘了调用`timer.cancel()`来终止定时器，否则进程是不会自动结束的，想想TimerTask如何实现就能明白了，因为为了实现TimerTask，进程中不只有主线程和计划任务线程。

## 循环执行

下面例子是每隔一定时间，循环执行某段计划任务代码，比如每小时备份一次数据库。

```java
import java.util.Timer;
import java.util.TimerTask;

class MyTask extends TimerTask
{

	private Timer timer;

	MyTask(Timer timer)
	{
		this.timer = timer;
	}

	@Override
	public void run()
	{
		System.out.println("Hello");
	}
}

public class Main
{
	public static void main(String[] args) throws InterruptedException
	{
		Timer timer = new Timer(false);
		timer.schedule(new MyTask(timer), 3000, 1000);
		while (true)
		{
			System.out.println("tick");
			Thread.sleep(1000);
		}
	}
}
```

注意`schedule()`的参数，第二个参数是启动计划任务前的延迟时间，第二个参数是计划任务执行的间隔时间。其实我们一般把第二个参数设为0。上面代码的运行结果：

```
tick
tick
tick
Hello
tick
Hello
tick
Hello
tick
Hello
tick
Hello
tick
...
```
