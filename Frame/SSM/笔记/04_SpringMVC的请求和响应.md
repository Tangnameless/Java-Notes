# 1. SpringMVC的数据响应

**SpringMVC的数据响应方式** 

1）页面跳转 

- 直接返回字符串 
- 通过ModelAndView对象返回

2）回写数据 

- 直接返回字符串
- 返回对象或集合

## 1.1 页面跳转

代码继承自课程03，前置的配置参考之。

### 1）返回字符串形式

直接返回字符串：此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转。

![image-20220322191857380](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203221918432.png)

返回带有前缀的字符串： 

转发：`forward:/WEB-INF/views/index.jsp`
重定向：`redirect:/index.jsp`



请求地址：http://localhost:8080/user/quick?username=tom

```java
@RequestMapping(value = "/quick", method = RequestMethod.GET, params = {"username"})
public String save() {
    System.out.println("Controller save running....");
    return "success";
}
```



### 2）返回ModelAndView对象

使用ModelAndView类用来存储处理完后的结果数据，以及显示该数据的视图。业务处理器调用模型层处理完用户请求后，把结果数据存储在该类的model属性中，把要返回的视图信息存储在该类的view属性中，然后让该ModelAndView返回该Spring MVC框架。框架通过调用配置文件中定义的视图解析器，对该对象进行解析，最后把结果数据显示在指定的页面上。


```java
void  setViewName(StringviewName)  //此ModelAndView的设置视图名称，由通过一个ViewResolverDispatcherServlet会得到解决。
```



```java
@RequestMapping(value = "/quick2")
public ModelAndView save2() {
    ModelAndView modelAndView = new ModelAndView();
    //设置模型数据
    modelAndView.addObject("username", "itcast");
    //设置视图名称
    modelAndView.setViewName("success");
    return modelAndView;
}

@RequestMapping(value = "/quick3")
public ModelAndView save3(ModelAndView modelAndView) {
    modelAndView.addObject("username", "itheima");
    modelAndView.setViewName("success");
    return modelAndView;
}
```

### 3）向request域存储数据 

在进行转发时，往往要向request域中存储数据，在jsp页面中显示，那么Controller中怎样向request 域中存储数据呢？

> request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据
>
> **request可以作为数据流传的载体,解决了一次请求内的不同 Servlet 的数据(请求数据+其他数据)共享问题。**
>
> 作用域：基于请求转发，一次请求中的所有 Servlet 共享。

① 通过SpringMVC框架注入的request对象`setAttribute()`方法设置

```java
@RequestMapping(value = "/quick5")
public String save5(HttpServletRequest request) {
    request.setAttribute("username", "酷丁鱼");
    return "success";
}
```

② 通过ModelAndView的`addObject()`方法设置

```java
@RequestMapping(value = "/quick5-2")
public ModelAndView save5_2(){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("forward:/jsp/success.jsp");
    modelAndView.addObject("name","lisi");
    return modelAndView;
}
```



## 1.2 回写数据

### 1）直接返回字符串 

Web基础阶段，客户端访问服务器端，如果想直接回写字符串作为响应体返回的话，只需要使用

```java
response.getWriter().print("hello world")
```

即可，那么在Controller中想直接回写字符串该怎样呢？

① 通过SpringMVC框架注入的response对象，使用`response.getWriter().print(“hello world”)` 回写数据，此时不需要视图跳转，业务方法返回值为`void`。

```java
@RequestMapping(value = "/quick6")
public void save6(HttpServletResponse response) throws IOException {
    response.getWriter().print("hello world");
}
```

② 将需要回写的字符串直接返回，但此时需要通过`@ResponseBody`注解告知SpringMVC框架，方法返回的字符串不是跳转是直接在http响应体中返回。

```java
@RequestMapping(value = "/quick7")
@ResponseBody  //告知SpringMVC框架 不进行视图跳转 直接进行数据响应
public String save7() throws IOException {
    return "hello world";
}
```

在异步项目中，客户端与服务器端往往要进行json格式字符串交互，此时我们可以手动拼接json字符串返回。

```java
@RequestMapping(value = "/quick8")
@ResponseBody
public String save8() throws IOException {
    return "{\"username\":\"zhangsan\",\"age\":18}";
}
```

上述方式手动拼接json格式字符串的方式很麻烦，开发中往往要将复杂的java对象转换成json格式的字符串， 我们可以使用web阶段学习过的json转换工具jackson进行转换，导入jackson坐标。

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

