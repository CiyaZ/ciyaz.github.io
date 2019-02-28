# RecyclerView 加强版列表

无论PC还是Android，显示一个数据集几乎是每个应用必做的工作，Android本来提供了ListView和GridView，使用起来还算方便。但是因为这个功能太常用了，大家都玩出了很多花样，因此RecyclerView出现了，我们可以通过recyclerview-v7支持包导入。这个控件作用类似ListView，但是高度可配置（当然也是极其难写）。

实现一个ListView只需要实现一个Adapter，实现RecyclerView需要这几个组件：

* LayoutManager 用于控制RecyclerView的显示布局
* ItemDecoration 控制对数据项的装饰效果
* ItemAnimation 控制数据项的增删动画

## 引入gradle依赖

```java
compile 'com.android.support:recyclerview-v7:26.+'
```

我们需要单独导入RecyclerView的支持包。

## 使用RecyclerView显示数据

下面演示使用RecyclerView实现展示数据列表，这个展示效果和ListView很像。

Activity主布局文件 activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:paddingLeft="16dp"
				android:paddingRight="16dp" >

	<android.support.v7.widget.RecyclerView
		android:id="@+id/rl_message"
		android:layout_width="match_parent"
		android:layout_height="match_parent"/>
</RelativeLayout>
```

数据项的布局文件 recyclerview_content.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			  android:layout_width="match_parent"
			  android:layout_height="wrap_content"
			  android:orientation="vertical">

	<TextView
		android:textAlignment="center"
		android:id="@+id/tv"
		android:textSize="20sp"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"/>

</LinearLayout>
```

ActivityMain.java
```java
package com.ciyaz.recyclerviewdemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.List;

public class MainActivity extends AppCompatActivity
{

	private RecyclerView recyclerView;
	private List<String> dataList = DataDao.getData();

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		recyclerView = (RecyclerView) findViewById(R.id.rl_data);
		//设置显示数据集的布局管理器
		recyclerView.setLayoutManager(new LinearLayoutManager(this));
		//设置显示数据使用的适配器
		recyclerView.setAdapter(new DataAdapter());
	}

	class DataAdapter extends RecyclerView.Adapter<DataAdapter.DataViewHolder>
	{
		/**
		 * 当RecyclerView需要构造一个新的ViewHolder时使用
		 * @param parent 作为容器，新的View需要添加到其中
		 * @param viewType 新的View的类型
		 * @return 新构造的ViewHolder
		 */
		@Override
		public DataViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
		{
			//使用自定义的布局文件填充View对象
			View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.recyclerview_content, parent, false);
			//构造新的ViewHolder并返回
			return new DataViewHolder(view);
		}

		/**
		 * 需要在指定position显示数据时回调
		 * @param holder 指定position对应的ViewHolder
		 * @param position 当前position
		 */
		@Override
		public void onBindViewHolder(DataViewHolder holder, int position)
		{
			holder.tv.setText(dataList.get(position));
		}

		/**
		 * @return 数据项个数
		 */
		@Override
		public int getItemCount()
		{
			return dataList.size();
		}

		/**
		 * 自定义ViewHolder类
		 */
		class DataViewHolder extends RecyclerView.ViewHolder
		{

			TextView tv;

			DataViewHolder(View itemView)
			{
				super(itemView);
				tv = itemView.findViewById(R.id.tv);
			}
		}
	}
}
```

代码写了详细注释，这里就不多解释了。

注：编写代码时发现一个问题，`recyclerview_content.xml`中，根组件`android:layout_height`设为`match_parent`，同时适配器`onCreateViewHolder()`方法中，View布局填充时使用了`parent`对象，会导致全屏幕只显示一个数据项，但可以下拉查看其他数据项。这里建议将前者改为`android:layout_height="wrap_content"`。

## ItemDecoration

上面只是实现了最基本的数据展示，目前显示效果还非常丑。下面介绍如何通过ItemDecoration实现数据项的装饰效果。



## LayoutManager

下面介绍RecyclerView中，使用LayoutManager控制数据集的显示方式。

<这个控件太tm复杂了，待补充>
