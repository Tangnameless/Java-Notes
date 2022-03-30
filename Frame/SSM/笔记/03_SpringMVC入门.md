> **环境配置：**
>
> [IntelliJ IDEA创建maven web项目（IDEA新手适用）](https://blog.csdn.net/czc9309/article/details/80304074)
>
> [解决Idea创建maven-archetype-webapp项目无java目录的问题](https://www.cnblogs.com/hafiz/p/5854693.html)

# 1.Spring与Web环境集成

**ApplicationContext应用上下文获取方式**

应用上下文对象是通过`new ClasspathXmlApplicationContext(spring配置文件)` 方式获取的，但是每次从容器中获得Bean时都要编写`new ClasspathXmlApplicationContext(spring配置文件)` ，这样的弊端是配置文件加载多次，应用上下文对象创建多次。

在Web项目中，可以使用ServletContextListener监听Web应用的启动，我们可以在Web应用启动时，就加载Spring的配置文件，创建应用上下文对象ApplicationContext，再将其存储到最大的域servletContext域中，这样就可以在任意位置从域中获得应用上下文ApplicationContext对象了。



**Spring提供获取应用上下文的工具** 

上面的分析不用手动实现（视频教程 P37 - P40）

Spring提供了一个监听器`ContextLoaderListener`就是对上述功能的封装，该监听器内部加载Spring配置文件，创建应用上下文对象，并存储到`ServletContext`域中，提供了一个客户端工具`WebApplicationContextUtils`供使用者获得应用上下文对象。

所以我们需要做的只有两件事： 
- 在`web.xml`中配置`ContextLoaderListener`监听器（导入spring-web坐标）
- 使用`WebApplicationContextUtils`获得应用上下文对象`ApplicationContext`



**导入Spring集成web的坐标**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```



**配置ContextLoaderListener监听器**

```xml
<!--全局初始化参数-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--配置监听器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

contextConfigLocation的初始化对于listener的配置是必须的！



**通过工具获得应用上下文对象**

```java
import org.springframework.web.context.support.WebApplicationContextUtils;
ApplicationContext app = WebApplicationContextUtils.getWebApplicationContext(servletContext);
UserService userService = app.getBean(UserService.class);
```



# 2. SpringMVC简介

**SpringMVC概述**

SpringMVC 是一种基于 Java 的实现MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于 SpringFrameWork 的后续产品，已经融合在 Spring Web Flow 中。

SpringMVC 已经成为目前最主流的MVC框架之一，并且随着Spring3.0 的发布，全面超越 Struts2，成为最优 秀的MVC 框架。它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口。同时它还支持 RESTful 编程风格的请求。



**简单的MVC模型**

我们把`UserServlet`看作业务逻辑处理，把`User`看作模型，把`user.jsp`看作渲染，这种设计模式通常被称为MVC：Model-View-Controller，即`UserServlet`作为控制器（Controller），`User`作为模型（Model），`user.jsp`作为视图（View），整个MVC架构如下：

```ascii
                   ┌───────────────────────┐
             ┌────>│Controller: UserServlet│
             │     └───────────────────────┘
             │                 │
┌───────┐    │           ┌─────┴─────┐
│Browser│────┘           │Model: User│
│       │<───┐           └─────┬─────┘
└───────┘    │                 │
             │                 ▼
             │     ┌───────────────────────┐
             └─────│    View: user.jsp     │
                   └───────────────────────┘
```

使用MVC模式的好处是，Controller专注于业务处理，它的处理结果就是Model。Model可以是一个JavaBean，也可以是一个包含多个对象的Map，Controller只负责把Model传递给View，View只负责把Model给“渲染”出来。



**SpringMVC快速入门**

需求：客户端发起请求，服务器端接收请求，执行逻辑并进行视图跳转。

![](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203220916898.png) 



开发步骤： 

**1、在 pom.xml 导入SpringMVC相关坐标** 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```

**2、在web.xml 配置SpringMVC核心控制器DispathcerServlet** 

```xml
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

**3、创建Controller类和视图页面，使用注解配置Controller类中业务方法的映射地址** 

```java
@Controller 
public class QuickController { 
    @RequestMapping("/quick") 
    public String quickMethod(){ 
        System.out.println("quickMethod running....."); 
        return "success.jsp";
	}
}
```

success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>Success!</h1>
</body>
</html>
```

附：IDEA tomcat 设置虚拟目录

![image-20220322091024009](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203220910080.png)

**5、配置SpringMVC核心文件 spring-mvc.xml**

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.alibaba.com/schema/stat"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.alibaba.com/schema/stat http://www.alibaba.com/schema/stat.xsd">

    <!--Controller的组件扫描-->
    <context:component-scan base-package="com.itheima.controller"/>
</beans>
```



**6、客户端发起请求测试**

访问测试地址：http://localhost:8080/quick



Spring访问流程（代码角度）

![02-Spring访问流程（代码角度）](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203220915209.png)

SpringMVC流程图示

![](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203220918939.png)

# 3. SpringMVC组件解析

## 3.1 SpringMVC的执行流程

![image-20220322092531826](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203220925903.png)

1）用户发送请求至前端控制器DispatcherServlet。 

2）DispatcherServlet收到请求调用HandlerMapping处理器映射器。 

3）处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果 有则生成)一并返回给DispatcherServlet。

4）DispatcherServlet调用HandlerAdapter处理器适配器。 

5）HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。 

6）Controller执行完成返回ModelAndView。 

7）HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

8）DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

9）ViewReslover解析后返回具体View。

10）DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。DispatcherServlet响应用户。



## 3.2 SpringMVC组件解析

**前端控制器：DispatcherServlet** 

用户请求到达前端控制器，它就相当于 MVC 模式中的 C，DispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet 的存在降低了组件之间的耦合性。

**处理器映射器：HandlerMapping** 

HandlerMapping负责根据用户请求找到Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**处理器适配器：HandlerAdapter** 

通过 HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**处理器：Handler** 

它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理。

**视图解析器：View Resolver** 

View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名，即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。

**视图：View** 

SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView等。最常用的视图就是 jsp。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

## 3.3 SpringMVC注解解析

`@RequestMapping`

作用：用于建立请求 URL 和处理请求方法之间的对应关系 

位置： 

- 类上，请求URL 的第一级访问目录。此处不写的话，就相当于应用的根目录 
  
- 方法上，请求 URL 的第二级访问目录，与类上的使用`@ReqquestMapping`标注的一级目录一起组成访问虚拟路径

属性： 

- value：用于指定请求的URL。它和path属性的作用是一样的 

- method：用于指定请求的方式 

- params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样

  例如： 
  -- `params = {"accountName"}`，表示请求参数必须有accountName
  -- `params = {"moeny!100"}`，表示请求参数中money不能是100



**mvc命名空间引入**

命名空间：`xmlns:context="http://www.springframework.org/schema/context" `
		 	    `xmlns:mvc="http://www.springframework.org/schema/mvc"`
约束地址：`http://www.springframework.org/schema/context `
		         `http://www.springframework.org/schema/context/spring-context.xsd `
                 `http://www.springframework.org/schema/mvc`
                 `http://www.springframework.org/schema/mvc/spring-mvc.xsd`

**组件扫描** 

SpringMVC基于Spring容器，所以在进行SpringMVC操作时，需要将Controller存储到Spring容器中，如果使用`@Controller`注解标注的话，就需要使用`<context:component-scan base-package=“com.itheima.controller"/>`进行组件扫描。



## 3.4 SpringMVC的XML配置解析

**视图解析器** 

SpringMVC有默认组件配置，默认组件都是`DispatcherServlet.properties`配置文件中配置的，该配置文件地址 org/springframework/web/servlet/DispatcherServlet.properties，该文件中配置了默认的视图解析器，如下：

```java
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
```

翻看该解析器源码，可以看到该解析器的默认设置，如下：

```java
REDIRECT_URL_PREFIX = "redirect:"  --重定向前缀
FORWARD_URL_PREFIX = "forward:"    --转发前缀（默认值）
prefix = ""; 	--视图名称前缀
suffix = "";	--视图名称后缀
```

可以通过属性注入的方式修改视图的的前后缀

```xml
<!--配置内部资源视图解析器--> 
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
    <property name="prefix" value="/WEB-INF/views/"></property> 
    <property name="suffix" value=".jsp"></property>
</bean>
```
