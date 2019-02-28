# MyBatis动态SQL

MyBatis能够在Mapper配置XML文件中，使用`<if>`、`<choose>`等标签动态拼接SQL语句。实际上，动态拼接SQL不代表我们把软件的业务逻辑搬到了配置文件中，这个功能只是为了让SQL能更好的完成每一个持久层的数据操作任务，因此动态SQL一般有固定的几个应用场景，再加上XML配置文件中拼接SQL可读性极差，而且完全无法debug，因此这个功能比较鸡肋。

下面简单介绍Mybatis的动态SQL标签。

例子使用的数据库表
```sql
create table t_user(
	user_id bigint auto_increment,
	username varchar(255) not null,
	email varchar(255) not null,
	password varchar(255) not null,
	primary key(user_id)
);
```

例子使用的实体类
```java
public class User
{
	private Long userId;
	private String username;
	private String email;
	private String password;
...省略get/set/toString
```

## if

这里实现的功能是根据多个字段的模糊匹配查询
```java
public List<User> searchUserByUsernameAndEmail(@Param("username") String username, @Param("email") String email);
```

```xml
<select id="searchUserByUsernameAndEmail" resultType="com.ciyaz.mbdemo.domain.User">
  select user_id as userId,username,email,password from t_user
  where 1=1
  <if test="username != null and username != ''">
    and username like concat('%',#{username},'%')
  </if>
  <if test="email != null and email != ''">
    and email like concat('%',#{email},'%')
  </if>
</select>
```

`<if>`标签必须有`test`属性，其中的内容实际上是OGNL表达式（在struts2章节介绍过），根据表达式结果是`true`或`false`判断该分支是否拼接到SQL上。

注意：`where 1=1`，因为每个匹配条件都是`and`开头的，这是为了拼接后的SQL不至于语法出错，这不是最好的写法，推荐使用`where`标签（后文会介绍）。

## choose

`if`只能实现一个分支判断，`choose`可以实现多个。

```xml
<choose>
  <when test""></when>
  <otherwise></otherwise>
</choose>
```

注意：

* 和`if`的`test`属性一样，`when`的`test`也是OGNL表达式
* `when`标签可以写多个

## where

这里再次使用`if`的例子，将其使用`where`进行改进。

```xml
<select id="searchUserByUsernameAndEmail" resultType="com.ciyaz.mbdemo.domain.User">
  select user_id as userId,username,email,password from t_user
  <where>
    <if test="username != null and username != ''">
      and username like concat('%',#{username},'%')
    </if>
    <if test="email != null and email != ''">
      and email like concat('%',#{email},'%')
    </if>
  </where>
</select>
```

`<where>`的作用：`where`内部包裹的元素有返回值时，在SQL中加上一个`WHERE`，否则不加；如果内部包裹的元素以`AND`或`OR`开头，将其去掉。总之，`where`就是配合动态拼接SQL条件子句使用的。

## set

这个标签类比`where`标签理解即可：如果`set`包裹的元素有返回值时，就插入一个`set`；如果内部包裹元素最后以逗号`,`结尾，将其去掉。

这个标签是配合`UPDATE`语句使用的。

## trim

实际上面介绍的`where`和`set`标签都可以用`trim`标签实现，`trim`是一个更加通用的用于辅助动态拼接SQL的标签：

`trim`实现`set`标签。

```xml
<trim prefix="SET" suffixOverrides=","></trim>
```

* `prefix`：`trim`内部包含内容时，会加上前缀
* `prefixOverrides`：`trim`内部包含内容时，会去掉该前缀
* `suffix`：`trim`内部包含内容时，会加上后缀
* `suffixOverrides`：`trim`内部包含内容时，会去掉该后缀

## foreach

该标签用于对实现了`Iterable`接口的Java对象进行遍历，分为`List`和`Map`两种情况，下面举两个例子。

生成一个`IN`子句的范围：

```xml
<foreach collection="idList" open="(" close= ")" separator= "," item="id" index="i">
  #{id}
</foreach>
```

* `collection`：要迭代的属性名
* `open`：整个的前缀
* `close`：整个的后缀
* `separator`：分隔符
* `item`：迭代的变量名
* `index`：`List`是当前的索引值，`Map`是当前的键值

## bind

`bind`标签用于创建一个值并绑定到一个变量上，基本没设么没用。

```
<bind name="" value="" />
```

* `name`:创建的变量名
* `value`:创建的变量值
