# Spring Data 之 JPA

JPA（Java Persistence API）是JavaEE标准中，一个类似Hibernate的持久化API，实际上和Hibernate使用起来没什么区别，只不过JPA是“JavaEE标准的”。Hibernate（JPA）虽然配置略微麻烦，但是使用起来还是比较方便的，Java类和数据库表映射配置完成后，我们写很少的HQL（JPA中是JPQL）代码就能完成比较复杂的业务功能。

然而，Spring Data JPA能够让我们几乎不写任何SQL（HQL、JPQL）就能完成相同的查询功能，尤其是配合SpringBoot使用，开发效率更是相当不错。下面简单介绍一下如何配置和使用Spring Data JPA，例子中并没有使用Web工程，只是创建了一个简单的Java工程并结合使用了Spring框架，但是原理是一样的。

这篇笔记代码较多，但实际上非常简单。

## Spring Data JPA的配置

依赖库如下：

pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.ciyaz.demo</groupId>
	<artifactId>demo</artifactId>
	<version>1.0-SNAPSHOT</version>

	<name>demo</name>
	<!-- FIXME change it to the project's website -->
	<url>http://www.example.com</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>
		<!--Spring框架-->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>5.0.8.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>5.0.8.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>5.0.8.RELEASE</version>
		</dependency>
		<!--MySQL驱动程序-->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
		<!--c3p0数据库连接池-->
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		<!--JPA实现-->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>4.3.11.Final</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>4.3.11.Final</version>
		</dependency>
		<!--Spring Data JPA-->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-jpa</artifactId>
			<version>2.0.9.RELEASE</version>
		</dependency>
	</dependencies>
</project>
```

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
	   http://www.springframework.org/schema/data/jpa
	   http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

	<!--Service层包注解扫描-->
	<context:component-scan base-package="com.ciyaz.demo.service"/>

	<jpa:repositories base-package="com.ciyaz.demo.dao" repository-impl-postfix="Impl"
	                  entity-manager-factory-ref="entityManagerFactory" transaction-manager-ref="txManager"/>

	<!-- 配置C3P0数据库连接池数据源-->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
	      destroy-method="close">
		<property name="driverClass" value="com.mysql.jdbc.Driver"/>
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/demo4"/>
		<property name="user" value="root"/>
		<property name="password" value="root"/>
		<property name="minPoolSize" value="10"/>
		<property name="maxPoolSize" value="100"/>
		<property name="initialPoolSize" value="10"/>
		<property name="maxIdleTime" value="300"/>
		<property name="maxStatements" value="1000"/>
		<property name="idleConnectionTestPeriod" value="60"/>
		<property name="acquireRetryAttempts" value="30"/>
	</bean>

	<!-- JPA实体管理器 -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="packagesToScan" value="com.ciyaz.demo.domain"/>
		<property name="persistenceProvider">
			<bean class="org.hibernate.ejb.HibernatePersistence"/>
		</property>
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<property name="generateDdl" value="false"/>
				<property name="database" value="MYSQL"/>
				<property name="databasePlatform" value="org.hibernate.dialect.MySQL5InnoDBDialect"/>
				<property name="showSql" value="true"/>
			</bean>
		</property>
		<property name="jpaDialect">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect"/>
		</property>
		<property name="jpaPropertyMap">
			<map>
				<entry key="hibernate.query.substitutions" value="true 1, false 0"/>
				<entry key="hibernate.default_batch_fetch_size" value="16"/>
				<entry key="hibernate.max_fetch_depth" value="2"/>
				<entry key="hibernate.generate_statistics" value="true"/>
				<entry key="hibernate.bytecode.use_reflection_optimizer" value="true"/>
				<entry key="hibernate.cache.use_second_level_cache" value="false"/>
				<entry key="hibernate.cache.use_query_cache" value="false"/>
			</map>
		</property>
	</bean>

	<!-- 事务管理器 -->
	<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory"/>
	</bean>

	<!-- 注解驱动的事务支持-->
	<tx:annotation-driven transaction-manager="txManager"/>
</beans>
```

这个配置看起来还是比较复杂，实际上就是在Spring中配置使用JPA，JPA接口则由Hibernate实现。如果使用SpringBoot免去这些复杂的配置就很爽了，使用SpringBoot，除了数据库驱动只需要如下starter依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 使用Spring Data JPA进行增删改查

下面所有的例子都基于以下三个数据表：

