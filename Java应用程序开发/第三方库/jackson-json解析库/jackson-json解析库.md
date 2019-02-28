# jackson解析库

jackson是一个流行的json解析库，SpringMVC处理和json相关的数据时，默认就需要使用jackson。我们不需要太多的高级用法，本篇笔记记录jackson库的简单使用。

## 添加jackson依赖

到目前为止，jackson的最新版本是2.8.9

```java
compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: '2.8.9'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.8.9'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-annotations', version: '2.8.9'
```

jackson主要包含这三个模块：

* jackson-core 核心模块，提供底层的streaming API
* jackson-annotations 提供了注解支持
* jackson-databind 实现了数据绑定和对象序列化API，依赖于core和annotations模块

我们可以根据需要，使用相应的模块。

## 处理json的三种方式

jackson提供了三种处理json的方式

1. 流式解析 streaming API
2. 文档树解析 Tree Model
3. 对象数据绑定 Data Binding

这几种解析方式和XML的SAX，DOM，和对象映射意思差不多。

json作为一种数据交换格式，最常用到的地方就是传输数据，比如：前台的JavaScript对象直接序列化为json，传递到后台后，后台直接映射到一个Java对象上，因此Data Binding是最常用到的解析方式。这和XML不同，XML作为配置文件用的更多。

## 映射json字符串到Java类

### 最简单的例子

实体类User
```java
class User
{
	private String username;
	private String password;

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

	@Override
	public String toString()
	{
		return "User{" +
				"username='" + username + '\'' +
				", password='" + password + '\'' +
				'}';
	}
}
```

user.json
```json
{
  "username":"tom",
  "password":"123"
}
```

Main.java
```java
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.io.InputStream;

public class Main
{
	public static void main(String[] args) throws IOException
	{
		ObjectMapper objectMapper = new ObjectMapper();

		InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("user.json");
		User user = objectMapper.readValue(inputStream, User.class);
		System.out.println(user);
	}
}
```

`ObjectMapper`就是用于将json字符串映射到对象的工具类。

`objectMapper.readValue()`有很多重载形式，上面我们传入的是输入流类型，实际上另一个常用的方式是直接传入json字符串。返回的结果就是填充好的Java对象。如果解析过程或者映射过程出错，都会抛出异常。

映射的过程中，实际上我们没有指定哪个json字段映射到哪个Java类的属性，但是json默认会将同名的字段和属性进行映射。实际上我们也不需要手动指定，我们约定json字段和Java类属性名保持一致即可。

### json数组和ArrayList

Teacher实体类
```java
class Teacher
{
	private String teacherName;
	private List<Student> studentList;

	public String getTeacherName()
	{
		return teacherName;
	}

	public void setTeacherName(String teacherName)
	{
		this.teacherName = teacherName;
	}

	public List<Student> getStudentList()
	{
		return studentList;
	}

	public void setStudentList(List<Student> studentList)
	{
		this.studentList = studentList;
	}

	@Override
	public String toString()
	{
		return "Teacher{" +
				"teacherName='" + teacherName + '\'' +
				", studentList=" + studentList +
				'}';
	}
}
```

Student实体类
```java
class Student
{
	private String studentName;
	private int score;

	public String getStudentName()
	{
		return studentName;
	}

	public void setStudentName(String studentName)
	{
		this.studentName = studentName;
	}

	public int getScore()
	{
		return score;
	}

	public void setScore(int score)
	{
		this.score = score;
	}

	@Override
	public String toString()
	{
		return "Student{" +
				"studentName='" + studentName + '\'' +
				", score=" + score +
				'}';
	}
}
```

teacher.json
```json
{
  "teacherName":"Tom",
  "studentList":[
	{
	  "studentName":"Kate",
	  "score":100
	},
	{
	  "studentName":"Bob",
	  "score":90
	},
	{
	  "studentName":"Lucy",
	  "score":20
	}
  ]
}
```

解析teacher.json
```java
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class Main
{
	public static void main(String[] args) throws IOException
	{
		ObjectMapper objectMapper = new ObjectMapper();

		InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("teacher.json");
		Teacher teacher = objectMapper.readValue(inputStream, Teacher.class);
		System.out.println(teacher);
	}
}
```

Teacher对象有一个`studentList`属性，ObjectMapper比较智能，能够自动把json数组映射到ArrayList上，我们也不需要做什么额外的工作。

## 序列化对象为json

有了前面解析json的知识，实际上序列化Java对象到json也是类似的。

```java
String jsonStr = objectMapper.writeValueAsString(teacher);
```

`objectMapper.writeValueAsString()`可以向一个对象按照jackson默认的规则序列化为json。我们一般约定，对象属性和json字段名相同，因此我们其实也不需要修改jackson的默认规则。

## 有关日期

日期类型也经常出现在实体类中，jackson默认能解析时间戳（一个很大的整数），其结果是java.util.Date类型。序列化Java对象时，我们可以对`ObjectMapper`指定`DateFormat`，输出一个代表时间的字符串。

time.json
```json
{
  "name":"timejson",
  "createTime":1363917730000
}
```

TimeObject实体类
```java
class TimeObject
{
	private String name;
	private Date createTime;

	public String getName()
	{
		return name;
	}

	public void setName(String name)
	{
		this.name = name;
	}

	public Date getCreateTime()
	{
		return createTime;
	}

	public void setCreateTime(Date createTime)
	{
		this.createTime = createTime;
	}

	@Override
	public String toString()
	{
		return "TimeObject{" +
				"name='" + name + '\'' +
				", createTime=" + createTime +
				'}';
	}
}
```

解析json
```java
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;

public class Main
{
	public static void main(String[] args) throws IOException
	{
		ObjectMapper objectMapper = new ObjectMapper();

		InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("time.json");
		TimeObject timeObject = objectMapper.readValue(inputStream, TimeObject.class);
		System.out.println(timeObject);
	}
}
```

序列化为json字符串
```java
objectMapper.writeValueAsString(timeObject)
```

序列化的输出结果：
```
{"name":"timejson","createTime":"2013-03-22"}
```
