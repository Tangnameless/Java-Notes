# 1. 拦截器的作用

Spring MVC 的拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。 

将拦截器按一定的顺序联结成一条链，这条链称为拦截器链（Interceptor Chain）。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。拦截器也是AOP思想的具体实现。



# 2. 拦截器和过滤器区别

| 区别 | 过滤器（Filter） |  拦截器（Interceptor）   |
| ---- | ------ | ---- |
| 使用范围 | 是 servlet 规范中的一部分，任何Java Web 工程都可以使用       |   是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用   |
| 拦截范围 | 在 url-pattern 中配置了`/*`之后， 可以对所有要访问的资源拦截 | 在`<mvc:mapping path=“”/>`中配置了`/**`之 后，也可以对所有资源进行拦截，但是可以通过`<mvc:exclude-mapping path=“”/>`标签排除不需要拦截的资源。 |



> **拦截器路径通配符**
>
> | 通配符 |      | 说明                                                         |
> | ------ | ---- | ------------------------------------------------------------ |
> | *      |      | 匹配单个字符，如 `/user/*` 匹配的是 `/user/aa`，`/user/bb` 等，又如  `/user/*/ab` 匹配到 `/user/p/ab`； |
> | **     |      | 匹配任意多字符(包括多级路径)，如：`/user/**` 匹配到 `/user/aa`、`/user/p/bb` 等； |



# 3. 拦截器快速入门

**1、创建拦截器类实现`HandlerInterceptor`接口。**

```java
public class MyInterceptor1 implements HandlerInterceptor {
    //在目标方法执行之前执行
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        System.out.println("preHandle.....");
        String param = request.getParameter("param");
        if("yes".equals(param)){
            return true;
        }else{
            request.getRequestDispatcher("/error.jsp").forward(request,response);
            return false;//返回true代表放行  返回false代表不放行
        }
    }

    //在目标方法执行之后 视图对象返回之前执行
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        modelAndView.addObject("name","itheima");
        System.out.println("postHandle...");
    }

    //在流程都执行完毕后执行
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("afterCompletion....");
    }
}
```

**2、配置拦截器**

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--对哪些资源执行拦截操作-->
        <mvc:mapping path="/**"/>
        <bean class="com.itheima.interceptor.MyInterceptor2"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <!--对哪些资源执行拦截操作-->
        <mvc:mapping path="/**"/>
        <bean class="com.itheima.interceptor.MyInterceptor1"/>
    </mvc:interceptor>
</mvc:interceptors>
```



**拦截器方法说明**

| 方法名              | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| `preHandle()`       | 方法将在请求处理之前进行调用，该方法的返回值是布尔值Boolean类型的， 当它返回为false 时，表示请求结束，后续的Interceptor 和Controller 都不会 再执行；当返回值为true 时就会继续调用下一个Interceptor 的preHandle 方法 |
| `postHandle()`      | 该方法是在当前请求进行处理之后被调用，前提是preHandle 方法的返回值为 true 时才能被调用，且它会在DispatcherServlet 进行视图返回渲染之前被调 用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作 |
| `afterCompletion()` | 该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图 之后执行，前提是preHandle 方法的返回值为true 时才能被调用 |



# 4. 案例-用户登录权限控制

在[D:\learning-area\SSM\代码\6_itheima_spring_test](D:\learning-area\SSM\代码\6_itheima_spring_test)的基础上进行修改，使用拦截器实现根据session的登陆验证。

需求：用户没有登录的情况下，不能对后台菜单进行访问操作，点击菜单跳转到登录页面，只有用户登录成功后才能进行后台功能的操作。

![image-20220324160408777](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203241604838.png)



==特别说明！==

按照老师给的例子在spring-mvc.xml中配置拦截器，会把静态资源也拦截掉！**拦截器中需要增加针对静态资源不进行过滤**

```xml
<!--配置权限拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--配置对哪些资源执行拦截操作-->
        <mvc:mapping path="/**"/>
        <!--配置哪些资源排除拦截操作-->
        <mvc:exclude-mapping path="/user/login"/>
        <mvc:exclude-mapping path="/**/fonts/*"/>
        <mvc:exclude-mapping path="/**/*.css"/>
        <mvc:exclude-mapping path="/**/*.js"/>
        <mvc:exclude-mapping path="/**/*.png"/>
        <mvc:exclude-mapping path="/**/*.gif"/>
        <mvc:exclude-mapping path="/**/*.jpg"/>
        <mvc:exclude-mapping path="/**/*.jpeg"/>
        <bean class="com.itheima.interceptor.PrivilegeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