```sql
create table t_user (
	user_id  bigint auto_increment,
	username varchar(20),
	password varchar(20),
	birthday datetime,
	primary key (user_id)
);
create table t_role (
	role_id   bigint auto_increment,
	role_name varchar(20),
	primary key (role_id)
);
create table t_user_role (
	user_role_id bigint auto_increment,
	user_id      bigint,
	role_id      bigint,
	primary key (user_role_id)
);
```

其中用户表和角色表具有多对多关系，多对多关系通过一张中间表来表达。Java类和数据库表的映射关系这里使用JPA注解进行配置（有关JPA注解的内容在Hibernate章节都介绍过了）。

User.java
```java
package com.ciyaz.demo.domain;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Date;
import java.util.Set;

@Entity
@Table(name = "t_user")
public class User implements Serializable
{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "user_id")
	private Long userId;
	@Column(name = "username")
	private String username;
	@Column(name = "password")
	private String password;
	@Column(name = "birthday")
	private Date birthday;

	@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
	@JoinTable(name = "t_user_role",
			joinColumns = {@JoinColumn(name = "user_id")},
			inverseJoinColumns = {@JoinColumn(name = "role_id")})
	private Set<Role> roleSet;


	public User()
	{
	}

	public User(String username, String password, Date birthday)
	{
		this.username = username;
		this.password = password;
		this.birthday = birthday;
	}

	public Long getUserId()
	{
		return userId;
	}

	public void setUserId(Long userId)
	{
		this.userId = userId;
	}

	public String getUsername()
	{
		return username;
	}

	public void setUsername(String username)
	{
		this.username = username;
	}

	public String getPassword()
	{
		return password;
	}

	public void setPassword(String password)
	{
		this.password = password;
	}

	public Date getBirthday()
	{
		return birthday;
	}

	public void setBirthday(Date birthday)
	{
		this.birthday = birthday;
	}

	public Set<Role> getRoleSet()
	{
		return roleSet;
	}

	public void setRoleSet(Set<Role> roleSet)
	{
		this.roleSet = roleSet;
	}

	@Override
	public String toString()
	{
		return "User{" +
				"userId=" + userId +
				", username='" + username + '\'' +
				", password='" + password + '\'' +
				", birthday=" + birthday +
				'}';
	}
}
```

Role.java
```java
package com.ciyaz.demo.domain;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Set;

@Entity
@Table(name = "t_role")
public class Role implements Serializable
{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "role_id")
	private Long roleId;
	@Column(name = "role_name")
	private String roleName;

	@ManyToMany(mappedBy = "roleSet", fetch = FetchType.LAZY)
	private Set<User> userSet;

	public Role()
	{
	}

	public Role(String roleName)
	{
		this.roleName = roleName;
	}

	public Long getRoleId()
	{
		return roleId;
	}

	public void setRoleId(Long roleId)
	{
		this.roleId = roleId;
	}

	public String getRoleName()
	{
		return roleName;
	}

	public void setRoleName(String roleName)
	{
		this.roleName = roleName;
	}

	public Set<User> getUserSet()
	{
		return userSet;
	}

	public void setUserSet(Set<User> userSet)
	{
		this.userSet = userSet;
	}

	@Override
	public String toString()
	{
		return "Role{" +
				"roleId=" + roleId +
				", roleName='" + roleName + '\'' +
				'}';
	}
}
```

## 查询的写法

下面代码定义了一个接口，实现了一些常用的Dao功能。

UserRepository.java
```java
package com.ciyaz.demo.dao;

import com.ciyaz.demo.domain.User;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;


public interface UserRepository extends JpaRepository<User, Long>
{

	/**
	 * 根据用户ID查询用户
	 * @param userId 用户ID
	 * @return 用户对象
	 */
	public User findByUserId(Long userId);

	/**
	 * 根据用户名查询用户
	 * @param username 用户名
	 * @return 用户对象
	 */
	public User findByUsername(String username);

	/**
	 * 根据用户名和密码查询用户
	 * @param username 用户名
	 * @param password 密码
	 * @return 用户对象
	 */
	public User findByUsernameAndPassword(String username, String password);

	/**
	 * 模糊查询用户
	 * @param pattern 匹配字符串，如[prefix]%这种形式
	 * @return 用户对象列表
	 */
	public List<User> findByUsernameLike(String pattern);
}
```

注意代码中`UserRepository`这个类的定义，它继承了`JpaRepository`，我们不需要写任何实现类，Spring Data JPA已经为我们做好了，我们只需要在我们的Service层中调用`UserRepository`即可。

这看起来比较神奇，但实际上Spring Data JPA会通过我们定义的方法名自动生成数据库查询，规则是这样的：

