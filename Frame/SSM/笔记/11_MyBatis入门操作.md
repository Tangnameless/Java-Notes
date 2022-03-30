# 1. MyBatis的简介 

##  原始jdbc操作的分析

**原始jdbc开发存在的问题如下：**

1、数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能 
2、sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变java代码。 
3、查询操作时，需要手动将结果集中的数据手动封装到实体中。插入操作时，需要手动将实体的数据设置到 sql 语句的占位符位置

**应对上述问题给出的解决方案： **

1、使用数据库连接池初始化连接资源
2、将sql语句抽取到xml配置文件中
3、使用反射、内省等底层技术，自动将实体与表进行属性与字段的自动映射

## 什么是MyBatis？

- MyBatis是一个优秀的基于java的持久层框架，它内部封装了 jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。
- MyBatis通过xml或注解的方式将要执行的各种 statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句。
- 最后MyBatis框架执行sql并将结果映射为java对象并返回。采用ORM思想解决了实体和数据库映射的问题，对jdbc 进行了 封装，屏蔽了jdbc api 底层访问细节，使我们不用与jdbc api打交道，就可以完成对数据库的持久化操作。

> MyBatis参考文档：https://mybatis.org/mybatis-3/zh/index.html

# 2. MyBatis的快速入门 

**MyBatis开发步骤：** 

1）添加MyBatis的坐标 
2）创建user数据表 
3）编写User实体类 
4）编写映射文件UserMapper.xml 
5）编写核心文件SqlMapConfig.xml
6）编写测试类

## 环境搭建

**1、导入MyBatis的坐标和其他相关坐标**

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

**2、创建user数据表**

**3、创建User实体类**

```java
public class User {

    private int id;
    private String username;
    private String password;
    // 省略set,get方法
}
```

**4、编写UserMapper映射文件**

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">
    <!--查询操作-->
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>
```



**5、编写MyBatis核心文件**

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--通过properties标签加载外部properties文件-->
    <properties resource="jdbc.properties"></properties>

    <!--数据源环境-->
    <environments default="developement">
        <environment id="developement">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    
    <!--加载映射文件-->
    <mappers>
        <mapper resource="com/itheima/mapper/UserMapper.xml"></mapper>
    </mappers>

</configuration>
```



## 编写测试代码

以测试查询list为例：

```java
// 加载核心配置文件(路径相对于resources目录)
InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
// 获得sqlSession工厂对象
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
// 获得sqlSession对象
SqlSession sqlSession = sqlSessionFactory.openSession();
// 执行sql语句 参数：namespace+id
List<User> userList = sqlSession.selectList("userMapper.findAll");
// 打印结果
System.out.println(userList);
// 释放资源
sqlSession.close();
```

# 3. MyBatis的映射文件概述 

![image-20220328153847489](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203281538556.png)

# 4. MyBatis的增删改查操作 

## 插入操作

**编写UserMapper.xml**

```xml-dtd
<!--插入操作-->
<insert id="save" parameterType="com.itheima.domain.User">
	insert into user values(#{id},#{username},#{password})
</insert>
```



> 注意区分`${}`和`#{}`
>
> `${}` 是 Properties ⽂件中的变量占位符，它可以⽤于标签属性值和 sql 内部，属于静态⽂本替换，⽐如`${driver}`会被静态替换为 `com.mysql.jdbc.Driver` 。 
>
> `#{}` 是 sql 的参数占位符，Mybatis 会将 sql 中的 `#{}` 替换为`?`号，在 sql 执⾏前会使⽤PreparedStatement 的参数设置⽅法，按序给 sql 的`?`号占位符设置参数值，⽐如 `ps.setInt(0, parameterValue)`， `#{item.name}` 的取值⽅式为使⽤反射从参数对象中获取，item 对象的 name 属性值，相当于 `param.getItem().getName()` 。

**编写插入实体User的代码**

