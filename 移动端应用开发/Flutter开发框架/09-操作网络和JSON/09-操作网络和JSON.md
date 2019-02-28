# 操作网络和JSON

我们的客户端程序和服务器程序通信，目前来看一般都选择HTTP REST API+JSON的形式。本片笔记记录如何在Flutter中使用网络和JSON。

## 需要的库

发起HTTP请求需要`http`这个库，一些老的教程中可能会用到`dart:io`里的库，但是我研究了半天也没搞明白怎么用，反倒是Flutter官网上的例子就是使用的`http`这个库，我们把它加入`pubspec.yaml`即可。

解析JSON需要`dart:convert`这个库，我们直接`import`就行了。

## 服务端建立

这里，我们设定请求URL为`http://192.168.1.107:8080/users/1`，返回内容为JSON格式：

```json
{"name":"Tom", "age":1}
```

这里我就不再起一个工程编写服务端了，而是使用这个小工具进行模拟：

[https://github.com/CiyaZ/servmock/releases](https://github.com/CiyaZ/servmock/releases)

## 请求数据例子

```javascript
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: "HTTP请求示例",
      home: new Scaffold(
        appBar: new AppBar(
          title: new Text("HTTP请求示例"),
        ),
        body: new HttpRequestDemoPage(),
      ),
    );
  }
}

class User {
  User(this.name, this.age);

  String name;
  int age;

  @override
  String toString() {
    return "I am " + name + ", " + age.toString() + " years old.";
  }
}

class HttpRequestDemoPage extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return new HttpRequestDemoPageState();
  }
}

class HttpRequestDemoPageState extends State<HttpRequestDemoPage> {
  get() async {
    // 发起网络请求
    http.Response response =
        await http.get("http://192.168.1.107:8080/users/1");
    if (response.statusCode == 200) {
      // JSON解析
      Map<String, dynamic> jsonMap = json.decode(response.body);
      User user = new User(jsonMap["name"], jsonMap["age"]);
      print(user.toString());
    }
  }

  @override
  Widget build(BuildContext context) {
    return new RaisedButton(
      onPressed: get,
      child: new Text("点击加载数据"),
    );
  }
}
```

上面代码比较容易理解，注意HTTP相关操作用到了`async/await`写法，解析JSON的写法和JavaScript非常像，这里代码就不多做解释了。
