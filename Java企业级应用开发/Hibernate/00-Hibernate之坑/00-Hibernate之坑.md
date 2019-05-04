# Hibernate之坑

Hibernate坑比较多，这里做一个记录，说的不一定对，以后会陆续更新。这里默认使用的环境是Hibernate3.6，MySQL5.7，Ubuntu16.04，Java8。

## 找不到hbm映射文件

hibernate要求把hbm映射文件放在实体类同一个文件夹下，但是maven和gradle结构中，`src/main/java`目录下的非`.java`文件都会被忽略，因此构建后就找不到hbm文件了。我们可以在`src/main/resources`下建立和包结构
相同的目录，存储这些文件。

现在更推荐的做法是使用JPA注解对实体类进行标注，可读性、可维护性、易用性都更好得多。

## String向TEXT映射

Hibernate自动建表你怎么调，都不会把String映射成TEXT，只会映射varchar(255)，插入超过255个字符的数据就报错。解决办法是手动建表，或自动建完表手动改一下。

实际上，在真实开发流程中，是不可能让Hibernate去自动建表的。

## 显示SQL语句却未插入任何数据 update()未执行

Hibernate里对持久化对象的操作必须配合Transaction使用，否则就会出奇怪的问题，包括数据库操作莫名其妙的未执行等，`session.flush()`都不行。
