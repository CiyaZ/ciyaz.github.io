# Application 全局对象

每一个Android应用有一个单例的Application对象，在Activity或Service中，我们能通过如下方法拿到这个Application对象的引用。

```java
getApplication()
```

Application全局唯一，那么它的一个明显的用途就是可以用来存储全局变量。当然全局变量我们完全也可以有自己的实现方式，但是既然Android提供了Application对象给我们，我们正确的使用它，代码就比较规范。

为了能让Application对象存储我们需要的全局变量引用，我们只需要继承它，并添加一些我们需要的数据引用。

```java
package com.ciyaz.myapplication;

import android.app.Application;

public class MyApplication extends Application
{
	private String str = "aaa";

	public String getStr()
	{
		return str;
	}

	public void setStr(String str)
	{
		this.str = str;
	}
}
```

这里例子中，简单起见，我们只添加了一个字符串全局变量，实际使用中，可能需要很复杂的全局变量。

manifest.xml
```xml
<application
  android:name=".MyApplication"
  android:allowBackup="true"
  android:icon="@mipmap/ic_launcher"
  android:label="@string/app_name"
  android:supportsRtl="true"
  android:theme="@style/AppTheme">
</application>
```

为了让系统使用我们自定义的Application对象而不是原来系统提供的Application，我们需要在清单文件中加上一个`name`属性，值为我们自定义Application的全名。这样，我们自定义的Application就能正常被系统实例化了。
