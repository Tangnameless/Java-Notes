# JdbcTemplate基本使用

参考资料：[使用 JDBC 进行数据访问](https://www.docs4dev.com/docs/zh/spring-framework/4.3.21.RELEASE/reference/jdbc.html)

## JdbcTemplate概述 

它是spring框架中提供的一个对象，是对原始繁琐的Jdbc API对象的简单封装。spring框架为我们提供了很多的操作模板类。例如：操作关系型数据的JdbcTemplate和HibernateTemplate，操作nosql数据库的RedisTemplate，操作消息队列的JmsTemplate等等。



| Action               | Spring   | You |
| ---------------------------- | ------ | ------ |
| 定义连接参数。               |        | X      |
| 打开连接。                   | X      |        |
| 指定 SQL 语句。              |        | X      |
| 声明参数并提供参数值         |        | X      |
| 准备并执行该语句。           | X      |        |
| 设置循环以遍历结果(如果有)。 | X      |        |
| 进行每次迭代的工作。         |        | X      |
| 处理任何异常。               | X      |        |
| Handle transactions.         | X      |        |
| 关闭连接，语句和结果集。     | X      |        |



## JdbcTemplate开发步骤 

**① 导入spring-jdbc和spring-tx坐标** 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```

**② 创建数据库表和实体** 

![image-20220323214153205](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203232141241.png)

```java
public class Account { 
    private String name; 
    private double money; 
    //省略get和set方法
}
```

**③ 创建JdbcTemplate对象**

**④ 执行数据库操作**

```java
public void test1() throws PropertyVetoException {
    // 1.创建数据源对象
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8");
    dataSource.setUser("root");
    dataSource.setPassword("root");
	// 2.创建JdbcTemplate对象
    JdbcTemplate jdbcTemplate = new JdbcTemplate();
    // 3.设置数据源对象(知道数据库在哪)
    jdbcTemplate.setDataSource(dataSource);
    // 4.执行操作
    int row = jdbcTemplate.update("insert into account values(?,?)", "jetty", 15000);
    System.out.println(row);
}
```



## Spring产生JdbcTemplate对象

可以将JdbcTemplate的创建权交给Spring，将数据源DataSource的创建权也交给Spring，在Spring容器内部将数据源DataSource注入到JdbcTemplate模版对象中，配置如下：

> 有些参数在某些阶段中是常量。如在开发阶段我们连接数据库时的url，username，password等信息，分布式应用中client端的server地址，端口等。这些参数在不同阶段之间又往往需要改变。我们可以将这些信息写入到配置文件中，通过spring加载到容器进行使用。
>
> 使用`<context:property-placeholder>`元素可以解决，说明如下：
>
> Activates replacement of ${...} placeholders by registering a PropertySourcesPlaceholderConfigurer within the application context. Properties will be resolved against the specified properties file or Properties object -- so called "local properties", if any, and against the Spring Environment's current set of PropertySources.

```xml
<!--加载jdbc.properties-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--数据源对象-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<!--jdbc模板对象-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

从容器中获得JdbcTemplate进行添加操作

```java
public void test2() throws PropertyVetoException {
    ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
    JdbcTemplate jdbcTemplate = app.getBean(JdbcTemplate.class);
    int row = jdbcTemplate.update("insert into account values(?,?)", "lisi", 5000);
    System.out.println(row);
}
```



## JdbcTemplate的常用操作

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JdbcTemplateCRUDTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    // 测试查询单个简单数据操作(聚合查询)
    public void testQueryCount() {
        Long count = jdbcTemplate.queryForObject("select count(*) from account", Long.class);
        System.out.println(count);
    }

    @Test
    // 测试查询单个对象操作
    public void testQueryOne() {
        Account account = jdbcTemplate.queryForObject("select * from account where name=?", new BeanPropertyRowMapper<Account>(Account.class), "tom");
        System.out.println(account);
    }

    @Test
    public void testQueryAll() {
        List<Account> accountList = jdbcTemplate.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
        System.out.println(accountList);
    }
	
    @Test
    // 测试修改
    public void testUpdate() {
        jdbcTemplate.update("update account set money=? where name=?", 10000, "tom");
    }
    
    @Test
    // 测试删除
    public void testDelete() {
        jdbcTemplate.update("delete from account where name=?", "tom");
    }
}
```



