# 1. 动态SQL语句

动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。



## if

使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分。比如：

```xml-dtd
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

这条语句提供了可选的查找文本功能。如果不传入 “title”，那么所有处于 “ACTIVE” 状态的 BLOG 都会返回；如果传入了 “title” 参数，那么就会对 “title” 一列进行模糊查找并返回对应的 BLOG 结果（细心的读者可能会发现，“title” 的参数值需要包含查找掩码或通配符字符）。

如果希望通过 “title” 和 “author” 两个参数进行可选搜索该怎么办呢？

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

## foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（**尤其是在构建 IN 条件语句的时候**）。

> 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 *foreach*。
>
> 当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

循环执行sql的拼接操作，例如：`SELECT * FROM USER WHERE id IN (1,2,5)`

```xml
<select id="findByIds" parameterType="list" resultType="user"> 
    select * from User 
    <where> 
        <foreach collection="array" open="id in(" close=")" item="id" separator=","> 
            #{id}
        </foreach> 
    </where>
</select>
```

测试代码：

```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

//模拟ids的数据
List<Integer> ids = new ArrayList<Integer>();
ids.add(1);
ids.add(2);

List<User> userList = mapper.findByIds(ids);
System.out.println(userList);
```

out:

```
18:51:26,142 DEBUG JdbcTransaction:137 - Opening JDBC Connection
18:51:26,303 DEBUG PooledDataSource:406 - Created connection 959629210.
18:51:26,303 DEBUG JdbcTransaction:101 - Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@3932c79a]
18:51:26,305 DEBUG findByIds:159 - ==>  Preparing: select * from user WHERE id in( ? , ? ) 
18:51:26,333 DEBUG findByIds:159 - ==> Parameters: 1(Integer), 2(Integer)
18:51:26,364 DEBUG findByIds:159 - <==      Total: 2
[User{id=1, username='tom', password='123456'}, User{id=2, username='jerry', password='123456'}]
```



foreach标签的属性含义如下： 
`<foreach>`标签用于遍历集合，它的属性： 

- collection：代表要遍历的集合元素，注意编写时不要写`#{}`
- open：代表语句的开始部分 
- close：代表结束部分
- item：代表遍历集合的每个元素，生成的变量名
- sperator：代表分隔符



# 2. SQL语句抽取

可将重复的 sql 提取出来，使用时用 `<include>` 引用即可，最终达到 sql 重用的目的

`<sql>`标签：sql片段抽取

将上面的语句改成：

```xml
<!--sql语句抽取-->
<sql id="selectUser">select * from user</sql>

<select id="findByCondition" parameterType="user" resultType="user">
    <include refid="selectUser"></include>
    <where>
        <if test="id!=0">
            and id=#{id}
        </if>
        <if test="username!=null">
            and username=#{username}
        </if>
        <if test="password!=null">
            and password=#{password}
        </if>
    </where>
</select>

<select id="findByIds" parameterType="list" resultType="user">
    <include refid="selectUser"></include>
    <where>
        <foreach collection="list" open="id in(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```



