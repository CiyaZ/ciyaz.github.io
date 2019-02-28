# HQL查询

SQL中，查询语句的编写是很重要的部分。Hibernate中，可以使用面向对象的HQL语句，SQL操作的是数据表，列等，HQL操作的就是对象，属性等，HQL的语法和SQL很像，HQL关键字不区分大小写，但是类名，属性名等是区分大小写的。

## 使用HQL进行查询的例子

这里使用的例子是上一节中的老师-同学双向N-N。

Student.java
```java
@Entity
@Table(name = "t_student")
public class Student
{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "student_id")
	private Long studentId;
	@Column(name = "name")
	private String studentName;

	@ManyToMany(mappedBy = "studentSet")
	private Set<Teacher> teacherSet = new HashSet<>();

  //省略构造函数，get/set方法略
}
```

Teacher.java
```java
@Entity
@Table(name = "t_teacher")
public class Teacher
{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "teacher_id")
	private Long teacherId;
	@Column(name = "name")
	private String teacherName;

	@ManyToMany
	@JoinTable(name = "teacher_student",
			joinColumns = {@JoinColumn(name = "teacher_id")},
			inverseJoinColumns = {@JoinColumn(name = "student_id")})
	private Set<Student> studentSet = new HashSet<>();

  //省略构造函数，get/set方法略
}
```

### 查询名为t1的老师

```java
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();

List<Teacher> teacherList = session.createQuery("from Teacher t where t.teacherName = :teacherName")
    .setString("teacherName", "t1")
    .list();

for(Teacher t : teacherList)
{
  System.out.println(t);
}

transaction.commit();
session.close();
```

我们发现，HQL里连SELECT都不用，实际上按照SQL的习惯写成`select t from ...`也是可以的。查询指定的是对象，而不是某张数据表。

除此之外，注意HQL中占位符的写法，是冒号加上占位符名。

### 连接查询t1老师的所有学生

```java
List<Student> studentList = session.createQuery("select s from Student s inner join s.teacherSet t where t.teacherName = :teacherName")
    .setString("teacherName", "t1")
    .list();
```

### 连接查询t1老师的所有学生的一些属性

```java
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();

List<Object[]> studentAttrList = session.createQuery("select s.studentId, s.studentName from Student s inner join s.teacherSet t where t.teacherName = :teacherName")
    .setString("teacherName", "t1")
    .list();

for(Object[] arr : studentAttrList)
{
  System.out.println(Arrays.toString(arr));
}

transaction.commit();
session.close();
```

HQL和SQL一样，深入研究是有点复杂的，这里就不多举例了。此外，我们还要知道，Hibernate也是支持使用SQL语句的，如果一些奇特的功能HQL无法完成，完全可以使用SQL进行代替。更多有关HQL的内容，可以参考文档。
