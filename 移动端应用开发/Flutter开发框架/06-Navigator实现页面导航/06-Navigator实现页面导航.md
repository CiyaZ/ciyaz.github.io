# Navigator实现页面导航

在Android中，我们进行页面跳转是通过Intend实现的，页面跳转传递的参数也是包含在Intend对象中的。但是像React这样的前端框架则不同，Web端有URL和URL参数这种神奇的东西，因此前端组件化框架更加强调“路由”的概念，Flutter也是模仿这种形式设计的。

## push和pop

Flutter中有一个Navigator组件，类似Web端的history，它像栈一样工作，用于维护页面的“地址”，跳转到另一个页面时地址入栈，返回时出栈即可。我们可以用Navigator实现最简单的页面跳转。

下面例子中有两个页面，第一个页面上有一个按钮可以跳转到第二个页面，第二个页面上的按钮可以跳转回第一个页面。

```javascript
import 'package:flutter/material.dart';

void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: "路由demo1",
      home: new FirstPage(),
    );
  }
}

class FirstPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("页面1"),
      ),
      body: new Center(
        child: new RaisedButton(
          onPressed: () {
            // 这里调用了Navigator.push()
            Navigator.push(
              context,
              new MaterialPageRoute(builder: (context) => new SecondPage()),
            );
          },
          child: new Text("跳转到页面2"),
        ),
      ),
    );
  }
}

class SecondPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("页面2"),
      ),
      body: new Center(
        child: new RaisedButton(
          onPressed: () {
            // 这里调用了Navigator.pop()
            Navigator.pop(context);
          },
          child: new Text("跳转到页面1"),
        ),
      ),
    );
  }
}
```

虽然代码很长，但是实际上我们只需要关注对`Navigator`的操作就行了，第一个页面跳转第二个页面是通过调用`Navigator.push()`实现的，第二个页面跳回第一个页面是用过`Navigator.pop()`实现的，调用`pop()`和`push()`不同，它代表的是返回，而不是再一次跳转。

## Navigator传递参数

上面我们实现了页面跳转和返回，实际上通过Navigator控制页面跳转是可以传递参数的。

### 携带参数跳转

Flutter中，一切都是Widget，因此`push()`可以通过实例化另一个页面Widget时，以构造函数参数的形式进行传值。

第一个页面按钮的回调函数：
```javascript
Navigator.push(
  context,
  new MaterialPageRoute(builder: (context) => new SecondPage("123")),
);
```

第二个页面构造函数有一个参数：
```javascript
class SecondPage extends StatelessWidget {
  SecondPage(String text) {
    this.text = text;
  }

  String text;

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("页面2"),
      ),
      body: new Center(
        child: new Text(text),
      ),
    );
  }
}
```

### 页面返回时指定结果值

使用`pop()`返回时，可以携带一个结果值参数。

读取返回结果：
```javascript
onPressed: () async {
  // 这里调用了Navigator.push()
  String result = await Navigator.push(
    context,
    new MaterialPageRoute(builder: (context) => new SecondPage()),
  );
  print(result);
},
```

设置返回结果参数：
```javascript
onPressed: () {
  // 这里调用了Navigator.pop()
  Navigator.pop(context, "this is result");
},
```
