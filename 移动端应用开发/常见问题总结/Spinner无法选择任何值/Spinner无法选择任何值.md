# Spinner无法选择任何值

这个可以算是Android框架的一个bug了。

问题描述：使用ArrayAdaptor动态为Spinner添加值，程序运行后，点击Spinner可以显示下拉框，但是选择一个值后，Spinner没有任何变化（一直是空白），而且Spinner对象不会触发`onItemSelected`事件。

出问题的截图：

![](res/1.png)

中间的那个三角形，其实是Spinner的按钮，图中Spinner没有任何值，一片空白。即使Adaptor中数据加载完成，下拉框中能正确显示所有加载的值，也再无法正确为Spinner选择值了。

## 解决方案

我们必须为Spinner指定一个默认值，也就是说我们使用ArrayAdaptor为Spinner添加值时，在向Spinner指定Adaptor之前，Adaptor关联的数组中必须有一个元素作为默认值，否则就会触发这个bug。

正确的代码：

```java
Spinner spinner = (Spinner) findViewById(R.id.spinner);
stringList = new ArrayList<>();
stringList.add("default");
ArrayAdapter<String> stringArrayAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_spinner_item, stringList);
stringArrayAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
spinner.setAdapter(stringArrayAdapter);
```

正确的运行效果：

![](res/2.png)

如果Spinner的选项全部是从网络异步加载而来的，那么如何给Adaptor指定默认值呢？此时。我们的做法是等数据加载完，再调用`setAdapter()`方法。

## 总结

总结来说，就是Spinner必须有默认值，否则不能正常工作。