[更多解决方案](https://www.cnblogs.com/mophy/p/8465598.html)



首先实现一个自定义拦截器 PrivilegeInterceptor

```java
public class PrivilegeInterceptor implements HandlerInterceptor {

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        //逻辑：判断用户是否登录  本质：判断session中有没有user
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        if (user == null) {
            //没有登录
            response.sendRedirect(request.getContextPath() + "/login.jsp");
            return false;
        }
        //放行  访问目标资源
        return true;
    }
}
```

拦截器的逻辑非常简单，判断每个请求中的session（HttpSession）中，是否带有user。带有就认为是登陆，不带有就认为是未登录。

> **HttpSession 服务端的技术**
> 服务器会为每一个用户 创建一个独立的HttpSession
>
> **HttpSession原理**
> 当用户第一次访问Servlet时,服务器端会给用户创建一个独立的Session，并且生成一个SessionID，这个SessionID在响应浏览器的时候会被装进cookie中，从而被保存到浏览器中。
> 当用户再一次访问Servlet时，请求中会携带着cookie中的SessionID去访问
> 服务器会根据这个SessionID去查看是否有对应的Session对象，有就拿出来使用;没有就创建一个Session(相当于用户第一次访问)
>
> **域的范围:**
>     Context域 > Session域 > Request域
>     Session域 只要会话不结束就会存在 但是Session有默认的存活时间(30分钟)
>
> ![how HttpSession works](https://static.studytonight.com/servlet/images/how-httpsession-works.jpg)
>
> **session进行身份验证的原理：**
>
> 当客户端第一次访问服务器的时候，此时客户端的请求中不携带任何标识给服务器，所以此时服务器无法找到与之对应的
>
> session，所以会新建session对象，当服务器进行响应的时候，服务器会将session标识放到响应头的Set-Cookie中，会以
>
> key-value的形式返回给客户端，例：JSESSIONID=7F149950097E7B5B41B390436497CD21；其中JSESSIONID是固定的，
>
> 而后面的value值对应的则是给该客户端新创建的session的ID，之后浏览器再次进行服务器访问的时候，客户端会将此key-value
>
> 放到cookie中一并请求服务器，服务器就会根据此ID寻找对应的session对象了；（当浏览器关闭后，会话结束，由于cookie消
>
> 失所以对应的session对象标识消失，而对应的session依然存在，但已经成为报废数据等待GC回收了）
>
> 对应session的ID可以利用此方法得到：`session.getId();`
>
> **个人理解：**
>
> cookie是一个存在本地浏览器端的词典，HttpSession是一个存在服务器端的词典。当用户登陆一个网站的时候，服务器端（假如是java web）给用户创建一个独立的Session，并且生成一个SessionID，SessionID在响应的时候给装到浏览器的cookie里。用户第二次访问的时候，请求中会携带着cookie中的SessionID去访问。
> 服务器端这个字典里，存的是{SessionID : HttpSession}，cookie里存的可能是{"SessionID" : SessionID}
>
> 不同网站的cookie是不同的，浏览器保证跨域隔离。



在UserController中，实现一个 /login 接口

```java
@RequestMapping("/login")
public String login(String username, String password, HttpSession session) {
    User user = userService.login(username, password);
    if (user != null) {
        //登录成功 将user存储到session
        session.setAttribute("user", user);
        return "redirect:/index.jsp";
    }
    return "redirect:/login.jsp";
}
```

在service层实现一个login方法，判断数据库中是否存在该用户且密码正确。

```java
public User login(String username, String password) {
    try {
        User user = userDao.findByUsernameAndPassword(username,password);
        return user;
    }catch (EmptyResultDataAccessException e){
        return null;
    }
}
```