通过jackson转换json格式字符串，回写字符串。

```java
@RequestMapping(value = "/quick9")
@ResponseBody
public String save9() throws IOException {
    User user = new User();
    user.setUsername("lisi");
    user.setAge(30);
    //使用json的转换工具将对象转换成json格式字符串在返回
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(user);

    return json;
}
```



### 2）返回对象或集合

通过SpringMVC帮助我们对对象或集合进行json字符串的转换并回写，为处理器适配器配置消息转换参数， 指定使用jackson进行对象或集合的转换，因此需要在 **spring-mvc.xml** 中进行如下配置：

```xml
<!--配置处理器映射器-->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        </list>
    </property>
</bean>
```
在方法上添加`@ResponseBody`就可以返回json格式的字符串
```java
@RequestMapping(value = "/quick10")
@ResponseBody
//期望SpringMVC自动将User转换成json格式的字符串
public User save10() throws IOException {
    User user = new User();
    user.setUsername("tom");
    user.setAge(32);
    return user;
}
```

可以使用mvc的注解驱动代替上述配置。

```xml
<!--mvc的注解驱动-->
<mvc:annotation-driven conversion-service="conversionService"/>
```

在 SpringMVC的各个组件中，处理器映射器、处理器适配器、视图解析器称为 SpringMVC 的三大组件。 使用`<mvc:annotation-driven>`自动加载 RequestMappingHandlerMapping（处理映射器）和 RequestMappingHandlerAdapter（处理适配器），可用在Spring-mvc.xml配置文件中使用 `<mvc:annotation-driven>`替代注解处理器和适配器的配置。
同时使用`<mvc:annotation-driven>`默认底层就会集成jackson进行对象或集合的json格式字符串的转换。

# 2. SpringMVC获得请求数据

客户端请求参数的格式是：`name=value&name=value…`

## 2.1 获得基本类型参数

Controller中的业务方法的参数名称要与请求参数的name一致，参数值会自动映射匹配。

```http
GET http://localhost:8080/user/quick11?username=zhangsan&age=12
```

```java
@RequestMapping(value = "/quick11")
@ResponseBody
public void save11(String username, int age) throws IOException {
    System.out.println(username);
    System.out.println(age);
}
```



## 2.2 获得POJO类型参数

Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值会自动映射匹配。

```http
GET http://localhost:8080/user/quick12?username=zhangsan&age=12
```



```java
@RequestMapping(value = "/quick12")
@ResponseBody
public void save12(User user) throws IOException {
    System.out.println(user);
}
```

out:

```
User{username='zhangsan', age=12}
```



## 2.3 获得数组类型参数

Controller中的业务方法数组名称与请求参数的name一致，参数值会自动映射匹配。

```http
GET http://localhost:8080/user/quick13?strs=111&strs=222&strs=333
```



```java
@RequestMapping(value = "/quick13")
@ResponseBody
public void save13(String[] strs) throws IOException {
    System.out.println(Arrays.asList(strs));
}
```

out:

```
[111, 222, 333]
```



## 2.4 获得集合类型参数

获得集合参数时，要将集合参数包装到一个POJO中才可以。

定义一个类VO

```java
public class VO {

    private List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }

    @Override
    public String toString() {
        return "VO{" +
                "userList=" + userList +
                '}';
    }
}
```

定义一个上传表单form.jsp：

```jsp
<form action="${pageContext.request.contextPath}/user/quick14" method="post">
    <%--表明是第几个User对象的username age--%>
    <input type="text" name="userList[0].username"><br/>
    <input type="text" name="userList[0].age"><br/>
    <input type="text" name="userList[1].username"><br/>
    <input type="text" name="userList[1].age"><br/>
    <input type="submit" value="提交">
</form>
```



> `${pageContext.request.contextPath}`是JSP取得绝对路径的方法，等价于`<%=request.getContextPath()%>` 
>
>   也就是取出部署的应用程序名或者是当前的项目名称。
>
>   比如项目名称是demo1，在浏览器中输入为`**http://localhost:8080/demo1/a.jsp` ，使用`${pageContext.request.contextPath}`或`<%=request.getContextPath()%>`取出来的就是`/demo1`,而"/"代表的含义就是`http://localhost:8080`
>
>   故有时候项目中这样写 `${pageContext.request.contextPath}/a.jsp`





```java
@RequestMapping(value = "/quick14")
@ResponseBody
public void save14(VO vo) throws IOException {
    System.out.println(vo);
}
```

