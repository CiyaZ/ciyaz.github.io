# 连接MySQL数据库

PHP可以通过MySQLi或PDO连接MySQL数据库，区别是PDO支持多种数据库，而MySQLi只支持一种数据库。实际上，MySQLi用的非常多，这是因为PHP开发的中小型网站大部分用的都是MySQL数据库服务器。这里为了方便起见，我们使用MySQLi。

注：PHP安装好后一般都内置了MySQLi扩展，但可能没有开启（Windows下），我们手动改下`php.ini`即可。

## 连接数据库

下面例子连接数据库，输出「连接数据库成功」，然后关闭连接。

```php
$server_name = "localhost";
$username = "root";
$password = "root";
$db_name = "learn_php";
$conn = new mysqli($server_name, $username, $password, $db_name);
if ($conn->connect_error) {
	die("连接数据库失败".$conn->connect_error);
}
echo "连接数据库成功";
$conn->close();
```

## 实现数据增删改查

我们这里设定如下数据库表结构：

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  PRIMARY KEY (`user_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

下面我们使用PHP对数据表进行增删改查，注意这里我们使用的都是`prepared statement`，这样能够提高SQL执行效率，也能一定程度上防止SQL注入，数据库连接开启和关闭的代码沿用上面。

### 插入数据

```php
$stmt = $conn->prepare("insert into t_user (user_id, username, password) values (?,?,?)");

$user_id = 1;
$username = "root";
$password = "123456";
$stmt->bind_param("iss", $user_id, $username, $password);
$stmt->execute();
```

删除，更新数据同理，拼接不同SQL即可实现，这里就不多做演示了。

### 查询数据

```php
$stmt = $conn->prepare("select * from t_user");
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows > 0) {
	while ($data = $result->fetch_assoc()) {
		echo strval($data["user_id"]) . $data["username"] . $data["password"] . "<br />";
	}
}
```

注：不知道为什么，网上好多文章连这个最简单的带预编译的查询代码都是错的，包括StackOverflow。

## 连接池

不用找了，PHP请求一次执行一次的模型使得这个语言没法实现连接池，当然，既然PHP是基于C的，我们魔改一番也能让PHP用上连接池，目前已经有了基于扩展的解决方案，这里就不多做介绍了。
