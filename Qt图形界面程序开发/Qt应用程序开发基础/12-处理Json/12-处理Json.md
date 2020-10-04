# 处理Json

在互联网相关应用开发中，因为对网络带宽、实时性的要求都不高，因此文本格式Json和XML是常用于数据交换的格式。此外，Json、XML作为配置文件，也具有可读性好，通用性高等特点。XML虽然功能更强大，但更为复杂，目前还是Json使用更多，这里我们学习如何在Qt中序列化和反序列化Json格式。

## 处理Json

Qt5中，提供了`QJsonDocument`类用于处理Json。

### 生成Json

下面代码使用`QJsonDocument`生成了一段简单的Json字符串。

```cpp
QJsonObject json;
json.insert("name", "Tom");
json.insert("age", 18);
QJsonArray hobbies;
hobbies.append("吃饭");
hobbies.append("睡觉");
hobbies.append("学C++");
json.insert("hobbies", hobbies);

QJsonDocument doc;
doc.setObject(json);
QString jsonStr(doc.toJson(QJsonDocument::Compact));
qDebug() << jsonStr;
```

输出结果：
```
{"age":18,"hobbies":["吃饭","睡觉","学C++"],"name":"Tom"}
```

代码中，`QJsonObject`、`QJsonArray`等用于组织Json结构，这和很多语言的JsonAPI都是类似的，这里就不多说了。`doc.toJson()`函数返回“UTF-8”格式的`QByteArray`类型字节数组，将其作为`QString`构造函数的参数即可生成UTF8字符串。

默认情况下，`toJson()`生成的Json数据带有缩进等格式化字符，通常实际开发中是不需要这些的，因此我们指定了`QJsonDocument::Compact`选项，将生成的Json数据最简化。

### 解析Json

下面代码中，我们在变量中定义了一个Json字符串，然后使用`QJsonDocument`进行解析。

```cpp
QString jsonStr = "{\"age\":18,\"hobbies\":[\"吃饭\",\"睡觉\",\"学C++\"],\"name\":\"Tom\"}";
QJsonParseError err;
QJsonDocument doc = QJsonDocument::fromJson(jsonStr.toUtf8(), &err);
if (!doc.isNull() && (err.error == QJsonParseError::NoError))
{
    QJsonObject jsonObj = doc.object();
    qDebug() << jsonObj.value("name");
    qDebug() << jsonObj.value("age");
    QJsonArray hobbies = jsonObj.value("hobbies").toArray();
    for (int i = 0; i < hobbies.size(); i++)
    {
        qDebug() << hobbies[i];
    }
}
```

注意：由于Json字符串中包含双引号，所以代码中的Json有很多转义符号`\`，正常从网络接口、磁盘文件等方式读取是不需要的。
