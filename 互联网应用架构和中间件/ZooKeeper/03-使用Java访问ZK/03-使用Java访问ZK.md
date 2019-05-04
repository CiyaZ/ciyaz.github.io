# 使用Java访问ZK

这篇笔记记录如何通过Java代码操作ZK集群上的数据。通过Java代码的操作方式和客户端命令行参数其实基本是对应的，非常简单，这里就简单举几个例子。

## 引入Maven依赖

其实，ZK客户端包就一个`org.apache.zookeeper`，但是它的日志系统比较烦人，ZK的客户端包依赖了`log4j`，但我们现在基本用的都是slf4j+logback的方式，我们需要把log4j和slf4j从ZK客户端包中排除，再引入我们的日志系统。

```xml
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.4.10</version>
  <exclusions>
    <exclusion>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
    </exclusion>
    <exclusion>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </exclusion>
    <exclusion>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>log4j-over-slf4j</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.2.3</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-access</artifactId>
  <version>1.2.3</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
</dependency>
```

## 代码演示

### 创建节点

下面代码创建一个ZK节点，并指定数据：
```java
@Test
public void createNode() throws Exception {
  ZooKeeper zkClient = new ZooKeeper(connStr, sessionTimeout, new Watcher() {
    @Override
    public void process(WatchedEvent event) {
    }
  });
  String path = zkClient.create("/mynode", "hello,world".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
  System.out.println(path);
}
```

注意`create`方法参数，分别是节点路径，数据（`byte[]`类型），访问权限（`Ids.OPEN_ACL_UNSAFE`，即任何人都可以访问），节点类型（`CreateMode.PERSISTENT`即非临时节点）

### 获取节点值

```java
@Test
public void getNodeValue() throws Exception {
  ZooKeeper zkClient = new ZooKeeper(connStr, sessionTimeout, new Watcher() {
    @Override
    public void process(WatchedEvent event) {
    }
  });
  byte[] data = zkClient.getData("/mynode", false, null);
  String dataStr = new String(data);
  System.out.println(dataStr);
}
```

上面代码，我们使用`getData()`获取节点值，第一个参数是节点的路径，第二个参数表示是否监听，第三个参数是节点状态，如果用不到，传`null`即可。

### 监听节点变化

```java
@Test
public void listenNodeChange() throws Exception {
  ZooKeeper zkClient = new ZooKeeper(connStr, sessionTimeout, new Watcher() {
    @Override
    public void process(WatchedEvent event) {
      System.out.println("监听触发");
    }
  });
  List<String> children = zkClient.getChildren("/", true);
  System.out.println(children);
  Thread.sleep(Integer.MAX_VALUE);
}
```

上面代码中，我们通过`getChildren`的第二个参数指定使用监听，这和在`zkCli.sh`中的命令指定`watch`参数是一样的，监听的回调函数是`new Watcher() { ... }`中指定的逻辑，其中的逻辑会在新线程中运行，因此我们测试时要注意主线程先别结束。
