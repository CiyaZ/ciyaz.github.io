# XML和JSON

xml和json都是流行的数据传输、存储格式。经常用于客户端和服务器交换数据和本地存储数据。实际上，xml更常用于配置文件的编写，或者作为某种结构化的文档存储数据，而json更常用于网络应用中和服务器交互数据。

# XML的生成和解析

Android中使用XmlSerializer生成xml文件，使用XmlPullParser解析xml文件。

生成和解析XML例子：
```java
public void createXML()
{
	FileOutputStream fileOutputStream = null;
	try
	{
		fileOutputStream = openFileOutput("user.xml", MODE_PRIVATE);
		XmlSerializer xmlSerializer = Xml.newSerializer();
		xmlSerializer.setOutput(fileOutputStream, "UTF-8");

		//encoding 编码 standalone 是否引用约束文件
		xmlSerializer.startDocument("UTF-8", true);

		//namespace 名称空间 name 标签名
		xmlSerializer.startTag(null, "users");
		xmlSerializer.startTag(null, "user");
		xmlSerializer.startTag(null, "name");
		xmlSerializer.text("admin");
		xmlSerializer.endTag(null, "name");
		xmlSerializer.startTag(null, "password");
		xmlSerializer.attribute(null, "test_attr", "test_value");
		xmlSerializer.text("123");
		xmlSerializer.endTag(null, "password");
		xmlSerializer.endTag(null, "user");
		xmlSerializer.endTag(null, "users");

		xmlSerializer.endDocument();
		xmlSerializer.flush();
	}
	catch (Exception e)
	{
		e.printStackTrace();
	}
	finally
	{
		if(fileOutputStream != null)
		{
			try
			{
				fileOutputStream.close();
			}
			catch (IOException e)
			{
				e.printStackTrace();
			}
		}
	}
}

public void parseXML()
{
	FileInputStream fileInputStream = null;
	try
	{
		fileInputStream = openFileInput("user.xml");
		XmlPullParser xmlPullParser = Xml.newPullParser();
		xmlPullParser.setInput(fileInputStream, "UTF-8");

		int eventType = xmlPullParser.getEventType();
		while(eventType != XmlPullParser.END_DOCUMENT)
		{
			String name = xmlPullParser.getName();
			if(eventType == XmlPullParser.START_TAG)
			{
				if("name".equals(name))
				{
					show(xmlPullParser.nextText());
				}
				if("password".equals(name))
				{
					show(xmlPullParser.getAttributeValue(null, "test_attr"));
					show(xmlPullParser.nextText());
				}
			}
			eventType = xmlPullParser.next();
		}

	}
	catch (Exception e)
	{
		e.printStackTrace();
	}
	finally
	{
		try
		{
			fileInputStream.close();
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
```

注：代码中show()其实就是使用吐司显示一下数据。实际开发中，应该根据实际需求，安排pull解析器的各种事件处理。

## JSON的生成和解析

Android中使用JsonObject进行json的生成和解析。这个JsonObject是Android框架提供的，在JavaSE中是不存在的。

生成和解析json：
```java
public void createJSON()
{
	JSONObject jsonObject = new JSONObject();
	JSONArray users = new JSONArray();
	JSONObject user = new JSONObject();
	try
	{
		user.put("username", "admin");
		user.put("password", "123");
		users.put(user);
		jsonObject.put("users", users);

		String json = jsonObject.toString();
		FileOutputStream fileOutputStream = openFileOutput("test.json", MODE_PRIVATE);
		fileOutputStream.write(json.getBytes());
		fileOutputStream.close();
	}
	catch (Exception e)
	{
		e.printStackTrace();
	}
}

public void parseJSON()
{
	try
	{
		FileInputStream fileInputStream = openFileInput("test.json");
		String json = "";
		byte[] buffer = new byte[1024];
		int read;
		while((read = fileInputStream.read(buffer)) != -1)
		{
			json += new String(buffer, 0, read);
		}
		fileInputStream.close();

		JSONObject jsonObject = new JSONObject(json);
		JSONArray users = jsonObject.getJSONArray("users");
		for(int i = 0; i < users.length(); i++)
		{
			JSONObject user = (JSONObject) users.get(i);
			String name = user.getString("username");
			String password = user.getString("password");
			show(name + " " + password);
		}

	}
	catch (Exception e)
	{
		e.printStackTrace();
	}
}
```

总体来看JsonObject使用起来还是比较麻烦的，我们生成json和解析json，就像一步步搭积木，再一步步拆掉。我们可能需要直接将json映射成Java类的工具，这时可以看看Jackson库，参考：`/Java/第三方库/jackson-json解析库`。