```java
@Test
//插入操作
public void test2() throws IOException {

    //模拟user对象
    User user = new User();
    user.setUsername("xxxx");
    user.setPassword("abc");

    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    sqlSession.insert("userMapper.save", user);
    //mybatis执行更新操作  提交事务
    sqlSession.commit();
    sqlSession.close();
}
```



**插入操作注意问题** 

- 插入语句使用insert标签 

- 在映射文件中使用`parameterType`属性指定要插入的数据类型 

- Sql语句中使用`#{实体属性名}`方式引用实体中的属性值 

- 插入操作使用的API是`sqlSession.insert(“命名空间.id”,实体对象); `

- 插入操作涉及数据库数据变化，所以要使用sqlSession对象显示的提交事务，即`sqlSession.commit()`

## 删除操作

**编写UserMapper.xml**

```xml-dtd
<!--删除操作-->
<delete id="delete" parameterType="int">
	delete from user where id=#{id}
</delete>
```

**编写删除数据的代码**

```java
@Test
//删除操作
public void test4() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    sqlSession.delete("userMapper.delete", 8);
    sqlSession.commit();
    sqlSession.close();
}
```

**删除操作注意问题** 

- 删除语句使用delete标签 
- Sql语句中使用#{任意字符串}方式引用传递的单个参数
- 删除操作使用的API是`sqlSession.delete(“命名空间.id”,Object);`

## 查询操作

**编写UserMapper.xml**

```xml-dtd
<!--查询操作-->
<select id="findAll" resultType="user">
	select * from user
</select>

<!--根据id进行查询-->
<select id="findById" resultType="user" parameterType="int">
	select * from user where id=#{id}
</select>
```

单个查询和列表查询

```java
@Test
//查询一个对象
public void test5() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    User user = sqlSession.selectOne("userMapper.findById", 1);
    System.out.println(user);
    sqlSession.close();
}

@Test
//查询列表
public void test1() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    List<User> userList = sqlSession.selectList("userMapper.findAll");
    System.out.println(userList);
    sqlSession.close();
}
```

## 修改操作

```xml-dtd
<!--修改操作-->
<update id="update" parameterType="com.itheima.domain.User">
	update user set username=#{username},password=#{password} where id=#{id}
</update>
```

```java
@Test
//修改操作
public void test3() throws IOException {

    //模拟user对象
    User user = new User();
    user.setId(7);
    user.setUsername("lucy");
    user.setPassword("123");

    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace+id
    sqlSession.update("userMapper.update", user);
    sqlSession.commit();
    sqlSession.close();
}
```

**修改操作注意问题** 

- 修改语句使用update标签
- 修改操作使用的API是`sqlSession.update(“命名空间.id”,实体对象);`

# 5. MyBatis的核心配置文件概述

## MyBatis核心配置文件层级关系

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

