# java.util.logging.Logger

Java的生态中，日志系统有log4j，log4j2，slf4j，logback，除此之外还有JDK自带的`java.util.logging.Logger`，JDK自带的日志系统虽然和前面的比起来在性能、功能上都不足，但是它使用简单，推荐优先考虑。

## 日志的级别

为了便于用户有效的利用日志，日志信息是分级的，比如系统报错和系统正常运行时的日志，肯定不是一个级别的，系统管理员在分析系统报错时，只要查找较高级别的日志即可。

JDK自带的日志系统中，日志分为如下级别（从高到低）：

* SEVERE(严重)
* WARNING（警告）
* INFO（信息）
* CONFIG
* FINE
* FINER
* FINEST

实际上，我们一般只使用前三种，系统发生致命错误时，输出`SEVERE`级别，系统存在异常但仍能正常运行时，输出`WARNING`级别，系统正常运行，输出`INFO`级别。

## 使用代码输出日志

下面代码中，测试了输出各种级别的日志。

```java
import java.util.logging.Logger;

public class Main
{

	private static void foo()
	{
		Logger logger = Logger.getLogger("main-app");
		logger.severe("这是severe级别的日志");
		logger.warning("这是warning级别的日志");
		logger.info("这是info级别的日志");
		logger.config("这是config级别的日志");
		logger.fine("这是fine级别的日志");
		logger.finer("这是finer级别的日志");
		logger.finest("这是finest级别的日志");
	}

	public static void main(String[] args)
	{
		foo();
	}
}
```

输出：
```
八月 04, 2018 10:22:05 下午 Main foo
严重: 这是severe级别的日志
八月 04, 2018 10:22:05 下午 Main foo
警告: 这是warning级别的日志
八月 04, 2018 10:22:05 下午 Main foo
信息: 这是info级别的日志
```

我们可以发现，默认控制台中只输出了`SEVERE`、`WARNING`、`INFO`这三个级别的日志。这是因为，Logger输出的默认级别是`INFO`，比`INFO`低级的日志不会输出。

## Handler

`Handler`负责对生成的日志信息进行“处置”，默认的处置方式是打印到控制台，除此之外，还可以将日志写入文件，或是发送到网络。

下面代码中，为一个`Logger`添加了一个`FileHandler`，将生成的日志信息写入文件。生成的日志是XML格式的。

```java
public class Main
{

	private static void foo() throws IOException
	{
		Logger logger = Logger.getLogger("main-app");

		FileHandler handler = new FileHandler("/home/ciyaz/1.log");
		logger.addHandler(handler);

		logger.severe("这是severe级别的日志");
		logger.warning("这是warning级别的日志");
		logger.info("这是info级别的日志");
		logger.config("这是config级别的日志");
		logger.fine("这是fine级别的日志");
		logger.finer("这是finer级别的日志");
		logger.finest("这是finest级别的日志");
	}

	public static void main(String[] args) throws IOException
	{
		foo();
	}
}
```

1.log
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
<record>
  <date>2018-08-04T22:32:01</date>
  <millis>1533393121530</millis>
  <sequence>0</sequence>
  <logger>main-app</logger>
  <level>SEVERE</level>
  <class>Main</class>
  <method>foo</method>
  <thread>1</thread>
  <message>这是severe级别的日志</message>
</record>
<record>
  <date>2018-08-04T22:32:01</date>
  <millis>1533393121585</millis>
  <sequence>1</sequence>
  <logger>main-app</logger>
  <level>WARNING</level>
  <class>Main</class>
  <method>foo</method>
  <thread>1</thread>
  <message>这是warning级别的日志</message>
</record>
<record>
  <date>2018-08-04T22:32:01</date>
  <millis>1533393121586</millis>
  <sequence>2</sequence>
  <logger>main-app</logger>
  <level>INFO</level>
  <class>Main</class>
  <method>foo</method>
  <thread>1</thread>
  <message>这是info级别的日志</message>
</record>
</log>
```

## Formatter

`Formatter`负责对输出的日志数据格式化，例如上面将日志写入文件中，默认使用的是XML格式，这是因为默认配置了`java.util.logging.XMLFormatter`，如果我们不想用XML格式，我们可以选用其他的`Formatter`。

下面例子中，改用简单模式输出日志数据到文件中
```java
public class Main
{

	private static void foo() throws IOException
	{
		Logger logger = Logger.getLogger("main-app");

		FileHandler handler = new FileHandler("/home/ciyaz/1.log");
		SimpleFormatter formatter = new SimpleFormatter();
		handler.setFormatter(formatter);
		logger.addHandler(handler);

		logger.severe("这是severe级别的日志");
		logger.warning("这是warning级别的日志");
		logger.info("这是info级别的日志");
		logger.config("这是config级别的日志");
		logger.fine("这是fine级别的日志");
		logger.finer("这是finer级别的日志");
		logger.finest("这是finest级别的日志");
	}

	public static void main(String[] args) throws IOException
	{
		foo();
	}
}
```

和上面代码的区别就是为`FileHandler`添加了一个`SimpleFormatter`，覆盖了默认的`XMLFormatter`。

1.log
```
八月 04, 2018 10:37:37 下午 Main foo
严重: 这是severe级别的日志
八月 04, 2018 10:37:37 下午 Main foo
警告: 这是warning级别的日志
八月 04, 2018 10:37:37 下午 Main foo
信息: 这是info级别的日志
```

## logging.properties

通过上文，我们发现这个JDK自带的Logger有很多默认配置，实际上这些配置都写在`$JAVA_HOME/jre/lib/logging.properties`里。

```
############################################################
#  	Default Logging Configuration File
#
# You can use a different file by specifying a filename
# with the java.util.logging.config.file system property.  
# For example java -Djava.util.logging.config.file=myfile
############################################################

############################################################
#  	Global properties
############################################################

# "handlers" specifies a comma separated list of log Handler
# classes.  These handlers will be installed during VM startup.
# Note that these classes must be on the system classpath.
# By default we only configure a ConsoleHandler, which will only
# show messages at the INFO and above levels.
handlers= java.util.logging.ConsoleHandler

# To also add the FileHandler, use the following line instead.
#handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

# Default global logging level.
# This specifies which kinds of events are logged across
# all loggers.  For any given facility this global level
# can be overriden by a facility specific level
# Note that the ConsoleHandler also has a separate level
# setting to limit messages printed to the console.
.level= INFO

############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

# default file output is in user's home directory.
java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter

# Limit the message that are printed on the console to INFO and above.
java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# Example to customize the SimpleFormatter output format
# to print one-line log message like this:
#     <level>: <log message> [<date/time>]
#
# java.util.logging.SimpleFormatter.format=%4$s: %5$s [%1$tc]%n

############################################################
# Facility specific properties.
# Provides extra control for each logger.
############################################################

# For example, set the com.xyz.foo logger to only log SEVERE
# messages:
com.xyz.foo.level = SEVERE
```

里面的具体内容就不详细解释了，自带的注释已经很详尽了，我们也可以根据注释的指引，让我们的应用程序使用我们自定义的日志配置文件。
