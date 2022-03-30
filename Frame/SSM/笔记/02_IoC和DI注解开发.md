# 1. Spring配置数据源

**数据源（连接池）的作用**

- 是数据源（连接池）是提高程序性能出现的
- 实现实例化数据源，初始化部分连接资源
- 使用连接资源时从数据源中获取
- 使用完毕后将连接资源归还给数据源

常见的数据源（连接池）：DBCP、C3P0、BoneCP、Druid等



**数据源的开发步骤**

1、导入数据源的坐标和数据库驱动坐标

2、创建数据源对象

3、设置数据源的基本连接数据

4、使用数据源获取连接资源和归还连接资源



## 1.2 数据源的手动创建

1）在 pom.xml 中导入mysql数据库驱动坐标

2）在 pom.xml 中导入C3P0和Druid的坐标

```xml
<!-- mysql驱动，注意和安装的mysql版本对应 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>
<!-- C3P0连接池 -->
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<!-- 连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```



3）创建C3P0连接池和Druid连接池

```java
String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false"
@Test
//测试手动创建 c3p0 数据源
public void test1() throws Exception {
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
    dataSource.setJdbcUrl(url);
    dataSource.setUser("root");
    dataSource.setPassword("root");
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}

@Test
//测试手动创建 druid 数据源
public void test2() throws Exception {
    DruidDataSource dataSource = new DruidDataSource();
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl(url);
    dataSource.setUsername("root");
    dataSource.setPassword("root");
    DruidPooledConnection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```

