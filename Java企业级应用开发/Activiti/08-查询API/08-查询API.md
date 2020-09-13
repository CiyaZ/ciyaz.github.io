# 查询API

我们的流程执行中，有很多需要查询的操作，比如查询某用户当前有哪些任务待处理等等。Activiti中各个接口提供了强大的查询API，使用非常方便，这里我们简单介绍一下。

## Query构造查询

对于各个接口，Activiti封装了`Query`对象，我们可以通过它来构造查询条件。

### 获取查询构造器

Activiti中，各种接口都提供了形如`createQuery()`的方法，用于获取该组接口对应实体类的查询构造器。

例子：
```java
UserQuery userQuery = identityService.createUserQuery();
```

### 组装查询条件

在查询构造器上，我们可以组装各种查询条件，通常都是对应实体类的字段。

例子：
```java
userQuery = userQuery.userId("tom");
```

上面代码中，`userId()`输入了一个查询条件，方法返回值还是`UserQuery`类型，因此我们可以使用链式调用的方式来编写。

### 获取结果

查询结果分为两种：单个结果和结果列表，其中后者还需要考虑分页的情况。Activiti把这些情况都封装好了。

单个结果：
```java
User user = userQuery.singleResult();
```

结果列表：
```
List<User> userList = userQuery.list();
```

结果列表分页：
```
List<User> userList = userQuery.listPage(firstResult, maxResults);
```
