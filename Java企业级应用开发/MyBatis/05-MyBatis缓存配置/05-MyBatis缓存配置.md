# MyBatis缓存配置

## 一级缓存

MyBatis的一级缓存和Hibernate一样，是SqlSession级别的缓存，是默认启用的，我们开发者一般也不需要更改它的设置，也感受不到它的存在，只要注意避免因缓存造成的bug即可。

注：可能发生的bug是指查询得到的结果因缓存未得到及时更新，这个情况很难碰到。

刷新一级缓存的例子：

```
<select id="selectByid" flushCache="true" resultMap="userMap" >
  select * from sys user where id = #{id}
</select>
```

这样配置`<select>`标签后，每次查询都会刷新缓存，当然，这样做会对性能产生影响。

## 二级缓存

MyBatis的二级缓存是和命名空间（一个Mapper）绑定的，一个Mapper对应一组缓存。MyBatis具有内置的二级缓存，配置方法如下代码：

```xml
<mapper namespace="">
  <cache/>
</mapper>
```

`<cache>`标签有入下几个可配置选项：

* `eviction`：缓存策略，如`LRU(默认)`，`FIFO`等，都很好理解
* `flushInterval`：缓存刷新时间，默认不定时刷新，只在调用语句时刷新
* `size`：缓存的引用数，默认1024个
* `readOnly`：是否为只读缓存，默认为`false`，只读缓存使用Map会给调用者返回相同的实例，因此这些对象不能修改否则就容易出bug，可读写缓存会通过序列化返回缓存对象的拷贝，但也因此性能不如只读缓存
* `type`：缓存的提供类全名，比如使用EhCache这个值就是`org.mybatis.caches.ehcache.EhcacheCache`

注意：使用可读写缓存，实体类必须实现`Serializable`接口

## 配置EhCache

MyBatis集成EhCache需要依赖：
```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

其余配置和Hibernate相同（参考Java/Hibernate/05-配置缓存和连接池）
