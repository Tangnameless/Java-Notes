> 官方文档：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html

# 1. Spring简介

Spring是分层的 Java SE/EE应用 full-stack 轻量级开源框架，以 **IoC**（Inverse Of Control：反转控制）和 **AOP**（Aspect Oriented Programming：面向切面编程）为内核。
提供了**展现层 SpringMVC** 和**持久层 Spring JDBCTemplate** 以及**业务层事务管理**等众多的企业级应用技术，还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的Java EE 企业应用开源框架。

在Spring Framework基础上，又诞生了Spring Boot、Spring Cloud、Spring Data、Spring Security等一系列基于Spring Framework的项目。

> **IoC容器**
>
> 在学习Spring框架时，我们遇到的第一个也是最核心的概念就是容器。
>
> 什么是容器？容器是一种为某种特定组件的运行提供必要支持的一个软件环境。例如，Tomcat就是一个Servlet容器，它可以为Servlet的运行提供运行环境。类似Docker这样的软件也是一个容器，它提供了必要的Linux环境以便运行一个特定的Linux进程。
>
> 通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。例如，Servlet容器底层实现了TCP连接，解析HTTP协议等非常复杂的服务，如果没有容器来提供这些服务，我们就无法编写像Servlet这样代码简单，功能强大的组件。早期的JavaEE服务器提供的EJB容器最重要的功能就是通过声明式事务服务，使得EJB组件的开发人员不必自己编写冗长的事务处理代码，所以极大地简化了事务处理。
>
> Spring的核心就是提供了一个IoC容器，它可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。

# 2. Spring快速入门

传统方式在Service中调用Dao层的方法，需要在Service中创建一个Dao对象。

![image-20220330101439838](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301014916.png)

使用Spring框架，直接获取Dao对象。

![image-20220318145435301](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203272238053.png)



**Spring程序开发步骤**
1、导入 Spring 开发的基本包坐标 
2、编写 Dao 接口和实现类 
3、创建 Spring 核心配置文件 
4、在 Spring 配置文件中配置 UserDaoImpl
5、使用 Spring 的 API 获得 Bean 实例



**1、导入Spring开发的基本包坐标**

```xml
<properties> 
    <spring.version>5.0.5.RELEASE</spring.version> 
</properties>
<dependencies> 
    <!--导入spring的context坐标，context依赖core、beans、expression--> 
    <dependency> 
        <groupId>org.springframework</groupId> 
        <artifactId>spring-context</artifactId> 
        <version>${spring.version}</version>
	</dependency>
</dependencies>
```



**2、编写Dao接口和实现类**

```java
public interface UserDao {
    public void save();
}
```



```java
public class UserDaoImpl implements UserDao {
    public void save() {
        System.out.println("save running...");
    }
}
```



**3、创建Spring核心配置文件** 

在类路径下（resources）创建applicationContext.xml配置文件。



**4、在Spring配置文件中配置UserDaoImpl**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl" scope="singleton"></bean>
        <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
            <constructor-arg name="userDao" ref="userDao"></constructor-arg>
        </bean>
</beans>
```



**5、使用Spring的API获得Bean实例**

```java
ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
UserDao userDao = (UserDao)app.getBean("userDao");
userDao.save();
```


# 3. Spring配置文件

## 3.1 Bean标签基本配置

用于配置对象交由Spring来创建。

默认情况下它调用的是类中的**无参构造函数**，如果没有无参构造函数则不能创建成功。

基本属性：

- id: Bean实例在Spring容器中的唯一标识
- class：Bean的全限定名称

## 3.2 Bean标签范围配置

scope: 指对象的作用范围

| 取值范围       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| singleton      | 默认值，单例的                                               |
| prototype      | 多例的                                                       |
| request        | WEB 项目中，Spring 创建一个 Bean 的对象，将对象存入到 request 域中 |
| session        | WEB 项目中，Spring 创建一个 Bean 的对象，将对象存入到 session 域中 |
| global session | WEB 项目中，应用在 Portlet 环境，如果没有 Portlet 环境那么globalSession 相当 于 session |



**1）当scope的取值为`singleton`时** 

![img](https://www.docs4dev.com/images/spring-framework/5.1.3.RELEASE/singleton.png)

Bean的实例化个数：1个

Bean的实例化时机：当Spring核心文件被加载时，实例化配置的Bean实例 

Bean的生命周期： 
- 对象创建：当应用加载，创建容器时，对象就被创建了 
- 对象运行：只要容器在，对象一直活着 
- 对象销毁：当应用卸载，销毁容器时，对象就被销毁了

  

**2）当scope的取值为`prototype`时** 

![prototype](https://www.docs4dev.com/images/spring-framework/5.1.3.RELEASE/prototype.png)

Bean的实例化个数：多个 

Bean的实例化时机：当调用`getBean()`方法时实例化Bean 
- 对象创建：当使用对象时，创建新的对象实例 
- 对象运行：只要对象在使用中，就一直活着
- 对象销毁：当对象长时间不用时，被 Java 的垃圾回收器回收了

## 3.3 Bean生命周期配置 

init-method：指定类中的初始化方法名称
destroy-method：指定类中销毁方法名称

## 3.4 Bean实例化三种方式

**1）用构造函数实例化**

当通过构造方法创建一个 bean 时，所有普通类都可以被 Spring 使用并与之兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。只需指定 bean 类就足够了。但是，根据您用于该特定 bean 的 IoC 的类型，您可能需要一个默认(空)构造函数。

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

**如果bean中没有默认无参构造函数，将会创建失败。**

举例：在UserDaoImpl中只写一个有参构造函数。

```java
public class UserDaoImpl implements UserDao {
    private int num;

