# SpringBoot简介

## 什么是SpringBoot

`SpringBoot`是一个先进的Java开发框架，用于简化基于Spring的应用程序开发。

假如我们使用类似`SpringMVC+Spring+JPA`这种开发模式，开发者首先需要面对的就是项目的依赖库，加入我们使用`Maven`进行依赖管理和构建，我们需要花费大量的时间，整理好我们项目需要的依赖包，然后把依赖配置添加到`pom.xml`中。之后，我们还必须面对复杂的配置文件，我们至少需要`web.xml`，`applicationContext.xml`，`<核心转发servlet名>-servlet.xml`这三个冗长的配置文件，最熟练的程序员也至少需要半个小时，才能把一个项目的框架搭建起来。

SpringBoot集成了各种方便的功能，能够帮助我们快速搭建和配置好一个Spring工程，我们写的代码实际上还是Spring那一套。

## SpringBoot的使用场景

目前来看，SpringBoot的主要应用场景之一就是APP、微信公众号、网页应用等的后台API服务器。虽然SpringBoot也支持FreeMarker、Thymeleaf等模板，但是实际情况中用的不多。

## 使用SpringBoot的优势

### 配置简化

SpringBoot充分考虑了开发者的习惯，内置了大量的默认配置，取代了繁琐的XML配置，对熟悉Spring的开发者相当友好。

### 启动依赖 spring-boot-starter

SpringBoot的启动依赖利用了传递依赖解析，能够自动把常用的库添加到我们的项目中，简化了依赖的配置。

### Java的优秀生态

SpringBoot出现的比`Django`等框架晚的多，Java阵营中，SpringBoot这种看似新鲜的概念，实际上早已玩烂许多年，但SpringBoot还是一个巨大的进步，Java是全世界使用人数最多的编程语言，也是生态最丰富最优秀的语言之一，SpringBoot的出现会给相关领域带来巨大的变化。
