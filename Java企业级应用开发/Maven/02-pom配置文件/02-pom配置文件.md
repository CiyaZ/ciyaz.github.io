# pom配置文件

我们编写的Maven工程具有一个配置文件`pom.xml`，使用`mvn`命令时，我们必须在这个有`pom.xml`的目录中，我们在这个文件中进行依赖包、Maven插件的配置。

下面是一个`pom.xml`的例子：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.ciyaz.demo</groupId>
	<artifactId>maven-demo</artifactId>
	<version>1.0-SNAPSHOT</version>

	<name>maven-demo</name>
	<url>http://www.example.com</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>
		<dependency>
			<groupId>commons-codec</groupId>
			<artifactId>commons-codec</artifactId>
			<version>1.11</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-clean-plugin</artifactId>
				<version>3.0.0</version>
			</plugin>
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<version>3.0.2</version>
			</plugin>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.7.0</version>
			</plugin>
			<plugin>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.20.1</version>
			</plugin>
			<plugin>
				<artifactId>maven-jar-plugin</artifactId>
				<version>3.0.2</version>
			</plugin>
			<plugin>
				<artifactId>maven-install-plugin</artifactId>
				<version>2.5.2</version>
			</plugin>
			<plugin>
				<artifactId>maven-deploy-plugin</artifactId>
				<version>2.8.2</version>
			</plugin>
		</plugins>
	</build>
</project>
```

## 项目坐标

Maven中央仓库中，`groupId`，`artifactId`，`version`三个标签就能确定一个唯一的项目。

* groupId：这个标签意义上用于标识一个项目，通常是域名+项目名，比如`org.apache.maven.plugins`
* artifactId：这个标签用于标识一个项目的模块，命名为模块名
* version：这个标签用来标识一个版本号

## 打包方式

`<packaging>`标签用于指示项目的打包方式，常用的有`jar`和`war`，默认情况下为`jar`包（上面代码中没有写`packaging`就是默认情况了）。除此之外，对于一些特殊的打包方式，比如可执行`jar`，就需要直接写一些配置信息给`maven-jar-plugin`了。

## 依赖

在`<dependencies>`标签里配置的就是项目的所有依赖，每个依赖用`<dependency>`标签配置，其中`groupId`、`artifactId`、`version`就是这个依赖包的坐标。

* `scope`：用于指定这个包的作用于
  * `compile`：这个依赖参与项目的编译构建
  * `test`：这个依赖仅在运行单元测试时参与编译构建，如JUnit
  * `runtime`：不参与编译但运行时需要，比如数据库的JDBC驱动，我们编写代码时使用的是JDBC接口，其实现在数据库驱动中，我们并不直接调用，因此并不参与编译，但是运行时是必须的
  * `provided`：参与编译，但运行时由其他组件提供，比如Tomcat的库，如果我们的项目最终需要部署到Tomcat上，应用服务器的库已经由服务器提供了，我们没必要再在工程里打包进去了
  * `system`：类似于`provided`，但是依赖库从本地文件系统取得，比如项目的`lib`目录，需要配合`systemPath`使用

## 构建

在`<build>`标签中，编写的就是和构建相关的配置，其中有很多参数如：`<defaultGoal>`等，但我们很少修改其默认值。我们一般在`<build>`中配置的就是Maven插件，插件配置在`<plugins>`中的`<plugin>`标签中，具体内容包括插件的Maven坐标，以及一个包含插件配置信息的`<configuration>`标签。有关Maven插件的内容会在后续章节详细介绍。

上面对于插件的配置仅仅是为了固定插件的版本，Maven为各个生命周期都绑定了一些默认插件，但是插件的版本都是自动选择的。

`<plugins>`和`<pluginManagemenrt>`的区别：`<pluginManagemenrt>`可作为父配置用于继承，其余两者无区别。

## 关于<properties>

上面代码中，在`<properties>`中指示了源码的文件编码和编译等级，实际上这些配置是用于指示Maven插件的，例如对于编译等级，我们也可以直接对`maven-compiler-plugin`进行配置。

## 还有很多其他配置

Maven还有很多其他的配置，这里就不一一介绍了。
