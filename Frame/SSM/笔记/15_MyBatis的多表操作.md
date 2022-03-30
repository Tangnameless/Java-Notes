# 1. 一对一查询

**一对一查询的模型**

用户表和订单表的关系为，一个用户有多个订单，**一个订单只从属于一个用户** 

一对一查询的需求：查询一个订单，与此同时查询出该订单所属的用户

![image-20220328230414547](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203282304584.png)



**一对一查询的sql语句**

sql查询语句（inner join）：

```sql
select * from orders o,user u where o.uid=u.id;
```

查询结果：

![image-20220329101203744](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203291012772.png)

**创建Order和User实体**

在创建Order和User实体时，不使用主键/外键，而使用【面向对象】的思想。

```java
public class Order {

    private int id;
    private Date ordertime;
    private double total;

    //当前订单属于哪一个用户
    private User user;
    
    // 省略set,get方法
}
```

```java
public class User {

    private int id;
    private String username;
    private String password;
    private Date birthday;

    //描述的是当前用户存在哪些订单
    private List<Order> orderList;
    // 省略set,get方法
}
```

创建OrderMapper接口

```java
public interface OrderMapper {

    //查询所有的订单（包含订单的用户信息）
    public List<Order> findAll();

}
```

配置OrderMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.OrderMapper">

    <resultMap id="orderMap" type="order">
        <!--手动指定字段与实体属性的映射关系
            column: 数据表的字段名称
            property：实体的属性名称
        -->
        <id column="oid" property="id"></id>
        <result column="ordertime" property="ordertime"></result>
        <result column="total" property="total"></result>
        <!--
            property: 当前实体(order)中的属性名称(private User user)
            javaType: 当前实体(order)中的属性的类型(User)
        -->
        <association property="user" javaType="user">
            <id column="uid" property="id"></id>
            <result column="username" property="username"></result>
            <result column="password" property="password"></result>
            <result column="birthday" property="birthday"></result>
        </association>

    </resultMap>

    <select id="findAll" resultMap="orderMap">
         SELECT *,o.id oid FROM orders o,USER u WHERE o.uid=u.id
    </select>

</mapper>
```

测试代码：

```java
public void test1() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    OrderMapper mapper = sqlSession.getMapper(OrderMapper.class);
    List<Order> orderList = mapper.findAll();
    for (Order order : orderList) {
        System.out.println(order);
    }
    sqlSession.close();
}
```

out：

```
Order{id=1, ordertime=Mon Mar 28 00:00:00 HKT 2022, total=3000.0, user=User{id=1, username='tom', password='123456', birthday=null, roleList=null}}
Order{id=2, ordertime=Mon Mar 28 00:00:00 HKT 2022, total=5000.0, user=User{id=1, username='tom', password='123456', birthday=null, roleList=null}}
Order{id=3, ordertime=Sat Jan 01 00:00:00 HKT 2022, total=10000.0, user=User{id=2, username='jerry', password='123456', birthday=null, roleList=null}}
Order{id=4, ordertime=Wed Feb 02 00:00:00 HKT 2022, total=100.0, user=User{id=3, username='lucy', password='123', birthday=null, roleList=null}}
```

# 2. 一对多查询

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户 

一对多查询的需求：查询一个用户，与此同时查询出该用户具有的订单

![image-20220329101310578](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203291013619.png)

**一对一查询的sql语句**

sql查询语句（left join）：

> LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。

```sql
select *,o.id oid from user u left join orders o on u.id=o.uid;
```

查询结果：

![image-20220329102237944](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203291022975.png)

实体类沿用一对一查询中的User，Order

**创建UserMapper接口：**

```java
public interface UserMapper {
    
    public List<User> findAll();
    
}
```

**配置UserMapper.xml：**

```xml
<resultMap id="userMap" type="user">
	<id column="id" property="id"></id>
	<result column="username" property="username"></result>
	<result column="password" property="password"></result>
	<result column="birthday" property="birthday"></result>
	<!--配置集合信息
		property:集合名称
		ofType：当前集合中的数据类型
	-->
	<collection property="orderList" ofType="order">
		<!--封装order的数据-->
		<id column="oid" property="id"></id>
		<result column="ordertime" property="ordertime"></result>
		<result column="total" property="total"></result>
	</collection>
