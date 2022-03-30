# 代理开发方式介绍 

采用Mybatis 的代理开发方式实现 DAO 层的开发，这种方式是我们后面进入企业的主流。 

Mapper 接口开发方法只需要程序员编写Mapper 接口（相当于Dao 接口），由Mybatis 框架根据接口定义创建接口的动态代理对象，代理对象的方法体同上边Dao接口实现类方法。 

Mapper 接口开发需要遵循以下规范： 

1、Mapper.xml文件中的namespace与mapper接口的全限定名相同 

2、Mapper接口方法名和Mapper.xml中定义的每个statement的id相同 

3、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同

4、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同



![image-20220328171122676](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203281711720.png)

(修正了原课件中的错误)



**Dao层（Mapper层）中只定义了接口**

```java
public interface UserMapper {

    public List<User> findAll() throws IOException;

    public User findById(int id);

}
```

```java
public class ServiceDemo {

    public static void main(String[] args) throws IOException {

        InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
	    // 获得MyBatis框架生成的UserMapper接口的实现类
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> all = mapper.findAll();
        System.out.println(all);

        User user = mapper.findById(1);
        System.out.println(user);

    }
}
```

