# MyBatis与Spring集成

这篇笔记的例子中，集成了MyBatis、Spring、SpringMVC，搭建了一个Web工程。工程放在参考[https://gitee.com/ciyaz/ssm-demo](https://gitee.com/ciyaz/ssm-demo)，具体没什么好描述的，直接用IDE导入Maven项目看代码即可。

主要配置都在applicationContext.xml中，下面是一个例子：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- 业务层注解扫描-->
	<context:component-scan base-package="com.ciyaz.mbdemo2.service"/>

	<!-- 配置数据源使用C3P0数据库连接池-->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
	      destroy-method="close">
		<!-- 数据库驱动-->
		<property name="driverClass" value="com.mysql.jdbc.Driver"/>
		<!-- 数据库连接URL-->
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/mbdemo2"/>
		<!-- 用户名-->
		<property name="user" value="root"/>
		<!-- 密码-->
		<property name="password" value="root"/>
		<!-- 连接池最小维持连接数-->
		<property name="minPoolSize" value="10"/>
		<!-- 连接池最大维持连接数-->
		<property name="maxPoolSize" value="100"/>
		<!-- 连接池初始连接数-->
		<property name="initialPoolSize" value="10"/>
		<!-- 连接最大空闲时间-->
		<property name="maxIdleTime" value="300"/>
		<!-- 数据源已加载预编译SQL语句最大数量-->
		<property name="maxStatements" value="1000"/>
		<!-- 空闲连接检查时间间隔-->
		<property name="idleConnectionTestPeriod" value="60"/>
		<!--获取新连接失败时的重试次数-->
		<property name="acquireRetryAttempts" value="30"/>
	</bean>

	<!--配置MyBatis的SqlSessionFactory-->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!--数据源-->
		<property name="dataSource" ref="dataSource"/>
		<!--mybatis-config.xml文件位置-->
		<property name="configLocation" value="classpath:mybatis-config.xml" />
		<!--mapper.xml文件位置-->
		<property name="mapperLocations">
			<array>
				<value>classpath:com/ciyaz/mbdemo2/mapper/*.xml</value>
			</array>
		</property>
		<!--该包下所有的类使用类名作为类全名的别名-->
		<property name="typeAliasesPackage" value="com.ciyaz.mbdemo2.domain"/>
	</bean>

	<!-- 配置Mapper扫描器 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!--扫描的包名-->
		<property name="basePackage" value="com.ciyaz.mbdemo2.mapper"/>
		<!--上面配置的bean名，注意这个配置比较怪，参数是bean名字符串，而不是引用-->
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
	</bean>

	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!-- 注解驱动的事务支持-->
	<tx:annotation-driven transaction-manager="txManager"/>

</beans>
```