4）提取jdbc.properties配置文件

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
jdbc.username=root
jdbc.password=root
```

5）读取jdbc.properties配置文件创建连接池

```java
@Test
//测试手动创建 c3p0 数据源(加载properties配置文件)
public void test3() throws Exception {
    //读取配置文件
    ResourceBundle rb = ResourceBundle.getBundle("jdbc");
    String driver = rb.getString("jdbc.driver");
    String url = rb.getString("jdbc.url");
    String username = rb.getString("jdbc.username");
    String password = rb.getString("jdbc.password");
    //创建数据源对象  设置连接参数
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass(driver);
    dataSource.setJdbcUrl(url);
    dataSource.setUser(username);
    dataSource.setPassword(password);

    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```



## 1.3 Spring配置数据源

```java
@Test
//测试Spring容器产生数据源对象
public void test4() throws Exception {
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        DataSource dataSource = app.getBean(DataSource.class);
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
}
```

在`applicationContext.xml`中配置

name对应set方法

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
    <property name="jdbcUrl" value="太长省略"></property>
    <property name="user" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```



## 1.4 抽取jdbc配置文件

applicationContext.xml加载jdbc.properties配置文件获得连接信息。 

首先，需要引入context命名空间和约束路径： 

- 命名空间：`xmlns:context="http://www.springframework.org/schema/context"`

- 约束路径：http://www.springframework.org/schema/context
  				 http://www.springframework.org/schema/context/spring-context.xsd

**Spring容器加载properties文件**

```xml
<!--加载外部的properties文件-->
<context:property-placeholder location="classpath:jdbc.properties"/>
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="user" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>
```

# 2. Spring注解开发

## 2.1 Spring原始注解

Spring是轻代码而重配置的框架，配置比较繁重，影响开发效率，所以注解开发是一种趋势，注解代替xml配置文件可以简化配置，提高开发效率。

Spring原始注解主要是替代`<Bean>`的配置

| 注解             | 说明                                             |
| ---------------- | ------------------------------------------------ |
| `@Component`     | 使用在类上用于实例化Bean                         |
| `@Controller`    | 使用在web层类上用于实例化Bean                    |
| `@Service`       | 使用在service层类上用于实例化Bean                |
| `@Repository`    | 使用在dao层类上用于实例化Bean                    |
| `@Autowired`     | 使用在字段上用于根据类型依赖主入                 |
| `@Qualifier`     | 结合`@Autowired`一起使用用于根据名称进行依赖注入 |
| `@Resource`      | 相当于@Autowired+@Qualifier，按照名称进行注入    |
| `@Value`         | 注入普通属性                                     |
| `@Scope`         | 标注Bean的作用范围                               |
| `@PostConstruct` | 使用在方法上标注该方法是Bean的初始化方法         |
| `@PreDestroy`    | 使用在方法上标注该方法是Bean的销毁方法           |

==**注意：**== 

使用注解进行开发时，需要在applicationContext.xml中配置组件扫描，作用是指定哪个包及其子包下的Bean需要进行扫描以便识别使用注解配置的类、字段和方法。

```xml
<!--配置组件扫描-->
<context:component-scan base-package="com.itheima"/>
```



**使用@Compont或@Repository标识UserDaoImpl需要Spring进行实例化**

原本在配置文件中：

```xml
//<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>
```

使用注解：

```java
//@Component("userDao")
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    public void save() {
        System.out.println("save running...");
    }
}
```



使用@Compont或@Service标识UserServiceImpl需要Spring进行实例化 

```xml
<bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
```

使用@Autowired或者@Autowired+@Qulifier或者@Resource进行userDao的注入

```xml
<property name="userDao" ref="userDao"></property>
```

使用注解：

```java
//@Component("userService")
@Service("userService")
public class UserServiceImpl implements UserService {

    //@Autowired //按照数据类型从Spring容器中进行匹配的
    //@Qualifier("userDao")  //是按照id值从容器中进行匹配的 但是主要此处@Qualifier结合@Autowired一起使用
    @Resource(name="userDao") //@Resource相当于@Qualifier+@Autowired
    private UserDao userDao;

    public void save() {
        userDao.save();
    }
```

使用@Value进行字符串的注入

使用@Scope标注Bean的范围

使用@PostConstruct标注初始化方法，使用@PreDestroy标注销毁方法

```java
@Service("userService")
@Scope("singleton")
public class UserServiceImpl implements UserService {
    @Value("注入普通数据") 
    private String str;
    @Value("${jdbc.driver}")
    private String driver;

    @Resource(name = "userDao") //@Resource相当于@Qualifier+@Autowired
    private UserDao userDao;

    public void save() {
        System.out.println(str);
        System.out.println(driver);
        userDao.save();
    }

    @PostConstruct
    public void init() {
        System.out.println("Service对象的初始化方法");
    }

    @PreDestroy
    public void destory() {
        System.out.println("Service对象的销毁方法");
    }
}
```



使用上面的注解还不能全部替代xml配置文件，还需要使用注解替代的配置如下： 

- 非自定义的Bean的配置：`<bean>` 
- 加载properties文件的配置：`<context:property-placeholder>`
- 组件扫描的配置：`<context:component-scan>`
- 引入其它文件：`<import>`



## 2.2 Spring新注解

| 注解              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `@Configuration`  | 用于指定当前类是一个 Spring 配置类，当创建容器时会从该类上加载注解 |
| `@ComponentScan`  | 用于指定 Spring 在初始化容器时要扫描的包。                                                                                                               作用和在 Spring 的 xml 配置文件中的`<context:component-scan base-package="com.itheima"/>`一样 |
| `@Bean`           | 使用在方法上，标注将该方法的返回值存储到 Spring 容器中       |
| `@PropertySource` | 用于加载 .properties 文件中的配置                            |
| `@Import`         | 用于导入其他配置类                                           |



对应配置文件：

```xml
<context:component-scan base-package="com.itheima"/>
<import resource=""/>
```



```java
//标志该类是Spring的核心配置类
@Configuration
@ComponentScan("com.itheima")
@Import({DataSourceConfiguration.class})
public class SpringCofiguration {
    
}
```

DataSourceConfiguration.java

```java
//<context:property-placeholder location="classpath:jdbc.properties"/>
@PropertySource("classpath:jdbc.properties")
public class DataSourceConfiguration {

    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean("dataSource")  //Spring会将当前方法的返回值以指定名称存储到Spring容器中
    public DataSource getDataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```



测试加载核心配置类创建Spring容器

```java
@Test
public void testAnnoConfiguration() throws Exception {
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringCofiguration.class);
    UserService userService = (UserService) applicationContext.getBean("userService");
    userService.save();
    DataSource dataSource = (DataSource) applicationContext.getBean("dataSource");
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
}
```



# 3. Spring整合Junit

**原始Junit测试Spring的问题** 

在测试类中，每个测试方法都有以下两行代码：

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml"); 
IAccountService as = ac.getBean("accountService",IAccountService.class);
```

这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常。所以又不能轻易删掉。



**上述问题解决思路** 

• 让 SpringJunit 负责创建 Spring容器，但是需要将配置文件的名称告诉它
• 将需要进行测试 Bean 直接在测试类中进行注入



**Spring集成Junit步骤**

① 导入spring集成Junit的坐标（spring5 及以上版本要求 junit 的版本必须是 4.12 及以上）
② 使用 `@Runwith` 注解替换原来的运行期 
③ 使用 `@ContextConfiguration` 指定配置文件或配置类 
④ 使用 `@Autowired` 注入需要测试的对象
⑤ 创建测试方法进行测试



```java
@RunWith(SpringJUnit4ClassRunner.class)
// 加载spring核心配置文件
// @ContextConfiguration(value = {"classpath:applicationContext.xml"})
// 加载spring核心配置类
@ContextConfiguration(classes = {SpringCofiguration.class})
//@ContextConfiguration("classpath:applicationContext.xml")
public class SpringJunitTest {

    @Autowired
    private UserService userService;

    @Autowired
    private DataSource dataSource;

    @Test
    public void test1() throws SQLException {
        userService.save();
        System.out.println(dataSource.getConnection());
    }
}
```