首先访问form.jsp，提交请求

out:

```
VO{userList=[User{username='tom', age=11}, User{username='jerry', age=12}]}
```



当使用ajax提交时，可以指定contentType为json形式，那么在==方法参数位置==使用`@RequestBody`可以直接接收集合数据而无需使用POJO进行包装。

```javascript
<script src="${pageContext.request.contextPath}/js/jquery-3.3.1.js"></script>
<script>
    var userList = new Array();
    userList.push({username:"zhangsan",age:18});
    userList.push({username:"lisi",age:28});

    $.ajax({
        type:"POST",
        url:"${pageContext.request.contextPath}/user/quick15",
        data:JSON.stringify(userList),
        contentType:"application/json;charset=utf-8"
    });

</script>
```



```java
@RequestMapping(value = "/quick15")
@ResponseBody
public void save15(@RequestBody List<User> userList) throws IOException {
    System.out.println(userList);
}
```

SpringMVC的前端控制器 DispatcherServlet的url-pattern配置的是/，代表对所有的资源都进行过滤操作，我们可以通过以下两种
方式指定放行静态资源，在**spring-mvc.xml**中

```xml
<!--开放资源的访问-->
<mvc:resources mapping="/js/**" location="/js/"/>
<mvc:resources mapping="/img/**" location="/img/"/>

<mvc:default-servlet-handler/>
```

## 2.5 请求数据乱码问题

当post请求时，数据会出现乱码，我们可以设置一个过滤器来进行编码的过滤。

在web.xml中

```xml
<!--配置全局过滤的filter-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



## 2.6 参数绑定注解@requestParam

当请求的参数名称与Controller的业务方法参数名称不一致时，就需要通过@RequestParam注解显示的绑定。

![image-20220322223233481](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203222232537.png)

注解@RequestParam还有如下参数可以使用： 

- value：与请求参数名称对应
- required：此在指定的请求参数是否必须包括，默认是true，提交时如果没有此参数则报错
- defaultValue：当没有指定请求参数时，则使用指定的默认值赋值



```java
@RequestMapping(value = "/quick16")
@ResponseBody
public void save16(@RequestParam(value = "name", required = false, defaultValue = "itcast") String username) throws IOException {
    System.out.println(username);
}
```

## 2.7 获得Restful风格的参数

在SpringMVC中可以使用占位符进行参数绑定。地址`/user/1`可以写成 `/user/{id}`，占位符`{id}`对应的就是1的值。在业务方法中我们可以使用`@PathVariable`注解进行占位符的匹配获取工作。

```http	
GET localhost:8080/user/quick17/zhangsan
```



```java
@RequestMapping(value = "/quick17/{name}")
@ResponseBody
public void save17(@PathVariable(value = "name") String username) throws IOException {
    System.out.println(username);
}
```

## 2.8 自定义类型转换器

SpringMVC 默认已经提供了一些常用的类型转换器，例如客户端提交的字符串转换成int型进行参数设置。 但是不是所有的数据类型都提供了转换器，没有提供的就需要自定义转换器，例如：日期类型的数据就需要自定义转换器。

**自定义类型转换器的开发步骤：** 

1. 定义转换器类实现Converter接口 

   ```java
   public class DateConverter implements Converter<String, Date> {
       public Date convert(String dateStr) {
           //将日期字符串转换成日期对象 返回
           SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
           Date date = null;
           try {
               date = format.parse(dateStr);
           } catch (ParseException e) {
               e.printStackTrace();
           }
           return date;
       }
   }
   ```

2. 在配置文件 **spring-mvc.xml** 中声明转换器

   ```xml
   <!--声明转换器-->
   <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
       <property name="converters">
           <list>
               <bean class="com.itheima.converter.DateConverter"></bean>
           </list>
       </property>
   </bean>
   ```

   

3. 在`<annotation-driven>`中引用转换器

   ```xml
   <mvc:annotation-driven conversion-service="conversionService"/>
   ```

发送请求（参数必须为date）

```http
GET http://localhost:8080/user/quick18?date=2022-03-22
```

```java
@RequestMapping(value = "/quick18")
@ResponseBody
public void save18(Date date) throws IOException {
    System.out.println(date);
}
```



## 2.9 获得Servlet相关API

SpringMVC支持使用原始ServletAPI对象作为控制器方法的参数进行注入，常用的对象如下： 

- HttpServletRequest 
- HttpServletResponse
- HttpSession

```java
@RequestMapping(value = "/quick19")
@ResponseBody
public void save19(HttpServletRequest request, HttpServletResponse response, HttpSession session) throws IOException {
    System.out.println(request);
    System.out.println(response);
    System.out.println(session);
}
```

out：

```
org.apache.catalina.connector.RequestFacade@36f0729d
org.apache.catalina.connector.ResponseFacade@38379a33
org.apache.catalina.session.StandardSessionFacade@3c0819ed
```

## 2.10 获得请求头

**1）@RequestHeader**

使用`@RequestHeader`可以获得请求头信息，相当于web阶段学习的`request.getHeader(name)`

@RequestHeader注解的属性如下：

- value：请求头的名称
- required：是否必须携带此请求头

```java
@RequestMapping(value = "/quick20")
@ResponseBody
public void save20(@RequestHeader(value = "User-Agent", required = false) String user_agent) throws IOException {
    System.out.println(user_agent);
}
```

out:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36
```



