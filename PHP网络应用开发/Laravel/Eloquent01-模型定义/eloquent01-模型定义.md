# EloquentORM 模型定义

Laravel集成了EloquentORM框架，虽然使用它并不是必须的，但实际上几乎在每个CRUD项目中，ORM都会用到。这篇笔记我们来学习一下这个ORM框架。

## 一些准备工作

使用ORM前，需要在`config/database.php`中正确的配置数据库，这在之前的章节已经介绍过，参考`PHP网络应用开发/Laravel/10-数据库`。

另外，我们还需要先设计并创建好数据表。Laravel的模型并不需要手动编写字段，它能够从数据表中自动读取字段，但前提是数据表已经创建好。

## 创建模型

使用如下命令能够创建一个数据模型：

```
php artisan make:model Student
```

创建的数据模型文件，默认在项目根目录`app`文件夹下，模型名中也可以指定更深层次的命名空间。

默认生成的模型代码是这样的：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    //
}
```

我们可以发现，数据模型都是继承`Model`类的。

## 模型配置

### 数据表名

```php
protected $table = 't_student';
```

`$table`可以指定数据表名，其默认值为类名的蛇形命名，比如`ProductType`，对应的表名为`product_type`。

### 主键

```php
protected $primarykey = 'student_id';
```

`$primarykey`可以指定表的主键，默认名为`id`。

```php
public $incrementing = true;
```

`$incrementing`可以指定整数主键自增，默认值为`true`。如果你的主键是非自增的，或非数字主键，需要指定其为`false`。

```php
protected $keyType = 'string';
```

`$keyType`指定主键的数据类型，如果你的主键非整形，则需要指定该属性。

注：EloquentORM不支持联合主键。

### 时间戳

```php
public $timestamps = false;
```

`$timestamps`用来配置数据表记录的时间戳功能。该功能需要在表中存在`create_at`和`update_at`两个字段，用来存储数据记录创建和最后更新的时间戳，并由ORM框架自动维护。如果不使用该功能，需要指定`$timestamps`为`false`。

```php
protected $dateFormat = 'Y-m-d H:i:s';
```

`$dateFormat`用于配置时间戳的格式，默认为Unix时间戳。

```php
const CREATED_AT = 'create_time';
const UPDATED_AT = 'last_modified_time';
```

如果需要改变两个时间戳字段的默认名字，可以指定`CREATED_AT`和`UPDATED_AT`这两个常量。

### 字段默认值

```php
protected $attributes = [
    'cust_info_create_time' => '1999-12-31 00:00:00',
];
```

`$attributes`可以指定数据模型关联表字段的默认值。关联数组的键是字段名，值是要插入的字段值。

## 实例化模型

下面例子代码中，实例化了一个数据模型`Student`并保存。

数据表定义：
```sql
CREATE TABLE `t_student`  (
  `student_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '名字',
  `age` int(11) NOT NULL COMMENT '年龄',
  `grade` int(11) NOT NULL COMMENT '年级',
  PRIMARY KEY (`student_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

数据模型Student.php
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    protected $table = 't_student';
    protected $primarykey = 'student_id';
    public $incrementing = true;
    public $timestamps = false;
}
```

ORM操作语句：
```php
$student = new Student();
$student->name = 'Lucy';
$student->age = 18;
$student->grade = 4;
$student->save();
```

实际上，我们的数据模型定义中，没有`name`、`age`等字段，这些都是EloquentORM根据数据表自动创建的。