    public UserDaoImpl(int num) {
        this.num = num;
        System.out.println("创建, num为：" + num);
    }
    
    public void save() {
        System.out.println("save running...");
    }
}
```

在Spring配置文件中，要给出参数

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl" scope="singleton">
    <constructor-arg name="num" value="1"/>
</bean>
```

否则会提示没有匹配的构造方法：

![image-20220330124110180](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301241223.png)



**2）工厂静态方法实例化**

工厂的静态方法返回Bean实例

定义使用静态工厂方法创建的 bean 时，请使用`class`属性来指定包含`static`工厂方法的类，并使用名为`factory-method`的属性来指定工厂方法本身的名称。您应该能够调用此方法(带有可选参数，如稍后所述)并返回一个活动对象，该对象随后将被视为已通过构造函数创建。

以下 bean 定义指定通过调用工厂方法来创建 bean。该定义不指定返回对象的类型(类)，而仅指定包含工厂方法的类。

在此的示例`createInstance()`方法必须是静态方法。

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

与前面的 bean 定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```



**3）工厂实例方法实例化**

工厂的非静态方法返回Bean实例

与通过[静态工厂方法](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-class-static-factory-method)实例化类似，使用实例工厂方法实例化从容器中调用现有 bean 的非静态方法以创建新 bean。要使用此机制，请将`class`属性留空，并在`factory-bean`属性中，在当前(或父容器或祖先容器)中指定包含要创建该对象的实例方法的 bean 的名称。使用`factory-method`属性设置工厂方法本身的名称。

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

相应的Java类

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

## 3.5 Bean的依赖注入

创建 UserService，我们希望UserService 在内部调用 UserDao的`save()` 方法。

**Bean的依赖注入分析**

假如不进行依赖注入，在UserServiceImpl中需要通过getBean获得UserDao实例，测试程序要使用UserService时，也需要通过getBean获得UserService实例

![image-20220330125157168](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301251229.png)

因为UserService和UserDao都在Spring容器中，而最终程序直接使用的是UserService，所以可以在 Spring容器中，将UserDao设置到UserService内部。

![image-20220318161654296](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203181616347.png)



**Bean的依赖注入概念**

依赖注入（==Dependency Injection==）：它是 Spring 框架核心 IOC 的具体实现。 

在编写程序时，通过控制反转，把对象的创建交给了 Spring，但是代码中不可能出现没有依赖的情况。 IOC 解耦只是降低他们的依赖关系，但不会消除。例如：业务层仍会调用持久层的方法。那这种业务层和持久层的依赖关系，在使用 Spring 之后，就让 Spring 来维护了。

简单的说，就是坐等框架把持久层对象传入业务层，而不用我们自己去获取。



**Bean的依赖注入方式**

- **构造方法**

  创建有参构造

  ```java
  public class UserServiceImpl implements UserService {
      private UserDao userDao;
  
      public UserServiceImpl(UserDao userDao) {
          this.userDao = userDao;
      }
  
      public void save() {
          userDao.save();
      }
  }
  ```
	配置Spring容器调用有参构造时进行注入
  ```xml
  <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
      <constructor-arg name="userDao" ref="userDao"></constructor-arg>
  </bean>
  ```

  

- **set方法**

  在UserServiceImpl中添加setUserDao方法 

  ```java
  public class UserServiceImpl implements UserService {
      private UserDao userDao;
  
      public void setUserDao(UserDao userDao) {
          this.userDao = userDao;
      }
  
      @Override
      public void save() {
          userDao.save();
      }
  }
  ```

  配置Spring容器调用set方法进行注入

  ```xml
  <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/> 
  
  <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"> 
      <property name="userDao" ref="userDao"/>
  </bean>
  ```

  P命名空间注入本质也是set方法注入，但比起上述的set方法注入更加方便，主要体现在配置文件中，如下： 

  首先，需要引入P命名空间：`xmlns:p="http://www.springframework.org/schema/p"` 

  其次，需要修改注入方式

  ```xml
  <bean id="userService" class="com.itheima.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>
  ```

  



**Bean的依赖注入的数据类型**

除了对象的引用可以注入，普通数据类型，集合等都可以在容器中进行注入。

注入数据的三种数据类型 

- 普通数据类型 
- 引用数据类型
- 集合数据类型



### 普通数据类型和集合数据类型的注入

**1）直值(Primitives，字符串等)**

举例：

```java
public class UserDaoImpl implements UserDao {
    private String company;
    private int age;
    // ...
}
```

配置：

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"> 
    <property name="company" value="传智播客"></property> 
    <property name="age" value="15"></property>
</bean>
```



