# MyBatis的常用注解

`@Insert`：实现新增 
`@Update`：实现更新 
`@Delete`：实现删除 
`@Select`：实现查询 
`@Result`：实现结果集封装 
`@Results`：可以与`@Result` 一起使用，封装多个结果集 
`@One`：实现一对一结果集封装
`@Many`：实现一对多结果集封装



# MyBatis的增删改查

不使用注解的操作见：[12_MyBatis入门操作](./12_MyBatis入门操作.md)

在增删查改的测试代码部分，可以统一获取 UserMapper 对象

注意这里的`@Before`不是切面的注解，而是 junit 的注解。

`@Before` – 表示在任意使用@Test注解标注的public void方法执行之前执行

```java
public class MyBatisTest {

    private UserMapper mapper;

    @Before
    public void before() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        mapper = sqlSession.getMapper(UserMapper.class);
    }

    @Test
    public void testSave() {
        User user = new User();
        user.setUsername("tom");
        user.setPassword("abc");
        mapper.save(user);
    }

    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(18);
        user.setUsername("lucy");
        user.setPassword("123");
        mapper.update(user);
    }

    @Test
    public void testDelete() {
        mapper.delete(18);
    }

    @Test
    public void testFindById() {
        User user = mapper.findById(2);
        System.out.println(user);
    }

    @Test
    public void testFindAll() {
        List<User> all = mapper.findAll();
        for (User user : all) {
            System.out.println(user);
        }
    }
}
```



修改MyBatis的核心配置文件，因为我们使用了注解替代的映射文件，所以只需要加载使用了注解的Mapper接口即可。

```xml
<mappers> 
	<!--扫描使用注解的类--> 
	<mapper class="com.itheima.mapper.UserMapper"></mapper>
</mappers>
```
或者指定扫描包含映射关系的接口所在的包
```xml
<mappers> 
	<!--扫描使用注解的类所在的包--> 
	<package name="com.itheima.mapper"></package>
</mappers>
```

**不再使用userMapper.xml**



**增删查改的注解实现（配置在userMapper接口中）：**

```java
public interface UserMapper {
	// 插入
    @Insert("insert into user values(#{id},#{username},#{password},#{birthday})")
    public void save(User user);
	// 修改
    @Update("update user set username=#{username},password=#{password} where id=#{id}")
    public void update(User user);
	// 删除
    @Delete("delete from user where id=#{id}")
    public void delete(int id);
	// 查一条
    @Select("select * from user where id=#{id}")
    public User findById(int id);
	// 查列表
    @Select("select * from user")
    public List<User> findAll();
}
```



# MyBatis的注解实现复杂映射开发

实现复杂关系映射之前我们可以在映射文件中通过配置`<resultMap>`来实现，使用注解开发后，可以使用@Results注解 ，@Result注解，@One注解，@Many注解组合完成复杂关系的配置

| 注解                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `@Results`                | 代替的是标签`<resultMap>`，该注解中可以使用单个@Result注解，也可以使用@Result集合。<br/>使用格式：`@Results({@Result()，@Result()})`或`@Results(@Result())` |
| `@Result`                 | 代替了`<id>`标签和`<result>`标签 <br/>@Result中属性介绍： <br/>column：数据库的列名 <br/>property：需要装配的属性名 <br/>one：需要使用的@One注解<br/>many：需要使用的@Many 注解 |
| `@One`（一对一） | 代替了`<assocation> `标签，是多表查询的关键，在注解中用来指定子查询返回单一对象。<br/> @One注解属性介绍： select: 指定用来多表查询的 sqlmapper<br/>使用格式：`@Result(column=" ",property="",one=@One(select=""))` |
| `@Many`（多对一）    | 代替了`<collection>`标签, 是是多表查询的关键，在注解中用来指定子查询返回对象集合。<br/> 使用格式：`@Result(property="",column="",many=@Many(select=""))` |



查询逻辑和 [15_MyBatis的多表操作](./15_MyBatis的多表操作.md) 中的完全相同，不再赘述。

## 一对一查询

查询一个订单，与此同时查询出该订单所属的用户



对应的sql语句（逻辑上不完全对应）：

```sql
select * from orders;
select * from user where id=[查询出订单的uid];
```



使用注解配置OrderMapper接口

```java
@Select("select * from orders")
@Results({
    @Result(column = "id", property = "id"),
    @Result(column = "ordertime", property = "ordertime"),
    @Result(column = "total", property = "total"),
    @Result(
        property = "user", //要封装的属性名称
        column = "uid", //根据哪个字段去查询user表的数据
        javaType = User.class, //要封装的实体类型
        one = @One(select = "com.itheima.mapper.UserMapper.findById") //select属性 代表查询那个接口的方法获得数据
    )
})
public List<Order> findAll();
```



![image-20220329202617324](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292026401.png)



下面是只做一次查询的写法

```java
@Select("select *,o.id oid from orders o,user u where o.uid=u.id")
@Results({
        @Result(column = "oid",property = "id"),
        @Result(column = "ordertime",property = "ordertime"),
        @Result(column = "total",property = "total"),
        @Result(column = "uid",property = "user.id"),
        @Result(column = "username",property = "user.username"),
        @Result(column = "password",property = "user.password")
})
public List<Order> findAll();
```



## 一对多查询

一对多查询的需求：查询一个用户，与此同时查询出该用户具有的订单

对应的sql语句： 

```sql
select * from user;
select * from orders where uid=查询出用户的id;
```

使用注解配置UserMapper接口

```java
@Select("select * from user")
@Results({
    @Result(id = true, column = "id", property = "id"),
    @Result(column = "username", property = "username"),
    @Result(column = "password", property = "password"),
    @Result(
        property = "orderList",
        column = "id",
        javaType = List.class,
        many = @Many(select = "com.itheima.mapper.OrderMapper.findByUid")
    )
})
public List<User> findUserAndOrderAll();
```



![image-20220329204846253](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292049849.png)

## 多对多查询

多对多查询的需求：查询用户同时查询出该用户的所有角色

对应的sql语句： 

```sql
select * from user; 
select * from role r,user_role ur where r.id=ur.role_id and ur.user_id=用户的id
```

使用注解配置RoleMapper接口

```java
@Select("SELECT * FROM sys_user_role ur,sys_role r WHERE ur.roleId=r.id AND ur.userId=#{uid}")
public List<Role> findByUid(int uid);
```

使用注解配置UserMapper接口

```java
@Select("select * from user")
@Results({
    @Result(id = true, column = "id", property = "id"),
    @Result(column = "username", property = "username"),
    @Result(column = "password", property = "password"),
    @Result(
        property = "roleList",
        column = "id",
        javaType = List.class,
        many = @Many(select = "com.itheima.mapper.RoleMapper.findByUid")
    )
})
public List<User> findUserAndRoleAll();
```



![image-20220329210529679](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292105760.png)

