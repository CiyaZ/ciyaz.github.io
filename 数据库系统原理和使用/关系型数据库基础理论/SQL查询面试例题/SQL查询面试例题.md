# SQL查询面试例题

参考：[https://www.cnblogs.com/deng-cc/p/6515166.html](https://www.cnblogs.com/deng-cc/p/6515166.html)

题目来自链接中文章，修正了其中不规范、错误的地方，并按照MySQL的风格重新命令了表和字段，并进行了在MySQL5.7中进行了测试。

这里有三张表：

```
student(student_id,student_name)
teacher(teacher_id,teacher_name)
course(course_id,course_name,teacher_id)
sc(student_id,course_id,score)
```

## 查询所有ID=1的课程比ID=2的课程分数低的同学的学号

```sql
SELECT
	a.student_id
FROM
	sc a
	INNER JOIN sc b ON a.student_id = b.student_id
WHERE
	a.score < b.score
	AND a.course_id = 1
	AND b.course_id =2
```

## 查询平均成绩大于60分的学号和他的平均成绩

```sql
SELECT
	student_id,
	avg( score )
FROM
	sc
GROUP BY
	student_id
HAVING
	avg( score ) > 60
```

## 查询所有同学的学号、姓名、选课数、总成绩

```sql
SELECT
	student.student_id,
	student.student_name,
	count( sc.course_id ),
	sum( sc.score )
FROM
	student
	LEFT JOIN sc ON student.student_id = sc.student_id
GROUP BY
	sc.student_id
```

## 查询名字`J`开头的教师的个数

```sql
SELECT
	count( teacher_id )
FROM
	teacher
WHERE
	teacher_name LIKE 'J%'
```

注意：LIKE后面用的单引号。

## 查询没学过`Jack`老师课的学生的学号和姓名

```sql
SELECT
	student.student_id,
	student.student_name
FROM
	student
WHERE
	student.student_id NOT IN (
	SELECT
		student.student_id
	FROM
		student
		LEFT JOIN sc ON student.student_id = sc.student_id
		INNER JOIN course ON sc.course_id = course.course_id
		INNER JOIN teacher ON course.teacher_id = teacher.teacher_id
	WHERE
	teacher.teacher_name = 'Jack'
	)
```

## 查询既学过ID=1课程也学过ID=2课程的同学学号和姓名

```sql
SELECT
	a.student_id,
	a.student_name
FROM
	(
	SELECT
		student.student_id,
		student.student_name
	FROM
		student
		INNER JOIN sc ON student.student_id = sc.student_id
	WHERE
		sc.course_id = 1 UNION ALL
	SELECT
		student.student_id,
		student.student_name
	FROM
		student
		INNER JOIN sc ON student.student_id = sc.student_id
	WHERE
		sc.course_id = 2
	) a
GROUP BY
	a.student_id
HAVING
	count( a.student_id ) =2
```

注明：这道题展示了如何求两个查询结果的交集，我们知道并集使用`UNION`和`UNION ALL`连接两个查询结果，但是MySQL中并没有求两个查询结果的交集和差集的操作，实现思路就稍微绕弯了：

1. 求交集，使用`UNION ALL`得到不去重并集，然后对并集中一个ID出现的次数进行判断，出现次数等于并集操作集合数，这个元素就在所求的交集中
2. 求两个集合差集，原理同上

## 查询学过`Jack`老师所教的所有课的学生的学号和姓名

```sql
SELECT
	student.student_id,
	student.student_name
FROM
	student
	INNER JOIN sc ON student.student_id = sc.student_id
	INNER JOIN course ON sc.course_id = course.course_id
	INNER JOIN teacher ON course.teacher_id = teacher.teacher_id
WHERE
	teacher.teacher_name = 'Jack'
GROUP BY
	student.student_id
HAVING
	count( student.student_id ) = (
	SELECT
		count( * )
	FROM
		course
		INNER JOIN teacher ON course.teacher_id = teacher.teacher_id
	WHERE
		teacher.teacher_name = 'Jack'
	GROUP BY
	teacher.teacher_id
	)
```

## 查询有课程成绩小于60分的同学的学号姓名

```sql
SELECT
	student.student_id,
	student.student_name
FROM
	student
	INNER JOIN sc ON student.student_id = sc.student_id
WHERE
	sc.score < 60
```

## 查询没有学全所有课的学生的学号姓名

```sql
SELECT
	student.student_id,
	student.student_name
FROM
	student
	LEFT JOIN sc ON student.student_id = sc.student_id
GROUP BY
	student.student_id
HAVING
	count( student.student_id ) < ( SELECT count( * ) FROM course )
```

## 查询至少有一门课与学号为1的同学所学的课程相同的学生和姓名

```sql
SELECT DISTINCT
	student.student_id,
	student.student_name
FROM
	student
	INNER JOIN sc ON student.student_id = sc.student_id
WHERE
	sc.course_id IN ( SELECT sc.course_id FROM sc WHERE sc.student_id = 1 )
```

## 查询和ID=1的同学所学课程完全相同的同学的学号和姓名

```sql
SELECT DISTINCT
	student.student_id,
	student.student_name
FROM
	student
	INNER JOIN sc ON student.student_id = sc.student_id
GROUP BY
	sc.student_id
HAVING
	count( sc.student_id ) = ( SELECT count( * ) FROM sc WHERE student_id = 1 )
```
