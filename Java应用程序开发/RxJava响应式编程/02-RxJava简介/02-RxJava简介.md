# RxJava

RxJava是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。 Rx即ReactiveExtensions，是由微软提出的一个综合了异步和基于事件驱动编程类库，使用可观察序列和LinQ-style查询操作，RxJava是其Java实现。

总的来说，RxJava可以简化和异步编程相关的方方面面，在Android和其他客户端开发中应用十分广泛，但是上手难度较高。

## 观察者和被观察者

上一篇笔记介绍了回调函数，其中button可以看做一个被观察者，button被点击一次，就会发送一个事件，而处理事件的回调函数就是观察者。在RxJava中，有两个对象：Observer和Obervable，对应观察者和被观察者。两者建立起观察/被观察的关系的操作，叫做「订阅(subscribe)」。我们现在使用RxJava分别创建一个Observer和Observable，并使用被观察者订阅观察者：

```java
//创建被观察者
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>()
{
	@Override
	public void call(Subscriber<? super String> subscriber)
	{
		subscriber.onNext("hello1");
		subscriber.onNext("hello2");
		subscriber.onNext("hello3");
		subscriber.onCompleted();
	}
});
//创建观察者
Subscriber<String> subscriber = new Subscriber<String>()
{
	@Override
	public void onCompleted()
	{
		System.out.println("completed");
	}

	@Override
	public void onError(Throwable throwable)
	{
	}

	@Override
	public void onNext(String s)
	{
		System.out.println(s);
	}
};
//订阅
observable.subscribe(subscriber);
```

### 代码说明

* Observable：传入了一个函数，包含了向Observer发送的事件。
* Subscriber：Subscriber是Observer的子抽象类，实际使用中建议使用Subscriber，其中实现的onNext()等方法，代表事件回调，事件队列完成回调，发生错误回调。

运行结果
```
hello1
hello2
hello3
completed
Process finished with exit code 0
```

## 术语总结

* Oberver（观察者）：决定事件触发时的行为。
* Observable（被观察者，事件源）：决定什么时候触发事件以及触发怎样的事件。
* subscribe（订阅）：连接Observer和Observable，Observable订阅Observer。
