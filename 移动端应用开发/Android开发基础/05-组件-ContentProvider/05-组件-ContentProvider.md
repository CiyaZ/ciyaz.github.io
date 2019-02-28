# ContentProvider

Android中，ContentProvider为应用之间共享数据提供统一接口，例如系统通讯录等。通过ContentProvider接口，其他应用可以和使用数据库一样，实现增删改查功能。实际上，开发中一般不用自定义ContentProvider，而是调用系统应用的ContentProvider。

# ContentResolver

从ContentProvider获取数据，使用ContentResolver。这里以读取联系人为例子。

MainActivity.java
```java
public class MainActivity extends Activity
{

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}


	public void readContacts(View view)
	{
		//封装好的联系人URI
		Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;
		//使用游标读取数据
		Cursor cursor = getContentResolver().query(uri, null, null, null, null);
		if (cursor != null)
		{
			while (cursor.moveToNext())
			{
				String displayName = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
				String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));

				Toast.makeText(this, displayName + number, Toast.LENGTH_SHORT).show();
			}

			cursor.close();
		}
	}
}
```

注：需要添加读取联系人权限