**2）Collections**

`<list/>`，`<set/>`，`<map/>`和`<props/>`元素分别设置 Java `Collection`类型`List`，`Set`，`Map`和`Properties`的属性和参数。

注入List举例：

```java
public class UserDaoImpl implements UserDao {
    private List<String> strList;
    // ...
}
```

配置：

```xml
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"> 
    <property name="strList"> 
        <list> 
            <value>aaa</value> 
            <value>bbb</value> 
            <value>ccc</value>
		</list> 
    </property>
</bean>
```

如果要注入的是一个对象的集合呢？使用`<ref bean="xxx" />`

比如一个`List<User>`的注入

```xml
<bean id="u1" class="com.itheima.domain.User"/>
<bean id="u2" class="com.itheima.domain.User"/>
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <property name="userList">
        <list>
            <bean class="com.itheima.domain.User"/>
            <bean class="com.itheima.domain.User"/>
            <ref bean="u1"/>
            <ref bean="u2"/>
        </list>
    </property>
</bean>
```

打印输出：

```
[com.itheima.domain.User@3c130745, com.itheima.domain.User@cd3fee8, com.itheima.domain.User@3e2e18f2, com.itheima.domain.User@470f1802]
```



> 通过`<ref/>`标签的`bean`属性指定目标 bean 是最通用的形式，并且允许创建对同一容器或父容器中任何 bean 的引用，而不管它是否在同一 XML 文件中。 `bean`属性的值可以与目标 Bean 的`id`属性相同，也可以与目标 Bean 的`name`属性中的值之一相同。



Map, Set, Properties注入如下：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">[emailprotected]</prop>
            <prop key="support">[emailprotected]</prop>
            <prop key="development">[emailprotected]</prop>
        </props>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

## 3.6 引入其他配置文件（分模块开发） 

实际开发中，Spring的配置内容非常多，这就导致Spring配置很繁杂且体积很大，所以，可以将部分配置拆解到其他配置文件中，而在Spring主配置文件通过import标签进行加载`<import resource="applicationContext-xxx.xml"/>`



**总结Spring的重点配置：**

```xml
<bean>标签 
	id属性:在容器中Bean实例的唯一标识，不允许重复 
	class属性:要实例化的Bean的全限定名 
	scope属性:Bean的作用范围，常用是singleton(默认)和prototype 
	<property>标签：属性注入 
		name属性：属性名称 
		value属性：注入的普通属性值 
		ref属性：注入的对象引用值 
		<list>标签 
         <map>标签
		<properties>标签 
	<constructor-arg>标签
<import>标签:导入其他的Spring的分文件
```



# 4. Spring相关API

**ApplicationContext的继承体系** 

applicationContext：接口类型，代表应用上下文，可以通过其实例获得 Spring 容器中的 Bean 对象



**ApplicationContext的实现类**

1）`ClassPathXmlApplicationContext`
	从类的根路径下加载配置文件 推荐使用这种
2）`FileSystemXmlApplicationContext`
	从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。
3）`AnnotationConfigApplicationContext`
	当使用注解配置容器对象时，需要使用此类来创建 Spring 容器。它用来读取注解。



`getBean()`方法使用

```java
public Object getBean(String name) throws BeansException { 
	assertBeanFactoryActive(); 
    return getBeanFactory().getBean(name);
}
public <T> T getBean(Class<T> requiredType) throws BeansException { 
    assertBeanFactoryActive(); 
    return getBeanFactory().getBean(requiredType);
}
```

其中，当参数的数据类型是字符串时，表示根据Bean的id从容器中获得Bean实例，返回是Object，需要强转。 

当参数的数据类型是Class类型时，表示根据类型从容器中匹配Bean实例，当容器中相同类型的Bean有多个时，则此方法会报错。



**Spring的重点API:**

```java
ApplicationContext app = new ClasspathXmlApplicationContext("xml文件") 
app.getBean("id")
app.getBean(xxx.class)
```