</resultMap>

<select id="findAll" resultMap="userMap">
	SELECT *,o.id oid FROM USER u,orders o WHERE u.id=o.uid
</select>
```

注：已在sqlMaoConfig.xml中设置了实体类的别名，user的id不应该使用列uid，左连接会出现null

查询代码：

```java
List<User> userList = mapper.findAll();

for (User user : userList) {
    List<Order> orderList = user.getOrderList();
    if (orderList.isEmpty()) {
        continue;
    }
    System.out.println(user.getUsername());
    for (Order order : orderList) {
        System.out.println(order);
    }
    System.out.println("----------------------------------");
}
```

out：

```
tom
Order{id=1, ordertime=Mon Mar 28 00:00:00 HKT 2022, total=3000.0, user=null}
Order{id=2, ordertime=Mon Mar 28 00:00:00 HKT 2022, total=5000.0, user=null}
----------------------------------
jerry
Order{id=3, ordertime=Sat Jan 01 00:00:00 HKT 2022, total=10000.0, user=null}
----------------------------------
lucy
Order{id=4, ordertime=Wed Feb 02 00:00:00 HKT 2022, total=100.0, user=null}
```

# 3. 多对多查询

**多对多查询的模型** 

用户表和角色表的关系为，一个用户有多个角色，一个角色被多个用户使用
多对多查询的需求：查询用户同时查询出该用户的所有角色

![image-20220329105517612](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203291055661.png)

多对多查询的语句对应的sql语句：
```sql
SELECT *
FROM USER u, sys_user_role ur, sys_role r
WHERE u.id = ur.userId
	AND ur.roleId = r.id;
```
查询的结果如下：

![image-20220329110120904](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203291101934.png)



**创建Role实体，修改User实体**

```java
public class User {
	// ...
    //描述的是当前用户具备哪些角色
    private List<Role> roleList;
}
```

```java
public class Role {

    private int id;
    private String roleName;
    private String roleDesc;
}
```

**添加UserMapper接口方法** 

```java
List<User> findAllUserAndRole();
```

**配置UserMapper.xml：**

```xml
<resultMap id="userRoleMap" type="user">
	<!--user的信息-->
	<id column="userId" property="id"></id>
	<result column="username" property="username"></result>
	<result column="password" property="password"></result>
	<result column="birthday" property="birthday"></result>
	<!--user内部的roleList信息-->
	<collection property="roleList" ofType="role">
		<id column="roleId" property="id"></id>
		<result column="roleName" property="roleName"></result>
		<result column="roleDesc" property="roleDesc"></result>
	</collection>
</resultMap>

<select id="findUserAndRoleAll" resultMap="userRoleMap">
	SELECT * FROM USER u,sys_user_role ur,sys_role r WHERE u.id=ur.userId AND ur.roleId=r.id
</select>
```

**测试代码：**

```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
List<User> userAndRoleAll = mapper.findUserAndRoleAll();
for (User user : userAndRoleAll) {
    System.out.println(user);
}
```

out:

```
User{id=1, username='tom', password='123456', birthday=null, orderList=null, 
	roleList=[Role{id=1, roleName='院长', roleDesc='负责全面工作'}, 
			 Role{id=2, roleName='研究员', roleDesc='课程研发工作'}]}
User{id=2, username='jerry', password='123456', birthday=null, orderList=null, 
	roleList=[Role{id=2, roleName='研究员', roleDesc='课程研发工作'}, 
			 Role{id=3, roleName='讲师', roleDesc='授课工作'}]}
```



**MyBatis多表配置方式： **

- 一对一配置：使用`<resultMap>`做配置 
- 一对多配置：使用`<resultMap>`+`<collection>`做配置
- 多对多配置：使用`<resultMap>`+`<collection>`做配置