* 所有查询方法名以`findBy`开头
* 查询条件写在`findBy`后面，名字需要和Java实体类的字段名相对应，如`findByUsername`
* 多个查询条件可以用`And`或`Or`等连接，如`findByUsernameAndPassword`
* 模糊查询在条件字段后加`Like`，如`findByUsernameLike`

上述规则能够覆盖80%的查询操作了，这就是Spring Data JPA的方便之处。

当然，如果查询条件十分复杂，上面这种写法不能满足的时候，我们也可以写接口的实现类，那就和Hibernate（JPA）没什么区别了，下面例子中我们自己实现了一个接口用于查询。

RoleRepository.java
```java
public Role findByRoleName2(String roleName);
```


```java
package com.ciyaz.demo.dao;

import com.ciyaz.demo.domain.Role;
import org.springframework.beans.factory.annotation.Autowired;

import javax.persistence.EntityManager;
import javax.persistence.Query;

public class RoleRepositoryImpl
{
	@Autowired
	private EntityManager entityManager;

	public Role findByRoleName2(String roleName)
	{
		String jpql = "select r from Role r where r.roleName=:roleName";
		Query query = entityManager.createQuery(jpql);
		query.setParameter("roleName", roleName);
		return (Role) query.getSingleResult();
	}
}
```

注意实现类的写法，我们没有继承任何接口，只是以`接口名Impl`命令了我们的实现类，并且实现类和接口在同一个包中，然后定义了一个相同名字的Java方法的实现，如果我们使用`implements`实现`RoleRepository`，我们需要同时实现大量其他方法（RoleRepository以及其父接口中定义的所有方法）才能编译通过，因此SpringDataJPA内部进行了一些处理，允许我们不使用`implements`指定实现的接口，而保持接口和实现类按照约定命名即可。

这种写法甚至有些脱离Java语法的一般常识了，除非必要否则强烈不建议用。

## 插入的写法

插入使用`save()`方法，和Hibernate（JPA）一样，例子代码如下：

```java
public void addUser(User user, Long[] roleIds)
{
  Set<Role> roleSet = new HashSet<>();
  for(Long l : roleIds)
  {
    Role role = roleRepository.findByRoleId(l);
    if(role != null)
    {
      roleSet.add(role);
    }
  }
  user.setRoleSet(roleSet);
  userRepository.save(user);
}
```

## 删除的写法

我们执行删除操作时，通常也是要附加一些查询条件的，删除的写法和查询差不多：

```java
/**
 * 根据角色名删除角色
 * @param roleName 角色名
 * @return 受影响行数
 */
public int deleteByRoleName(String roleName);
```

## 修改的写法

修改的用法也和Hibernate（JPA）一样，临时态对象修改后使用`save()`保存为持久态，Java对象的修改就会持久化到数据库中了。

## QBE查询

QBE（Query By Example）是一种查询方式，能够直接通过模板对象，从数据库中匹配符合条件的记录，形成结果集。

下面是一段从项目中摘取的一段QBE查询的例子代码，主要实现了分页和条件查询：

```java
@Override
public Map<String, Object> getFileRecordList(Integer pageSize, Integer currentPage, String namePattern, Integer isExist)
{
  // 排序分页
  Sort sort = new Sort(Sort.Direction.ASC, "fileId");
  PageRequest pageRequest = PageRequest.of(currentPage - 1, pageSize, sort);
  // QBE查询
  FileRecord fileRecord = new FileRecord();
  fileRecord.setName(namePattern);
  fileRecord.setIsExist(isExist);
  ExampleMatcher matcher = ExampleMatcher.matching()
      .withMatcher("name", ExampleMatcher.GenericPropertyMatchers.contains())
      .withMatcher("isExist", ExampleMatcher.GenericPropertyMatchers.exact());
  Example<FileRecord> example = Example.of(fileRecord, matcher);

  Page<FileRecord> resultPage = fileRecordRepository.findAll(example, pageRequest);

  Map<String, Object> resultMap = new HashMap<>();
  // 所有记录
  resultMap.put("records", resultPage.getContent());
  // 当前页码
  resultMap.put("currentPage", currentPage);
  // 总记录数
  resultMap.put("totalElements", resultPage.getTotalElements());
  return resultMap;
}
```

## Criteria高级动态查询

有些很复杂的动态查询，JpaRepository和QBE都不能很好的满足我们的需求，而使用拼接JPQL或SQL也是相当麻烦而且不可取的，这时我们可以使用Criteria查询。在Spring Data JPA中，对Criteria操作进行了封装，使用起来还算简单。