- configuration（配置）
  - [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties)
  - [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings)
  - [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)
  - [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
  - [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
  - [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
  - [environments（环境配置）](https://mybatis.org/mybatis-3/zh/configuration.html#environments)
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)
  - [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers)

## MyBatis常用配置解析

### 1) environments标签

![image-20220328161136702](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203281611764.png)

**事务管理器（transactionManager）**

在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

- JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。

- MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。例如:

  ```xml
  <transactionManager type="MANAGED">
    <property name="closeConnection" value="false"/>
  </transactionManager>
  ```



数据源（dataSource）类型有三种： 

- UNPOOLED：这个数据源的实现只是每次被请求时打开和关闭连接。
- POOLED：这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来。 
- JNDI：这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。



### 2）mapper标签 

该标签的作用是加载映射的，需要告诉 MyBatis 到哪里去找到这些sql语句。

加载方式有如下几种： 

- **使用相对于类路径的资源引用，例如：`<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>`**
- 使用完全限定资源定位符（URL），例如：`<mapper url="file:///var/mappers/AuthorMapper.xml"/>`
- 使用映射器接口实现类的完全限定类名，例如：`<mapper class="org.mybatis.builder.AuthorMapper"/>`
- 将包内的映射器接口实现全部注册为映射器，例如：`<package name="org.mybatis.builder"/>`



### 3）properties标签

![image-20220328161922306](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203281619376.png)



### 4）typeAliases标签

![image-20220328162349123](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203281623188.png)

下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

# 6. MyBatis的相应API

## SqlSession工厂构建器SqlSessionFactoryBuilder

常用API：`SqlSessionFactory build(InputStream inputStream) `

通过加载mybatis的核心文件的输入流的形式构建一个SqlSessionFactory对象

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

其中， Resources 工具类，这个类在 `org.apache.ibatis.io` 包中。Resources 类帮助你从类路径下、文件系统或 一个 web URL 中加载资源文件。



## SqlSession工厂对象SqlSessionFactory



```java
SqlSession openSession()
SqlSession openSession(boolean autoCommit) // 参数为是否自动提交，如果设置为true，那么不需要手动提交事务
```

默认的 openSession() 方法没有参数，它会创建具备如下特性的 SqlSession：

- 事务作用域将会开启（也就是不自动提交）。
- 将由当前环境配置的 DataSource 实例中获取 Connection 对象。
- 事务隔离级别将会使用驱动或数据源的默认设置。
- 预处理语句不会被复用，也不会批量处理更新。



## SqlSession会话对象

SqlSession 在 MyBatis 中是非常强大的一个类。它包含了所有执行语句、提交或回滚事务以及获取映射器实例的方法。

### 语句执行方法

这些方法被用来执行定义在 SQL 映射 XML 文件中的 SELECT、INSERT、UPDATE 和 DELETE 语句。你可以通过名字快速了解它们的作用，每一方法都接受语句的 ID 以及参数对象，参数可以是原始类型（支持自动装箱或包装类）、JavaBean、POJO 或 Map。

```java
<T> T selectOne(String statement, Object parameter)
<E> List<E> selectList(String statement, Object parameter)
<T> Cursor<T> selectCursor(String statement, Object parameter)
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)
int insert(String statement, Object parameter)
int update(String statement, Object parameter)
int delete(String statement, Object parameter)
```

selectOne 和 selectList 的不同仅仅是 selectOne 必须返回一个对象或 null 值。如果返回值多于一个，就会抛出异常。如果你不知道返回对象会有多少，请使用 selectList。如果需要查看某个对象是否存在，最好的办法是查询一个 count 值（0 或 1）。selectMap 稍微特殊一点，它会将返回对象的其中一个属性作为 key 值，将对象作为 value 值，从而将多个结果集转为 Map 类型值。由于并不是所有语句都需要参数，所以这些方法都具有一个不需要参数的重载形式。

### 事务控制方法

有四个方法用来控制事务作用域。当然，如果你已经设置了自动提交或你使用了外部事务管理器，这些方法就没什么作用了。然而，如果你正在使用由 Connection 实例控制的 JDBC 事务管理器，那么这四个方法就会派上用场：

```java
void commit()
void commit(boolean force)
void rollback()
void rollback(boolean force)
```

默认情况下 MyBatis 不会自动提交事务，除非它侦测到调用了插入、更新或删除方法改变了数据库。如果你没有使用这些方法提交修改，那么你可以在 commit 和 rollback 方法参数中传入 true 值，来保证事务被正常提交（注意，在自动提交模式或者使用了外部事务管理器的情况下，设置 force 值对 session 无效）。大部分情况下你无需调用 rollback()，因为 MyBatis 会在你没有调用 commit 时替你完成回滚操作。不过，当你要在一个可能多次提交或回滚的 session 中详细控制事务，回滚操作就派上用场了。

### 确保 SqlSession 被关闭

```java
void close()
```

对于你打开的任何 session，你都要保证它们被妥善关闭，这很重要。保证妥善关闭的最佳代码模式是这样的：

```java
SqlSession session = sqlSessionFactory.openSession();
try (SqlSession session = sqlSessionFactory.openSession()) {
    // 假设下面三行代码是你的业务逻辑
    session.insert(...);
    session.update(...);
    session.delete(...);
    session.commit();
}
```

