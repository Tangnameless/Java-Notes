# 类型处理器（typeHandlers）

> 官方文档：https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers

MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

**默认的类型处理器无需设置！**

| 类型处理器              | Java 类型                      | JDBC 类型                            |
| :---------------------- | :----------------------------- | :----------------------------------- |
| `BooleanTypeHandler`    | `java.lang.Boolean`, `boolean` | 数据库兼容的 `BOOLEAN`               |
| `ByteTypeHandler`       | `java.lang.Byte`, `byte`       | 数据库兼容的 `NUMERIC` 或 `BYTE`     |
| `ShortTypeHandler`      | `java.lang.Short`, `short`     | 数据库兼容的 `NUMERIC` 或 `SMALLINT` |
| `IntegerTypeHandler`    | `java.lang.Integer`, `int`     | 数据库兼容的 `NUMERIC` 或 `INTEGER`  |
| `LongTypeHandler`       | `java.lang.Long`, `long`       | 数据库兼容的 `NUMERIC` 或 `BIGINT`   |
| `FloatTypeHandler`      | `java.lang.Float`, `float`     | 数据库兼容的 `NUMERIC` 或 `FLOAT`    |
| `DoubleTypeHandler`     | `java.lang.Double`, `double`   | 数据库兼容的 `NUMERIC` 或 `DOUBLE`   |
| `BigDecimalTypeHandler` | `java.math.BigDecimal`         | 数据库兼容的 `NUMERIC` 或 `DECIMAL`  |
| `StringTypeHandler`     | `java.lang.String`             | `CHAR`, `VARCHAR`                    |
|`DateTypeHandler`	| `java.util.Date`	| `TIMESTAMP`|

==**提示** 从 3.4.5 开始，MyBatis 默认支持 JSR-310（日期和时间 API） 。==

你可以重写已有的类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 

具体做法为：实现`org.apache.ibatis.type.TypeHandler` 接口， 或继承一个很便利的类 `org.apache.ibatis.type.BaseTypeHandler`， 并且可以（可选地）将它映射到一个 JDBC 类型。

**需求举例：**

一个Java中的Date数据类型，我想将之存到数据库的时候存成一个1970年至今的毫秒数，取出来时转换成java的Date，即java的Date与数据库的毫秒值（bigint）之间转换。

**开发步骤：** 

1、定义转换类继承类`BaseTypeHandler<T>`
2、覆盖4个未实现的方法，其中`setNonNullParameter`为java程序设置数据到数据库的回调方法，`getNullableResult` 为查询时mysql的字符串类型转换成 java的Type类型的方法
3、在MyBatis核心配置文件中进行注册
4、测试转换是否正确



```java
public class DateTypeHandler extends BaseTypeHandler<Date> {
    // 将java类型转换成数据库需要的类型
    // int i表示表中第几个字段
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Date date, JdbcType jdbcType) throws SQLException {
        long time = date.getTime();
        preparedStatement.setLong(i, time);
    }

    // 将数据库中类型转换成java类型
    // String参数  要转换的字段名称
    // ResultSet 查询出的结果集
    public Date getNullableResult(ResultSet resultSet, String s) throws SQLException {
        // 获得结果集中需要的数据(long) 转换成Date类型 返回
        long aLong = resultSet.getLong(s);
        Date date = new Date(aLong);
        return date;
    }

    // 将数据库中类型转换成java类型
    public Date getNullableResult(ResultSet resultSet, int i) throws SQLException {
        long aLong = resultSet.getLong(i);
        Date date = new Date(aLong);
        return date;
    }

    // 将数据库中类型转换成java类型
    public Date getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        long aLong = callableStatement.getLong(i);
        Date date = new Date(aLong);
        return date;
    }
}
```

> `public abstract long getLong(String columnLabel)`方法
>
> 检索此ResultSet对象的当前行中指定列的值，作为Java编程语言中的long返回



在`sqlMapConfig.xml`中注册自定义的类型处理器

```xml-dtd
<!--注册类型处理器-->
<typeHandlers>
	<typeHandler handler="com.itheima.handler.DateTypeHandler"></typeHandler>
</typeHandlers>
```

测试代码：

```java
@Test
public void test1() throws IOException {
	... // 省略获取mapper
    //创建user
    User user = new User();
    user.setUsername("ceshi1");
    user.setPassword("abcdf");
    user.setBirthday(new Date());
    //执行保存操作
    mapper.save(user);

    sqlSession.commit();
    sqlSession.close();
}
```

数据库中的数据

![image-20220328224924958](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203282249989.png)

其中birthday字段以bigint方式存储

对刚插入的数据进行查询：

```
user中的birthday：Mon Mar 28 22:07:51 HKT 2022
```

返回的数据又被封装为Date类型

疑问：在表的设计中，birthday字段采用varchar和bigint都顺利完成了转换，为什么呢？

# plugins标签（使用分页助手PageHelper）

MyBatis可以使用第三方的插件来对功能进行扩展，分页助手**PageHelper**是将分页的复杂操作进行封装，使用简单的方式即 可获得分页的相关数据 

开发步骤：
1、导入通用PageHelper的坐标
2、在mybatis核心配置文件中配置PageHelper插件
3、测试分页数据获取



**1、导入通用PageHelper的坐标**

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>3.7.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.github.jsqlparser</groupId>
    <artifactId>jsqlparser</artifactId>
    <version>0.9.1</version>
</dependency>
```



**2、在mybatis核心配置文件中配置PageHelper插件**

```xml
<!--配置分页助手插件-->
<!-- 注意：分页助手的插件 配置在通用mapper之前 -->
<plugins>
    <plugin interceptor="com.github.pagehelper.PageHelper">
        <!-- 指定方言 -->
        <property name="dialect" value="mysql"></property>
    </plugin>
</plugins>
```



**3、测试分页代码实现**

```java
@Test
public void test3() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    //设置分页相关参数   当前页+每页显示的条数
    PageHelper.startPage(3,2); // 每页显示两条数据，获取第3页的数据

    List<User> userList = mapper.findAll();
    for (User user : userList) {
        System.out.println(user);
    }

    //获得与分页相关参数
    PageInfo<User> pageInfo = new PageInfo<User>(userList);
    System.out.println("当前页："+pageInfo.getPageNum());
    System.out.println("每页显示条数："+pageInfo.getPageSize());
    System.out.println("总条数："+pageInfo.getTotal());
    System.out.println("总页数："+pageInfo.getPages());
    System.out.println("上一页："+pageInfo.getPrePage());
    System.out.println("下一页："+pageInfo.getNextPage());
    System.out.println("是否是第一个："+pageInfo.isIsFirstPage());
    System.out.println("是否是最后一个："+pageInfo.isIsLastPage());

    sqlSession.close();
}
```