想要使用Criteria查询的Repository接口，需要继承`JpaSpecificationExecutor<>`接口，在repository实例上调用`findAll()`方法时，传入一个`Specification<>`对象。具体我们看一个例子，下面例子是来自实际项目中的一段代码，实现了共5个条件的动态查询，查询参数类型包括字符串模糊查询、字符串精确查询、日期比较查询、整数查询：

查询参数封装类型：QueryParam.java
```java
public class QueryParam
{
	private Date startTime;
	private Date endTime;
	private String keyword;
	private Integer publicStatus;
	private Integer starStatus;

	private Integer pageSize;
	private Integer currentPage;

  //getter and setter
}
```

动态查询的写法：
```java
@Override
public Map<String, Object> queryDiaryByPageAndParam(QueryParam queryParam)
{
  // 排序分页
  Sort sort = new Sort(Sort.Direction.ASC, "diaryId");
  PageRequest pageRequest = PageRequest.of(queryParam.getCurrentPage() - 1, queryParam.getPageSize(), sort);
  // 动态查询
  Page<Diary> queryResult = diaryRepository.findAll(new Specification<Diary>()
  {
    @Override
    public Predicate toPredicate(Root<Diary> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder)
    {
      List<Predicate> predicateList = new ArrayList<>();

      // 时间字段大于起始时间
      Predicate startTimePredicate = null;
      if (queryParam.getStartTime() != null)
      {
        startTimePredicate = criteriaBuilder.greaterThanOrEqualTo(root.<Date>get("createTime"), queryParam.getStartTime());
        predicateList.add(startTimePredicate);
      }

      // 时间字段小于结束时间
      Predicate endTimePredicate = null;
      if (queryParam.getEndTime() != null)
      {
        endTimePredicate = criteriaBuilder.lessThanOrEqualTo(root.<Date>get("createTime"), queryParam.getEndTime());
        predicateList.add(endTimePredicate);
      }

      // 标题和内容中包含指定关键字
      Predicate keywordPredicateInTitle = null;
      Predicate keywordPredicateInContent = null;
      if (queryParam.getKeyword() != null && !"".equals(queryParam.getKeyword()))
      {
        keywordPredicateInTitle = criteriaBuilder.like(root.<String>get("title"), "%" + queryParam.getKeyword() + "%");
        keywordPredicateInContent = criteriaBuilder.like(root.<String>get("content"), "%" + queryParam.getKeyword() + "%");

        Predicate keywordPredicate = criteriaBuilder.or(keywordPredicateInTitle, keywordPredicateInContent);
        predicateList.add(keywordPredicate);
      }

      // 具有指定的公开状态
      Predicate publicStatusPredicate = null;
      if (queryParam.getPublicStatus() != null)
      {
        publicStatusPredicate = criteriaBuilder.equal(root.<Integer>get("isPublic"), queryParam.getPublicStatus());
        predicateList.add(publicStatusPredicate);
      }
      // 具有指定的加星状态
      Predicate starStatusPredicate = null;
      if (queryParam.getStarStatus() != null)
      {
        starStatusPredicate = criteriaBuilder.equal(root.<Integer>get("isStar"), queryParam.getStarStatus());
        predicateList.add(starStatusPredicate);
      }
      // 动态查询条件合并
      int predicateSize = predicateList.size();
      if (predicateSize > 0)
      {
        Predicate predicate = criteriaBuilder.and((Predicate[]) predicateList.toArray(new Predicate[predicateSize]));
        // 将查询条件提交给查询计划
        query.where(predicate);
      }
      return null;
    }
  }, pageRequest);

  Map<String, Object> resultMap = new HashMap<>(64);
  // 所有记录
  resultMap.put("records", queryResult.getContent());
  // 当前页码
  resultMap.put("currentPage", queryParam.getCurrentPage());
  // 总记录数
  resultMap.put("totalElements", queryResult.getTotalElements());
  return resultMap;
}
```

## 总结

Spring Data JPA非常适合配合SpringBoot快速实现简单、不追求性能的需求；相反，在SQL查询十分复杂、性能要求高的场景，Spring Data JPA就不适用了，因为Spring Data JPA基于方法名自动实现JPQL查询难以表达查询参数非常多、非常复杂的情况，而编写实现类就和直接使用Hibernate（JPA）差不多麻烦了；除此之外，毕竟是基于Hibernate（JPA），性能优化也是个麻烦的问题，需要性能的情况推荐使用JDBC、MyBatis的方式。