**2）@CookieValue**

使用`@CookieValue`可以获得指定Cookie的值 

`@CookieValue`注解的属性如下： 

- value：指定cookie的名称
- required：是否必须携带此cookie

```java
@RequestMapping(value = "/quick21")
@ResponseBody
public void save21(@CookieValue(value = "JSESSIONID") String jsessionId) throws IOException {
    System.out.println(jsessionId);
}
```



## 2.11 文件上传

**文件上传客户端三要素**

- 表单项`type=“file”` 
- 表单的提交方式是post
- 表单的`enctype`属性是多部分表单形式，及`enctype=“multipart/form-data”`

![image-20220322225720423](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203222257472.png)



**文件上传原理**

- 当form表单修改为多部分表单时，`request.getParameter()`将失效。 
- `enctype="application/x-www-form-urlencoded"`时，form表单的正文内容格式是： **key=value&key=value&key=value**
- 当form表单的enctype取值为`Mutilpart/form-data`时，请求正文内容就变成多部分形式：

![image-20220322225923088](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203222259155.png)



## 2.12 单文件上传

① 导入fileupload和io坐标 

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.3</version>
</dependency>
```

② 配置文件上传解析器

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"> 
    <!--上传文件总大小-->
    <property name="maxUploadSize" value="5242800"/> 
    <!--上传单个文件的大小--> 
    <property name="maxUploadSizePerFile" value="5242800"/> 
    <!--上传文件的编码类型--> 
    <property name="defaultEncoding" value="UTF-8"/>
</bean>
```

③ 编写文件上传代码

```java
@RequestMapping(value = "/quick22")
@ResponseBody
public void save22(String username, MultipartFile uploadFile, MultipartFile uploadFile2) throws IOException {
    System.out.println(username);
    //获得上传文件的名称
    String originalFilename = uploadFile.getOriginalFilename();
    uploadFile.transferTo(new File("D:\\迅雷下载\\upload\\" + originalFilename));
    String originalFilename2 = uploadFile2.getOriginalFilename();
    uploadFile2.transferTo(new File("D:\\迅雷下载\\upload\\" + originalFilename2));
}
```



多文件上传，只需要将页面修改为多个文件上传项，将方法参数MultipartFile类型修改为`MultipartFile[]`即可

```jsp
<body>
    <h1>分别上传</h1>
    <form action="${pageContext.request.contextPath}/user/quick22" method="post" enctype="multipart/form-data">
        名称<input type="text" name="username"><br/>
        文件1<input type="file" name="uploadFile"><br/>
        文件2<input type="file" name="uploadFile2"><br/>
        <input type="submit" value="提交">
    </form>

    <h1>多文件上传</h1>
    <form action="${pageContext.request.contextPath}/user/quick23" method="post" enctype="multipart/form-data">
        名称<input type="text" name="username"><br/>
        文件1<input type="file" name="uploadFiles"><br/>
        文件2<input type="file" name="uploadFiles"><br/>
        <input type="submit" value="提交">
    </form>
</body>
```



```java
@RequestMapping(value = "/quick23")
@ResponseBody
public void save23(String username, MultipartFile[] uploadFiles) throws IOException {
    System.out.println(username);
    for (MultipartFile multipartFile : uploadFiles) {
        String originalFilename = multipartFile.getOriginalFilename();
        multipartFile.transferTo(new File("D:\\迅雷下载\\upload\\" + originalFilename));
    }
}
```

