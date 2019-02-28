# BroadcastReceiver

BroadcastReceiver可以用来接收其他组件发送的广播，广播是一个系统级的消息传递机制。

## 有序广播和无序广播

某个应用发出有序广播后，这个广播会被匹配的广播接收者按照优先级的顺序依次接收，广播接收者可以让这个广播停止继续传递。无序广播则可以理解为是所有匹配的接受者一起收到这个广播。

## 发送接收广播的简单例子

下面演示最简单的从`Activity`中发送一个无序广播，然后编写一个`BroadcastReceiver`进行接收，然后使用Log打印出来。

MainActivity.java
```java
public class MainActivity extends Activity
{

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}

	public void send(View view)
	{
		Intent intent = new Intent("android.intent.action.MyBroadcastReceiver");
		intent.putExtra("msg", "test broadcast");
		sendBroadcast(intent);
	}
}
```

发送广播，我们也需要一个Intent对象，Intent对象中可以带有数据，这样广播接收者就能接收其中的数据，实现数据传递。

MyBroadcastReceiver.java

```java
public class MyBroadcastReceiver extends BroadcastReceiver
{

	@Override
	public void onReceive(Context context, Intent intent)
	{
		String text = intent.getStringExtra("msg");
		Log.i("msg", text);
	}
}
```

BroadcastReceiver组件比较简单，我们只需要重写`onReceive()`方法，其参数有一个`Intent`对象，这个就是发送广播时发送的Intent，广播接收者可以从其中读取广播中的数据。

AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest package="com.ciyaz.broadcastdemo"
		  xmlns:android="http://schemas.android.com/apk/res/android">

	<application
		android:allowBackup="true"
		android:icon="@mipmap/ic_launcher"
		android:label="@string/app_name"
		android:supportsRtl="true"
		android:theme="@style/AppTheme">
		<activity android:name=".MainActivity">
			<intent-filter>
				<action android:name="android.intent.action.MAIN"/>

				<category android:name="android.intent.category.LAUNCHER"/>
			</intent-filter>
		</activity>
		<receiver android:name=".MyBroadcastReceiver">
			<intent-filter>
				<action android:name="android.intent.action.MyBroadcastReceiver" />
			</intent-filter>
		</receiver>
	</application>

</manifest>
```

在清单文件中，我们可以配置这个BroadcastReceiver，注意`<intent-filter>`，其中的内容可以包括action，data，category。发送广播时我们至少为Intent指定一个action，为BroadcastReceiver配置的action必须和这个action完全匹配，广播才能收到。

## 发送有序广播

发送有序广播和无序广播差不多，也需要一个Intent。

发送有序广播
```java
sendOrderedBroadcast(intent, null);
```

注：第二个参数是权限字符串，如果不需要可以传null。

```xml
<receiver android:name=".MyBroadcastReceiver">
	<intent-filter  android:priority="1000">
		<action android:name="android.intent.action.MyBroadcastReceiver" />
	</intent-filter>
</receiver>
```

接收有序广播的`<intent-filter>`可以用`android:priority`配置优先级，优先级的最大值为`1000`。优先级大的BroadcastReceiver会先收到有序广播，如果两个BroadcastReceiver优先级相同，那么他们收到广播的顺序是随机的。

阻止有序广播继续传播：

```java
abortBroadcast();
```

调用该方法后，更低优先级的BroadcastReceiver就不会再收到广播了。

## 在Activity或Service中动态注册广播

Activity或Service继承于Context，具有如下方法：
```java
public Intent registerReceiver(BroadcastReceiver receiver, IntentFilter filter)
```

* receiver：广播接收者实例
* filter：IntentFilter对象

调用该方法能够为该Activity或Service绑定一个BroadcastReceiver，因此我们可以利用动态注册的BroadcastReceiver接收到的数据执行一些更新UI之类的操作。

动态注册广播接收者接收广播例子

MainActivity.java
```java
package com.ciyaz.broadcastdemo;

import android.app.Activity;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.util.Log;
import android.view.View;

public class MainActivity extends Activity
{

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		BroadcastReceiver broadcastReceiver = new BroadcastReceiver()
		{
			@Override
			public void onReceive(Context context, Intent intent)
			{
				Log.v("msg", intent.getStringExtra("msg"));
			}
		};
		registerReceiver(broadcastReceiver, new IntentFilter("android.intent.action.MyBroadcastReceiver"));
	}

	public void send(View view)
	{
		Intent intent = new Intent("android.intent.action.MyBroadcastReceiver");
		intent.putExtra("msg", "test broadcast");
		sendBroadcast(intent);
	}
}
```

上面例子比较简单，其实就是在Activity中，广播自收自发。
