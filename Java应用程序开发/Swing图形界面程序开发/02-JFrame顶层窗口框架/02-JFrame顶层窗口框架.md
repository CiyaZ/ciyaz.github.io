# JFrame

Java中顶层窗口（没有包含在其他窗口中的窗口）叫做`Frame`。Swing中，有一个`JFrame`类作为顶层窗口框架。

注意和awt的`Frame`区分开，awt也有顶层窗口，写程序时不要搞错包。Swing中的控件都是以`J`开头的。

## 显示JFrame

Swing/awt提供的API实在太多了，我们不可能一一列举，把常用的例子记下，使用时翻一翻就足够了。如果不够用，再看文档，查google。

下面是一个最简单的程序，显示一个什么都没有的窗口。

MyJFrame.java
```java
import javax.swing.*;

public class MyFrame extends JFrame
{
	public MyFrame()
	{
		this.setSize(300, 200);
	}
}
```

Main.java
```java
import javax.swing.*;
import java.awt.*;

public class Main
{
	public static void main(String[] args)
	{
		EventQueue.invokeLater(() ->
		{
			MyFrame myFrame = new MyFrame();
			myFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
			myFrame.setVisible(true);
		});
	}
}
```

### 为什么继承JFrame

我初学Java时经常想，为什么不直接在一个Main类里实例化JFrame，然后`frame.setSize()`之类的调用若干个配置JFrame的API，最后`setVisible()`呢？

因为这不符合面向对象的封装性。我们继承一个控件，可以把这个控件做一系列扩展，然后进行复用。否则，只能一次次复制粘贴一堆恶心的代码，这不优雅。

### EventQueue.invokeLater()

《Java核心技术》这本书中，要求我们使用这种方式把UI线程和主线程隔离，这是十分合理的。尽管不这么写也基本不会出错。

### 运行结果

![](res/1.png)

### JFrame常用的API

这里介绍一些JFrame常用的API便于使用时快速查阅。

#### 有关显示

* `void setSize(int width, int height)` 设置窗口大小。如果不指定大小，默认大小`0*0`。
* `void pack()` 窗口放入组件后，使用`pack()`而不是`size()`可以根据组件的大小自动设置窗口的大小。
* `void setDefaultCloseOperation(int operation)` 设置窗口关闭时的动作，默认点击窗口的`x`不会退出程序，只是隐藏了这个窗口。参数的其他选项请参考文档`WindowConstants`。
* `void setVisible(boolean b)` 设置为`true`窗口就显示。
* `void setUndecorated(boolean undecorated)` 是否显示窗口边框。

#### 有关窗口位置和大小

* `void setLocation(int x, int y)` 设置窗口显示位置，左上角坐标为`(0, 0)`。
* `void setBounds(int x, int y, int width, int height)` 设置坐标和大小。
* `void setResizable(boolean resizable)` 设置窗口是否能由用户调整大小。
* `void setExtendedState(int state)` 传入`Frame.MAXIMIZED_BOTH`可最大化显示窗口，这个方法非常有用。

#### 标题和图标

* `void setIconImage(@Nullable java.awt.Image image)` 设置窗口图标图片。
* `void setTitle(String title)` 设置窗口标题

#### 获取属性

上面列举了一大堆`get`方法，实际上很多属性有对应的`set`方法。我们使用Java编程时，应该有这个意识，不用记住API，随时使用IDE的快捷键查看方法提示和JavaDoc文档。

#### 向JFrame添加组件

* `java.awt.Component add(java.awt.Component comp)` 例如添加一个JButton。
