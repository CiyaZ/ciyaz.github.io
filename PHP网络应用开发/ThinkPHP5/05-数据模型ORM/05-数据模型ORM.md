# 数据模型ORM

ThinkPHP是一个MVC框架，这篇笔记我们主要看一下M层，即数据模型。在Web程序的敏捷开发中，一个ORM框架时必不可少的，ThinkPHP有一个简易的ORM功能，但是也够用了。

## 连接数据库

在学习如何使用数据模型前，我们首先需要正确的配置数据库，这里我们以MySQL为例。

config/database.php
```php
// 数据库类型
'type'            => 'mysql',
// 服务器地址
'hostname'        => '127.0.0.1',
// 数据库名
'database'        => 'netstore',
// 用户名
'username'        => 'root',
// 密码
'password'        => 'root',
// 端口
'hostport'        => '3306',
// 连接dsn
'dsn'             => '',
// 数据库连接参数
'params'          => [],
// 数据库编码默认采用utf8
'charset'         => 'utf8mb4',
// 数据库表前缀
'prefix'          => 't_',
```

上面注释都非常清晰，这里就不多叙述了。

## 创建数据模型

我们可以使用`think`命令创建一个数据模型。下面例子创建一个客户模型：

```
php think make:model index/Cust
```

注意：模型的名字不是随便起的，我们的数据表叫`t_cust`数据库配置中已经配置了前缀`t_`，因此我们创建的数据模型的名字就是`Cust`。所有其他数据模型的命名都以此类推。

application/index/model/Cust.php
```php
<?php

namespace app\index\model;

use think\Model;

class Cust extends Model
{
}
```

ThinkPHP中数据模型不会自动识别主键，我们需要手动指定一下：
```php
class Cust extends Model
{
    protected $pk = 'cust_id';
}
```

`cust_id`是`t_cust`表的主键。

## 简单查询

我们这里直接在`index/Index`控制器上进行实验，下面例子根据主键选择一个数据项，并以JSON形式返回：

```php
public function index()
{
  $cust = Cust::get(2);
  return json($cust);
}
```

ThinkPHP提供了一系列的链式查询方法，比如查询数据库中前三个姓「赵」的客户：

```php
$custs = Cust::where('cust_name', 'like', '%赵%')->limit(0, 3)->select();
```

这种查询构造的写法很像Java的Criteria查询，我们能够通过控制代码的逻辑实现动态查询。

## 关联查询

在真正的业务数据库中，我们的数据实体之间，通常有复杂的关联关系。这里我们再创建一个`Order`模型表示订单，然后查询一个客户对应的所有订单。

application/index/model/Order.php
```php
class Order extends Model
{
	protected $pk = 'order_id';
}
```

application/index/model/Cust.php
```php
class Cust extends Model
{
	protected $pk = 'cust_id';

	public function orders()
	{
		return $this->hasMany('Order', 'cust_id', 'cust_id');
	}
}
```

注意客户模型中我们添加了一个`orders`方法，该方法用于查询该客户对应的所有订单信息。`hasMany()`表示一对多关联关系，其参数为关联模型名，外键，当前主键。

需要注意下在控制器中调用时的写法，写法如下（注意`orders`后面没有小括号）：

```php
$cust = Cust::get(1);
$orders = $cust->orders;
```

## 直接使用SQL语句

ORM有时并不能满足我们的需求（或者说实现起来太费事），在那些使用Oracle、Hive的数据仓库型项目中尤其如此，这种时候我们更倾向于直接写一句SQL解决问题。

下面例子SQL语句统计2017年1月1日，沈阳市的营业额：
```sql
SELECT
	sum( t_order.payment ) income
FROM
	t_order
	INNER JOIN t_cust ON t_order.cust_id = t_cust.cust_id
	INNER JOIN t_area area_district ON t_cust.area_id = area_district.area_id
	INNER JOIN t_area area_city ON area_district.parent_id = area_city.area_id
WHERE
	area_city.area_name = '沈阳市'
	AND t_order.create_time BETWEEN '2017-01-01 00:00:00'
	AND '2017-01-01 23:59:59';
```

下面例子演示如何在ThinkPHP中执行这条SQL语句：

```php
public function index()
{

  $sql = <<<sql
SELECT
sum( t_order.payment ) income
FROM
t_order
INNER JOIN t_cust ON t_order.cust_id = t_cust.cust_id
INNER JOIN t_area area_district ON t_cust.area_id = area_district.area_id
INNER JOIN t_area area_city ON area_district.parent_id = area_city.area_id
WHERE
area_city.area_name = ?
AND t_order.create_time BETWEEN ?
AND ?
sql;
  $result = Db::query($sql, ['沈阳市', '2017-01-01 00:00:00', '2017-01-01 23:59:59']);
  return json($result);
}
```

SQL中我们使用了`?`作为占位符，在`Db::query()`中使用数组参数将其传入。

注：上面对日期的处理只是个演示，其实使用字符串格式的日期作为用户传入的查询参数不是很好，建议使用时间戳。

如上的写法，我们在Navicat或PL/SQL中调试好SQL语句，直接粘进PHP代码中就能解决问题。如果在PHP代码中一点点组织Criteria查询一点点的调试，再弄个三模型关联，地区模型还得自关联，如果还有分组聚合的需求，那就太困难了。Java中许多奇葩项目MyBatis和JPA（或Hibernate）同时存在，其实就是这个原因。ORM适合操作业务库实现数据实体增删改查，原生SQL适合操作数据仓库进行统计分析和报表导出。

## 增删改操作

增删改非常的简单，这里就不多做介绍了。具体请参考文档。

## 业务逻辑放在哪

经过上面学习，我们来自Java体系的同学可能产生一个问题：把SQL或ORM操作写在控制器里，这显然不利于显示数据处理逻辑和业务逻辑解耦，Java分为表现层、业务逻辑层、数据持久层，ThinkPHP如何划分呢？

我的理解是这样的：ThinkPHP没有考虑太过复杂的场景（JavaWeb常说的三层架构实际上也有些过度设计了），为了解耦显示数据处理逻辑和业务逻辑，我们可以把业务逻辑放在MVC的M层。将业务代码封装成数据模型的方法，控制器仅负责调用数据和组织数据用于输出显示即可。
