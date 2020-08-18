# StreamAPI

我们知道C#有Linq，它处理集合是相当好用的。那么Java中有什么替代语法呢？

StreamAPI是Java8新引入的一个特性，使用StreamAPI能够更加方便的处理集合。虽然Linq和StreamAPI写法很不一样（C#是用类SQL的指令，Java是函数式的代码），但本质和编写思路其实是一种，我个人其实更倾向于后者。

## StreamAPI例子

下面我们实现一个例子，从一个装有若干`Student`学生对象的`List`中，筛选出`student.age`值为`16`的学生，生成一个新的`List`。

如果不使用StreamAPI，我们的代码将是如下这样的：

```java
List<Student> students = Arrays.asList(
        new Student("Tom", 1, 16),
        new Student("Jerry", 1, 16),
        new Student("Lucy", 1, 17),
        new Student("Bob", 2, 17),
        new Student("Kate", 2, 18)
);

List<Student> result = new ArrayList<>();
for (Student s : students) {
    if (s.getAge() == 16) {
        result.add(s);
    }
}
```

使用StreamAPI后，我们代码逻辑的可读性大大提高了：

```java
List<Student> students = Arrays.asList(
        new Student("Tom", 1, 16),
        new Student("Jerry", 1, 16),
        new Student("Lucy", 1, 17),
        new Student("Bob", 2, 17),
        new Student("Kate", 2, 18)
);

List<Student> result = students.stream()
        .filter((s) -> s.getAge() == 16)
        .collect(Collectors.toList());
```

前面代码中，其实可以看出，我们使用StreamAPI分三步：

1. 调用`stream()`得到流对象
2. 调用`filter()`等中间操作，进行集合对象的过滤、去重、排序等变换
3. 调用`collect()`终止操作并生成结果

## StreamAPI使用场景

实际开发工作中，Java代码的数据库增删改查、调用接口和集合处理这些业务逻辑几乎占九成以上，然而业务逻辑其实是十分复杂的！使用上面第一种写法，数据结构嵌套N多层，基本写着写着人就傻了，如果使用StreamAPI能够较大的提高我们工程代码的可维护性。

不过需要注意的是StreamAPI需要Java8，可能许多线上系统还没过渡到这个“新”版本的JDK。

## Stream常用操作

### 中间操作

| 方法                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| filter(Predicate p)     | 过滤操作                                                     |
| distinct()              | 去重操作                                                     |
| limit(long maxSize)     | 按数量限制                                                   |
| map(Function f)         | 对每个元素执行映射函数                                       |
| flatMap(Function f)     | 对每个元素执行映射函数（和map的区别是flatMap用于将一个元素映射为多个元素） |
| sorted()                | 排序操作                                                     |
| sorted(Comparator comp) | 自定义排序操作                                               |

### 终止操作

| 方法                 | 说明                            |
| -------------------- | ------------------------------- |
| forEach(Consumer c)  | 迭代                            |
| collect(Collector c) | 将流封装成List、Set等形式的结果 |
| max(Comparator c)    | 取出流中最大值                  |
| min(Comparator c)    | 取出流中最小值                  |
| count()              | 返回流中元素个数                |
