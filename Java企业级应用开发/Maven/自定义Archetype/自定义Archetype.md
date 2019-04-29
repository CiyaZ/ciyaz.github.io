# 自定义Archetype

当我们的Java项目比较复杂时，每次新建一个工程都要耗费大量精力去建好对应目录，加入依赖，编写配置文件，这个过程可能耗时一上午甚至更久。

我们知道Maven提供了项目骨架（Archetype）功能，我们经常使用的`maven-archetype-webapp`和`maven-archetype-quickstart`能够帮助我们快速生成基于Maven的Web项目或是控制台项目。但是这两个简易的通用模板是无法满足特定团队特定需求的。因此，我们需要定制一个自己的项目骨架。

注：该操作这里讲的可能有不详细的地方，我也没办法。这个Maven模板定制起来真的很困难，这种高灵活度的操作正是Maven不擅长的，尤其是项目非常复杂的时候。文字表达能力有限，只可意会难以言传，具体还要自己操作，坑上一两天时间才能搞明白。

## 编写项目骨架的流程

编写一个项目骨架的基本流程：

1. 编写一个拥有全功能的精简项目并测试通过
2. 抽取项目骨架
3. 定制项目骨架

我们这里通常假设要为一个相对比较复杂的`Maven多模块`项目抽取骨架。

## 编写精简项目

这一步比较简单，但也是最麻烦的步骤。举个例子，我们要创建一个SSM项目，按Maven多模块进行划分，数据库使用MySQL，缓存使用Redis，服务治理使用阿里的Dubbo，消息队列要用到RabbitMQ和Kafka，还要接入项目的单点登录，更多的细节就不说了......

现在你要抽取一个这种「大型」项目的骨架，你建完项目后，还要写一个Demo，事无巨细的把上面的功能全都实现一遍，并确认它好使。或者也可以直接从一个现有的项目往下删文件，达到抽取骨架的目的。

具体就不详细叙述了。

## 抽取骨架

抽取骨架要用到Archetype插件，但不需要显示写在配置文件中，直接在命令行中执行即可。

```
mvn clean archetype:create-from-project
```

这会在项目`根目录/target/generated-sources/archetype`下生成一个新的Maven工程，它就是真正的Archetype工程，我们可以把它拷贝出来使用。

## 定制项目骨架

我这里也只能简单介绍下我所用到的内容，可能不太全面。

首先是一个重要的配置文件：src/main/resources/META-INF/maven/archetype-metadata.xml，这个文件定义的就是我们从骨架到项目的生成规则。还是比较容易看懂的，具体就不详细叙述了，其原理就是替换，包括配置文件中字符串的替换，以及文件目录名的替换。

我们要知道几个用于替换的占位变量：

* groupId：用骨架生成时指定的项目组名
* artifactId：用骨架生成时指定的项目名（如果是多模块项目，值是模块项目名）
* rootArtifactId：用骨架生成时指定的根项目名

文件中，可以使用类似`${rootArtifactId}`的形式指定占位变量，文件夹则可以使用类似`__rootArtifactId__`的形式指定，指定位置会在骨架生成项目时替换。

## 使用项目骨架

使用如下命令调用项目骨架：

```
mvn archetype:generate \
-DarchetypeCatalog=local \
-DarchetypeGroupId=<骨架groupId> \
-DarchetypeArtifactId=<骨架artifactId> \
-DarchetypeVersion=<骨架版本> \
-DgroupId=<项目groupId> \
-DartifactId=<项目artifactId> \
-DinteractiveMode=false
```

注意：使用`-DarchetypeCatalog=local`参数主要是因为我在测试骨架的时候，只在本地进行了安装，入过将骨架部署到Nexus或中央仓库，就不需要这个参数了。

## 主要注意问题

### Java类的包名

对于Maven多模块项目，Java类的自动包名生成规则有点问题。比如`pro-api`模块下，有控制器类`com.ciyaz.pro.controller.UserController`，我们要指定这样的`package`：

```
package ${groupId}.${rootArtifactId}.controller;
```

同时，骨架工程文件夹也要对应修改为`src/main/java/__rootArtifactId__/controller/UserController.java`。

另外要注意，路径中没有`groupId`的指定，这是因为骨架有一个默认的`package`，它会默认加在Java包最开始，我们一般`groupId`就是这个`package`值，因此不用指定`__groupId__`这种文件夹。

### 空目录

骨架中，某个模块可能只建了文件夹，例如`src/main/java`，但没有文件，骨架默认不生成这些文件，如果我们又想让骨架把这些文件给生成出来，应该怎么做呢？

配置`archetype`插件，允许生成空目录。

pom.xml
```xml
<plugin>
  <artifactId>maven-archetype-plugin</artifactId>
  <version>3.0.1</version>
  <configuration>
    <includeEmptyDirs>true</includeEmptyDirs>
  </configuration>
</plugin>
```

骨架配置中，加一个`fileSet`指定生成空目录。

archetype-metadata.xml
```xml
<fileSet filtered="true" encoding="UTF-8">
  <directory>src/main/java</directory>
  <includes>
    <include>**/*.java</include>
  </includes>
</fileSet>
```
