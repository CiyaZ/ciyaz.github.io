#  Maven创建项目卡住

### 卡在刷新可用项目列表

发现卡在一个下载archetype-catelog.xml的地方，手动下载，放到`～/.m2/archetype-catelog.xml`。

### 卡在创建项目

```
$mvn archetype:generate -DarchetypeCatalog=local
```

* `-DarchetypeCatalog`：该参数意思是使用本地archetype-catelog.xml，使用IDEA创建Maven项目也要加上，否则会卡在创建项目

注：使用gradle时未发现此问题，且gradle比maven更好用，新项目建议使用gradle。
