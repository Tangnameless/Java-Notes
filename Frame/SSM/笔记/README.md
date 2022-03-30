# 1. 学习课程

**课程名称：**

黑马程序员最全SSM框架教程|Spring+SpringMVC+MyBatis全套教程(spring+springmvc+mybatis)

**课程链接：**

https://www.bilibili.com/video/BV1WZ4y1P7Bp

**课程所用软件资源下载：**

百度网盘

https://pan.baidu.com/s/1LxIxcHDO7SYB96SE-GZfuQ
提取码：dor4

# 2. 辅助参考教程

[廖雪峰：Spring开发](https://www.liaoxuefeng.com/wiki/1252599548343744/1266263217140032)

# 3. 主要环境

```
JDK 15
Spring 5.0.5.RELEASE
mysql 8
tomcat 9
maven 3.8
```

# 4. 课程中常见的错误

**可能出现的报错：**

**1）运行maven程序报错：java 不支持发行版本5**

```
Error : java 不支持发行版本5
```

解决方法：https://blog.csdn.net/qq_22076345/article/details/82392236

直接在pom.xml中加入

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>1.15</source>
                <target>1.15</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**2）pom.xml中正确设置坐标**

pom.xml中mysql-connector-java的版本要与电脑上安装的MySQL版本对应

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>
```

**3）jdbc.properties的设置**

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
jdbc.username=xxx
jdbc.password=xxx
```

MySQL6以后用的是`com.mysql.cj.jdbc.Driver`， JDBC连接MySQL6 （com.mysql.cj.jdbc.Driver），需要指定时区serverTimezone。mysql数据库的用户名和密码要正确设置。



**其它问题：**

1）评论区反应第六节的练习中，jsp页面图片不显示，需要在jsp头部加入`<%@ page isELIgnored="false" %>`，实测web.xml 4.0版本没有出现图片不显示的问题。



2）第七节SpringMVC拦截器部分，老师忽略了一个问题。登陆页面使用的图片和样式资源也会被拦截，因为浏览器有缓存，视频中未触发这个问题。应该修改 spring-mvc.xml中的配置为：

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
