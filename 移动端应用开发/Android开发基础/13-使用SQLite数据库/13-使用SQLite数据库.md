# SQLite

在客户端编程中经常使用轻量级数据库SQLite，其优点是体积小，使用文件存储，十分轻便，缺点是功能较弱，性能较差，但客户端上并没有性能要求，因此SQLite十分合适。

Android编程中，是没有JDBC的，数据的持久化API是Android框架集成SQLite实现的，因此我们直接就能使用SQLite，但在不添加额外的Java库时，也只能使用它。Android框架提供的持久化API和JDBC实际上比较像，里面有很多相同的概念，比如：用占位符组织SQL语句，查询结果集的游标等概念。

# Android中使用SQLite

1. 继承SQLiteOpenHelper类，填充其中的回调函数
2. 初始化自己的SQLiteOpenHelper，获取SQLiteDatabase对象，执行增删改查

## 初始化数据库

MyOpenHelper.java
```java
public class MyOpenHelper extends SQLiteOpenHelper
{

	public MyOpenHelper(Context context)
	{
		//context 上下文对象 name 数据库名 factory 传null即可 version 版本号
		super(context, "test.db", null, 1);
	}

	/**
	 * 数据库创建时执行
	 * @param db 数据库对象
	 */
	@Override
	public void onCreate(SQLiteDatabase db)
	{
		String createDatabase =
				"create table t_user(" +
				"_id integer primary key autoincrement," +
				"username varchar(20)," +
				"birthday integer" +
				");";
		db.execSQL(createDatabase);
	}

	/**
	 * 数据库版本号发生改变时回调
	 * @param db 数据库对象
	 * @param oldVersion 旧版本号
	 * @param newVersion 新版本号
	 */
	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)
	{
		//执行更新使用的数据库版本号
		db.execSQL("alter table t_user add phone varchar(11)");
	}
}
```

数据库版本号的概念：应用版本升级，如果旧版本数据库已经存在，而新版本应用有更高的数据库版本号，就执行onUpdate()，如果是全新安装，以前的数据库还不存在，就执行onCreate()。

## 执行数据库增删改查操作

UserDao.java
```java
public class UserDao
{
	private SQLiteOpenHelper sqLiteOpenHelper = null;

	public UserDao(Context context)
	{
		this.sqLiteOpenHelper = new MyOpenHelper(context);
	}

	/**
	 * 按名字查询用户
	 * @param username 用户名
	 * @return 包含所有相同名字用户的列表
	 */
	public List<User> queryFromName(String username)
	{
		List<User> userList = new ArrayList<>();

		SQLiteDatabase db = sqLiteOpenHelper.getReadableDatabase();
		String queryString = "select _id, birthday from t_user where username=?;";

		//rawQuery() 更灵活更适合查询 query() 不灵活
		//sql 包含占位符的SQL语句 selectionArgs 参数数组
		Cursor cursor = db.rawQuery(queryString, new String[]{username});

		if(cursor != null && cursor.getCount() > 0)
		{
			while(cursor.moveToNext())
			{
				int id = cursor.getInt(0);
				long birthdayInt = cursor.getLong(1);
				Date birthday = new Date(birthdayInt);
				userList.add(new User(id, username, birthday));
			}
			cursor.close();
		}
		db.close();
		return userList;
	}

	/**
	 * 添加用户
	 * @param user 用户对象
	 * @return 添加成功返回True
	 */
	public boolean add(User user)
	{
		SQLiteDatabase db = sqLiteOpenHelper.getWritableDatabase();

		//ContentValues对象包含插入的参数
		ContentValues contentValues = new ContentValues();
		contentValues.put("username", user.getUsername());
		contentValues.put("birthday", user.getBirthday().getTime());

		//插入操作 table 表名 nullColumnHack 返回null时的默认值,传null即可 values 包含参数的ContentValues对象
		long result = db.insert("t_user", null, contentValues);
		db.close();

		boolean success = true;
		if(result == -1)
		{
			success = false;
		}
		return success;
	}

	/**
	 * 按Id删除用户
	 * @param id 用户id
	 * @return 成功删除多少行
	 */
	public int delete(int id)
	{
		SQLiteDatabase db = sqLiteOpenHelper.getWritableDatabase();

		String idStr = Integer.toString(id);
		//table 表名 whereClause where条件 whereArgs 条件参数,必须是String类型
		int result = db.delete("t_user", "_id=?", new String[]{idStr});

		db.close();
		return result;
	}

	/**
	 * 根据对象中主键更新用户信息
	 * @param user 用户对象
	 * @return 操作影响多少行
	 */
	public int update(User user)
	{
		SQLiteDatabase db = sqLiteOpenHelper.getWritableDatabase();

		ContentValues contentValues = new ContentValues();
		contentValues.put("username", user.getUsername());
		contentValues.put("birthday", user.getBirthday().getTime());
		String idStr = Integer.toString(user.getId());
		//table 表名 values 修改的参数值 whereClause where条件 whereArgs 条件参数,必须是String类型
		int result = db.update("t_user", contentValues, "_id=?", new String[]{idStr});
		db.close();
		return result;
	}
}
```

上述代码展示了增删改查的最佳实践例子，查询建议使用rawQuery()，能够自己组织SQL语句，更加灵活，增删改建议使用定安卓制化的API，不容易出错。

上述代码还展示了日期的存取，实际上SQLite中使用integer类型保存日期，在DAO层进行 Java的Date类型转换。这里补充一点SQLite中Integer的定义：

```
INTEGER. The value is a signed integer, stored in 1, 2, 3, 4, 6, or 8 bytes depending on the magnitude of the value.
```

SQLite中的integer会自动根据存储数据的大小进行调节，最大能达到8字节，Java中long类型就是8字节的，所有完全可以用integer类型来存储Java的long类型数据。但是千万要注意，取出数据时（尤其是存储的时间），不要错写成`cursor.getInt()`，否则就会发生溢出，产生奇怪的结果。

## 数据库事务

```java
//开启一个数据库事务
db.beginTransaction();
try
{
    db.execSQL("update account set money= money-200 where name=?",new String[]{"李四"});
    db.execSQL("update account set money= money+200 where name=?",new String[]{"张三"});
    //标记事务中的sql语句全部成功执行
    db.setTransactionSuccessful();
}
finally
{
    //判断事务的标记是否成功，如果不成功，回滚错误之前执行的sql语句
    db.endTransaction();
}
```

2行调用beginTransaction()开启事务，如果执行成功就调用setTransactionSuccessful()标记成功，最终无论成功与否都要调用endTransaction()结束事物，事物内部实现会判断是否回滚。

## Android中SQLite的调试

* 最直接的方式是进入adb shell，切换到`/data/data/<应用包名>/databases/`，执行`sqlite3 xxx.db`。
* 如果数据库较为复杂，shell观察起来比较麻烦，那就只能用ddms或者`adb pull`把数据库文件下载下来，用数据库管理工具查看。
