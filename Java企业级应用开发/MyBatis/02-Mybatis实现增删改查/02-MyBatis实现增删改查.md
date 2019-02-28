# MyBatis实现简单的增删改查

这篇笔记主要记录如何使用MyBatis实现简单的增删改查，如何编写Mapper的接口类和XML配置。

这里我们使用MySQL进行演示，新建一个表`t_role`：
```sql
create table t_role(
	role_id bigint auto_increment,
	rolename varchar(255) not null,
	primary key (role_id)
);
```

上一篇笔记中，我们仅创建了实体类和Mapper的XML配置，这在引用Mapper时不方便，通常情况下，我们都会创建一个Mapper接口，接口中定义了我们要实现的方法声明，包括方法名、参数和返回值。

实体类 com.ciyaz.mbdemo.domain.Role

```java
package com.ciyaz.mbdemo.domain;

public class Role
{
	private Long roleId;
	private String rolename;
...省略get/set/toString方法
```

接口 com.ciyaz.mbdemo.mapper.RoleMapper，这里我们实现了一些用作例子的增删改查方法。

```java
package com.ciyaz.mbdemo.mapper;

import com.ciyaz.mbdemo.domain.Role;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface RoleMapper
{
	public Role queryRoleById(Long roleId);

	public List<Role> queryRoleByRolename(String rolename);

	public List<Role> queryAllRole();

	public List<Role> queryRoleByPage(@Param("start_index") Integer startIndex, @Param("page_size") Integer pageSize);

	public int insertRole(Role role);

	public int deleteRole(Long roleId);

	public int updateRole(Role role);
}
```

Mapper配置 resources/com/ciyaz/mbdemo/mapper/RoleMapper.xml，里面的配置在后文中详细介绍。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
		PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ciyaz.mbdemo.mapper.RoleMapper">
</mapper>
```

注意：`namespace`属性对应接口的类全名。

## 查询

### 查询一条结果

接口方法：
```java
public Role queryRoleById(Long roleId);
```

对应XML配置：
```xml
<select id="queryRoleById" resultType="com.ciyaz.mbdemo.domain.Role">
  select role_id as roleId,rolename from t_role where role_id=#{roleId}
</select>
```

说明以下几点：

1. `select`的`id`属性对应接口中的方法名
2. `resultType`对应实体类的类全名，这里`Role`是该查询方法的返回值
3. SQL语句中，`role_id as roleId`为查出的一个字段起了别名，这是为了和实体类的属性名对应，只有这两个名相同MyBatis才能自动映射
4. 方法声明中的参数在XML中的SQL语句里，需要用`#{}`包裹，里面的参数名需要和方法声明中的参数名对应
5. 只有一个参数时可以不用`@param`注解标注（上面就是），多个参数需要用注解标注，后面会给出例子

查询一条结果的另一个例子：
```java
public List<Role> queryRoleByRolename(String rolename);
```

```xml
<select id="queryRoleByRolename" resultType="com.ciyaz.mbdemo.domain.Role">
		select role_id as roleId,rolename from t_role where rolename=#{rolename}
</select>
```

### 使用`resultMap`手动映射查询结果

之前的例子是MyBatis自动映射到Java实体类对象的，其中因为数据库中字段名是`role_id`而实体类中对应属性名是`roleId`，我们使用了别名MyBatis才能正常识别，除此之外，我们还可以手动进行映射。

```xml
<resultMap id="roleMap" type="com.ciyaz.mbdemo.domain.Role">
	<id property="roleId" column="role_id" />
	<result property="rolename" column="rolename" />
</resultMap>
<select id="queryRoleById" resultMap="roleMap">
	select role_id,rolename from t_role where role_id=#{roleId}
</select>
```

* `id`：用于映射主键
* `result`：用于映射普通字段

除了上面介绍的用法，在关联查询中，`resultMap`还有其他的配置，将在后文介绍。

### 查询多条结果

接口方法：
```java
public List<Role> queryAllRole();
```

对应XML配置：
```xml
<select id="queryAllRole" resultType="com.ciyaz.mbdemo.domain.Role">
  select role_id as roleId,rolename from t_role
</select>
```

这里SQL语句返回的可能是一条结果，也可能是多条结果，也可能没有结果。方法的返回值是一个`List`，返回结果可能里面有多条、1条、或是0条数据。

注意：

1. 实际上返回一个结果时，也可以使用`List`接收返回结果
2. 不使用List接收，但是返回了多条结果会报错

### 传递多个参数进行查询

接口方法：
```java
public List<Role> queryRoleByPage(@Param("start_index") Integer startIndex, @Param("page_size") Integer pageSize);
```

对应XML配置：
```xml
<select id="queryRoleByPage" resultType="com.ciyaz.mbdemo.domain.Role">
  select role_id as roleId,rolename from t_role limit #{start_index},#{page_size}
</select>
```

这里传递了多个参数进行查询，注意参数需要用`@param`进行标注，这个注解是`org.apache.ibatis.annotations.Param`包的，不要和别的弄混。只有一个参数时，`@param`可以省略，SQL中的参数名对应方法的参数名。

## 插入

接口方法：
```java
public int insertRole(Role role);
```

对应XML配置：
```xml
<insert id="insertRole" useGeneratedKeys="true" keyProperty="roleId">
  insert into t_role(rolename) values (#{rolename})
</insert>
```

由于我们使用了MySQL的自增长主键，因此我们需要在插入后得知该条数据主键值，`useGeneratedKeys="true"`表示插入时使用数据库提供的主键自增长功能，`keyProperty`是实体类中主键对应的字段名。

这里SQL中的参数字段对应实体类中的属性名，实际上这也是传入多个参数的一种方式。

注意：接口方法的返回值是受影响的列数，主键值在传入的对象中。

获取得到的主键值：

```java
Role role = new Role();
role.setRolename("新角色");
mapper.insertRole(role);
System.out.println("pk=" + role.getRoleId());
```

## 删除

接口方法：
```java
public int deleteRole(Long roleId);
```

对应XML配置：
```xml
<delete id="deleteRole">
  delete from t_role where role_id=#{roleId}
</delete>
```

## 修改

接口方法：
```java
public int updateRole(Role role);
```

对应XML配置：
```xml
<update id="updateRole">
  update t_role set rolename=#{rolename} where role_id=#{roleId}
</update>
```
