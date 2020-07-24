# EloquentORM 增删改查

这篇笔记我们介绍如何用EloquentORM实现简单的增删改查。

## 查询

Laravel对数据库的操作，提供了一组类似Java的QBC查询接口，Laravel中叫做查询构造器。因为SQL语言本身就是通过结构化的查询指令来查询数据的，查询构造器其实就是用代码函数实现`where`、`count`、`orderby`之类的各种功能。

不过在直接使用SQL的情况下，基本没人用查询构造器，所以之前章节没有介绍。使用EloquentORM时，由于不需要写SQL了，那么自然使用查询构造器就是更好的方式了。

由于指令函数非常多，而且基本和SQL语言差不多，这里除了下面例子中用到的指令就不过多介绍了，具体使用时参考文档即可。

### 查询全部数据记录

下面例子代码中，我们直接取出`Student`模型中的所有表记录。

```php
$students = Student::all();
```

如果未查到结果，返回空数组。

### 根据主键查询记录

`find()`能够根据给定一个或多个主键，查询对应主键的记录。参数是单个主键值，或主键数组。

```php
$students = Student::find([1, 2, 3]);
```

如果给定了单个主键，返回的是模型对象（或`null`），如果给定多个主键，返回的是集合。

### 查询单条记录

`first()`能够从结果集合中获取单条记录。

```php
$student = Student::where('grade', 1)->first();
```

如果未查到结果，返回`null`。

### 基于查询构造器筛选记录

下面例子代码中，我们使用查询构造器，取出`Student`模型中，`grade=1`的数据，并按照`age`字段倒序排序。

```php
$students = Student::where('grade', 1)
    ->orderBy('age', 'desc')
    ->get();
```

### 处理结果集合

`all()`或`get()`返回的可能是多个值，因此它是一个可迭代的集合，具体类型为`Illuminate\Database\Eloquent\Collection`。我们可以遍历这个集合。

```php
foreach ($students as $s) {
    Debugbar::info($s->name);
}
```

## 插入

插入一个模型，只需要使用`new`将其实例化，然后调用`save()`方法保存即可。

```php
$student = new Student();
$student->name = 'Lucy';
$student->age = 18;
$student->grade = 4;
$student->save();
```

### 获取插入后的主键

一个常见的需求是插入后，需要得到插入数据的主键。这可以通过`save()`后在数据模型上获取主键字段实现。如果开启了时间戳功能，`create_at`字段会自动进行维护。

```php
$student = new Student();
$student->name = 'Lucy';
$student->age = 18;
$student->grade = 4;
$student->save();

$pk = $student->student_id;
```

## 更新

`save()`方法也可以同样用于模型更新，对于ORM查询出来的模型，修改其字段后，调用`save()`即可更新数据记录。如果开启了时间戳功能，`update_at`字段会自动进行维护。

```php
$student = Student::find(3);
$student->grade = $student->grade + 1;
$student->save();
```

## 删除

在查询出来的模型上调用`delete()`方法，能够删除模型。如果查询结果是多个，多个数据都会被删除。

```php
Student::where('name', 'Lucy')->delete();
```

### 根据主键删除

`destroy()`方法能够根据给定的一个或多个主键删除数据记录。

```php
Student::destroy([2, 5]);
```

## 模型比较

模型的`is()`方法能够比较两个模型，观察它们是否是同一条数据记录（主键和字段值全部相同）。

```php
$s1 = Student::find(1);
$s2 = Student::where('name', 'tom')->first();
dump($s1->is($s2));
```
