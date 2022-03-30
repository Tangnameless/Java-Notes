这一章的内容主要是对已学习内容和复习，并在此基础上进行优化。

# 1. 原始方式整合

创建数据库和表

```sql
create database ssm; 
create table account(
	id int primary key auto_increment, name varchar(100), money double(7,2)
);
```

插入一些数据

![image-20220329211200929](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292112953.png)

创建带webapp的maven工程（略）

导入Maven坐标

编写实体类Account

```java
public class Account {

    private Integer id;
    private String name;
    private Double money;
    // 省略set,get,toString()方法
}
```

编写Mapper接口 AccountMapper.java

```java
public interface AccountMapper {
    public void save(Account account);
    public List<Account> findAll();
}
```

编写Service接口

```java
public interface AccountService {
    public void save(Account account);
    public List<Account> findAll();
}
```

编写Service接口实现（未整合方式）

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    public void save(Account account) {
        SqlSession sqlSession = MyBatisUtils.openSession();
        AccountMapper accountMapper = sqlSession.getMapper(AccountMapper.class);
        accountMapper.save(account);
        sqlSession.commit();
        sqlSession.close();
    }

    public List<Account> findAll() {
        SqlSession sqlSession = MyBatisUtils.openSession();
        AccountMapper accountMapper = sqlSession.getMapper(AccountMapper.class);
        return accountMapper.findAll();
    }
}
```

发现需要编写工具类获得SqlSession，手动获得mapper，手动管理事务的提交等等。

编写Controller

```java
@Controller
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    //保存
    @RequestMapping(value = "/save", produces = "text/html;charset=UTF-8")
    @ResponseBody
    public String save(Account account) {
        accountService.save(account);
        return "保存成功";
    }

    //查询
    @RequestMapping("/findAll")
    public ModelAndView findAll() {
        List<Account> accountList = accountService.findAll();
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("accountList", accountList);
        modelAndView.setViewName("accountList");
        return modelAndView;
    }
}
```

编写增加和查列表的jsp页面

1）新增数据页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>添加账户信息表单</h1>
<form name="accountForm" action="${pageContext.request.contextPath}/account/save" method="post">
    账户名称:<input type="text" name="name"><br>
    账户金额:<input type="text" name="money"><br>
    <input type="submit" value="保存"><br>
</form>
</body>
</html>
```

2）查询列表页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>展示账户数据列表</h1>
<table border="1">
    <tr>
        <th>账户id</th>
        <th>账户名称</th>
        <th>账户金额</th>
    </tr>

    <c:forEach items="${accountList}" var="account">
        <tr>
            <td>${account.id}</td>
            <td>${account.name}</td>
            <td>${account.money}</td>
        </tr>
    </c:forEach>

</table>
</body>
</html>
```



**编写相应配置文件** 

- Spring配置文件：applicationContext.xml 
- SprngMVC配置文件：spring-mvc.xml 
- MyBatis映射文件：AccountMapper.xml 
- MyBatis核心文件：sqlMapConfig.xml 
- 数据库连接信息文件：jdbc.properties 
- Web.xml文件：web.xml
- 日志文件：log4j.xml



测试结果：

![image-20220329221457039](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292214070.png)



**完成原始方式整合前进行的配置：**

Spring配置文件中：

- 组件扫描，扫描service

SprngMVC配置文件中：

- 组件扫描 ，主要扫描controller
- 配置mvc注解驱动
- 内部资源视图解析器（主要在controller中返回视图时简化jsp文件位置）
- 开放静态资源访问权限

MyBatis核心文件

- 加载properties文件
- 定义别名
- 配置数据源
- 加载映射

web.xml文件

- spring监听器
- springMVC的前端控制器
- 乱码过滤器



# 2. Spring整合MyBatis

**整合思路：**

![image-20220329215312683](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203292153804.png)

引入

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
</dependency>
```



在applicationContext.xml中

将SqlSessionFactory配置到Spring容器中

```xml
<!--配置sessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <!--加载mybatis核心文件-->
    <property name="configLocation" value="classpath:sqlMapConfig-spring.xml"></property>
</bean>
```

扫描Mapper，让Spring容器产生Mapper实现类

```xml
    <!--扫描mapper所在的包 为mapper创建实现类-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.mapper"></property>
    </bean>
```

配置声明式事务控制

```xml
<!--配置声明式事务控制-->
<!--平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--配置事务增强-->
<tx:advice id="txAdvice">
    <tx:attributes>
        <!--默认配置-->
        <tx:method name="*"/>  
    </tx:attributes>
</tx:advice>

<!--事务的aop织入-->
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.itheima.service.impl.*.*(..))"></aop:advisor>
</aop:config>
```

**修改Service实现类代码**

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountMapper accountMapper;

    @Override
    public void save(Account account) {
        accountMapper.save(account);
    }

    @Override
    public List<Account> findAll() {
        return accountMapper.findAll();
    }
}
```



**完成整合后进行的配置：**

Spring配置文件中：

- ==组件扫描 扫描service和mapper（排除controller的扫描）==
- ==加载propeties文件==
- ==配置数据源信息==
- ==配置sessionFactory==
- ==扫描mapper所在的包 为mapper创建实现类==
- ==配置声明式事务控制==

SprngMVC配置文件中：

- 组件扫描 ，主要扫描controller
- 配置mvc注解驱动
- 内部资源视图解析器（主要在controller中返回视图时简化jsp文件位置）
- 开放静态资源访问权限

MyBatis核心文件

- ~~加载properties文件~~
- 定义别名
- ~~配置数据源~~
- ~~加载映射~~

web.xml文件

- spring监听器
- springMVC的前端控制器
- 乱码过滤器

**总结：**信息从MyBatis核心文件转移到Spring配置文件中

