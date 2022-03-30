# 1. Spring的AOP简介

## 1.1 什么是AOP?

AOP 为 Aspect Oriented Programming 的缩写，意思为面向切面编程，是通过预编译方式和运行期动态代理 实现程序功能的统一维护的一种技术。
AOP 是 OOP 的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。



**OOP 的局限性**

当有重复代码出现时，可以就将其封装出来然后复用。我们通过分层、分包、分类来规划不同的逻辑和职责。但这里的复用的都是**核心业务逻辑**，并不能复用一些**辅助逻辑**，比如：日志记录、性能统计、安全校验、事务管理，等等。这些边缘逻辑往往贯穿你整个核心业务，传统 OOP 很难将其封装：

![](https://img2020.cnblogs.com/blog/1496775/202102/1496775-20210203092605947-1239661656.jpg)

这一条条横线仿佛切开了 OOP 的树状结构，犹如一个大蛋糕被切开多层，每一层都会执行相同的辅助逻辑，所以大家将这些辅助逻辑称为层面或者切面。

## 1.2 AOP 的作用及其优势

作用：在程序运行期间，在不修改源码的情况下对方法进行功能增强

优势：减少重复代码，提高开发效率，并且便于维护

## 1.3 AOP 的底层实现

AOP 的底层是通过 Spring 提供的的动态代理技术实现的。在运行期间，Spring通过动态代理技术动态的生成代理对象，代理对象方法执行时进行增强功能的介入，在去调用目标对象的方法，从而完成功能的增强。

> 关于动态代理，见：[09_补充：代理模式](D:\BaiduNetdiskWorkspace\Md笔记\Java后端开发\Java框架\SSM\09_补充：代理模式.md)

## 1.4 AOP的动态代理技术

常用的动态代理技术 

- JDK 代理 : 基于接口的动态代理技术
- cglib 代理：基于父类的动态代理技术



![image-20220325155728208](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203251557269.png)

## 1.5 JDK的动态代理

1）目标类接口

```java
public interface TargetInterface {
    public void save();
}
```

2）目标类

```java
public class Target implements TargetInterface {
    public void save() {
        System.out.println("save running.....");
    }
}
```

3）动态代理代码

```java
public class ProxyTest {

    public static void main(String[] args) {
        //目标对象
        final Target target = new Target();
        //增强对象
        final Advice advice = new Advice();
        //返回值 就是动态生成的代理对象
        TargetInterface proxy = (TargetInterface) Proxy.newProxyInstance(
            	//目标对象类加载器
                target.getClass().getClassLoader(), 
            	//目标对象相同的接口字节码对象数组
                target.getClass().getInterfaces(), 
                new InvocationHandler() {
                    //调用代理对象的任何方法  实质执行的都是invoke方法
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //前置增强
                        advice.before();
                        //执行目标方法
                        Object invoke = method.invoke(target, args);
                        //后置增强
                        advice.afterReturning();
                        return invoke;
                    }
                }
        );
        //调用代理对象的方法
        proxy.save();
    }
}
```

4）调用代理对象的方法测试

```java
// 测试,当调用接口的任何方法时，代理对象的代码都无需修改 
proxy.save();
```



## 1.6 cglib 的动态代理（原理未学习）

1）目标类

```java
public class Target {
    public void save() {
        System.out.println("save running.....");
    }
}
```

2）增强类对象

```java
public class Advice {

    public void before(){
        System.out.println("前置增强....");
    }

    public void afterReturning(){
        System.out.println("后置增强....");
    }

}
```

3）动态代理代码

```java
public class ProxyTest {

    public static void main(String[] args) {

        //目标对象
        final Target target = new Target();

        //增强对象
        final Advice advice = new Advice();

        //返回值 就是动态生成的代理对象  基于cglib
        //1、创建增强器
        Enhancer enhancer = new Enhancer();
        //2、设置父类（目标）
        enhancer.setSuperclass(Target.class);
        //3、设置回调
        enhancer.setCallback(new MethodInterceptor() {
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                advice.before(); //执行前置
                Object invoke = method.invoke(target, args);//执行目标
                advice.afterReturning(); //执行后置
                return invoke;
            }
        });
        //4、创建代理对象
        Target proxy = (Target) enhancer.create();

        proxy.save();
    }
}
```

## 1.7 AOP的相关概念

Spring 的 AOP 实现底层就是对上面的动态代理的代码进行了封装，封装后我们只需要对需要关注的部分进行代码编 写，并通过配置的方式完成指定目标的方法增强。
AOP 常用的相关术语如下：

- **Target（目标对象）**：代理的目标对象 
- **Proxy （代理）**：一个类被 AOP 织入增强后，就产生一个结果代理类 
- **Joinpoint（连接点）**：所谓连接点是指那些被拦截到的点。在spring中,这些点指的是方法，因为spring只支持方 法类型的连接点
- **Pointcut（切入点）**：所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义 
- **Advice（通知/ 增强）**：所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知 
- **Aspect（切面）**：是切入点和通知（引介）的结合 
- **Weaving（织入）**：是指把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入





## 1.8 AOP 开发明确的事项 

1. 需要编写的内容
- 编写核心业务代码（目标类的目标方法） 
- 编写切面类，切面类中有通知(增强功能方法) 
- 在配置文件中，配置织入关系，即将哪些通知与哪些连接点进行结合

2. AOP 技术实现的内容
Spring 框架监控切入点方法的执行。一旦监控到切入点方法被运行，使用代理机制，动态创建目标对象的 代理对象，根据通知类别，在代理对象的对应位置，将通知对应的功能织入，完成完整的代码逻辑运行。

3. AOP 底层使用哪种代理方式
在 spring 中，框架会根据目标类是否实现了接口来决定采用哪种动态代理的方式。

# 2. 基于XML的AOP开发

## 2.1 快速入门

**步骤：**

1、导入 AOP 相关坐标 

2、创建目标接口和目标类（内部有切点） 

3、创建切面类（内部有增强方法） 

4、将目标类和切面类的对象创建权交给 spring 

5、在 applicationContext.xml 中配置织入关系

6、测试代码



**1、导入AOP相关坐标**

```xml
<!--导入spring的context坐标，context依赖aop-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<!-- aspectj的织入 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.4</version>
</dependency>
```

**2、创建目标接口和目标类（内部有切点）** 

接口

```java
public interface TargetInterface {
    public void save();
}
```

目标类

```java
public class Target implements TargetInterface {
    public void save() {
        System.out.println("save running.....");
    }
}
```

**3、创建切面类（内部有增强方法）** 

```java
public class MyAspect {

    public void before() {
        System.out.println("前置增强..........");
    }

    public void afterReturning() {
        System.out.println("后置增强..........");
    }

    //Proceeding JoinPoint:  正在执行的连接点===切点
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前增强....");
        Object proceed = pjp.proceed();//切点方法
        System.out.println("环绕后增强....");
        return proceed;
    }

    public void afterThrowing() {
        System.out.println("异常抛出增强..........");
    }

    public void after() {
        System.out.println("最终增强..........");
    }

}
```

**4、将目标类和切面类的对象创建权交给 spring** 

在`applicationContext.xml`中配置Bean

```xml
<!--配置目标类-->
<bean id="target" class="com.itheima.aop.Target"></bean>

<!--配置切面类-->
<bean id="myAspect" class="com.itheima.aop.MyAspect"></bean>
```

**5、在 applicationContext.xml 中配置织入关系**

导入aop命名空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
">
```

配置切点表达式和前置增强的织入关系：

```xml
    <!--配置织入：告诉spring框架 哪些方法(切点)需要进行哪些增强(前置、后置...)-->
    <aop:config>
        <!--声明切面-->
        <aop:aspect ref="myAspect">
		   <!--配置Target的save方法执行时要进行myAspect的before方法前置增强--> 
            <aop:before method="before" pointcut="execution(public void com.itheima.aop.Target.save())"/>
        </aop:aspect>
    </aop:config>
```

**6、测试代码**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AopTest {

    @Autowired
    private TargetInterface target;

    @Test
    public void test1() {
        target.save();
    }   
}
```

## 2.2 XML 配置 AOP 详解

### 1）切点表达式的写法

 表达式语法：

```
execution([修饰符] 返回值类型 包名.类名.方法名(参数))
```

- 访问修饰符可以省略 
- 返回值类型、包名、类名、方法名可以使用星号* 代表任意 
- 包名与类名之间一个点 `.` 代表当前包下的类，两个点 `..` 表示当前包及其子包下的类
- 参数列表可以使用两个点 `..` 表示任意个数，任意类型的参数列表

```shell
# 全
execution(public void com.itheima.aop.Target.method()) 
# 任意方法任意参数
execution(void com.itheima.aop.Target.*(..)) 
# 任意返回值com.itheima.aop包下任意类任意方法任意参数
execution(* com.itheima.aop.*.*(..)) 
# 你故意找茬是吧
execution(* *..*.*(..))
```

### 2）通知的类型

通知的配置语法：

```xml
<aop:通知类型 method="切面类中方法名" pointcut="切点表达式"></aop:通知类型>
```



| 名称         | 标签                    | 说明                                                         |
| ------------ | ----------------------- | ------------------------------------------------------------ |
| 前置通知     | `<aop:before>`          | 用于配置前置通知。指定增强的方法在切入点方法之前执行         |
| 后置通知     | `<aop:after-returning>` | 用于配置后置通知。指定增强的方法在切入点方法之后执行         |
| 环绕通知     | `<aop:around>`          |   于配置环绕通知。指定增强的方法在切入点方法之前和之后都 执行                                                           |
| 异常抛出通知 | `<aop:throwing>`        |   用用于配置异常抛出通知。指定增强的方法在出现异常时执行                                                           |
| 最终通知     | `<aop:after>`           | 用于配置最终通知。无论增强方式执行是否有异常都会执行 |

### 3）切点表达式的抽取

当多个增强的切点表达式相同时，可以将切点表达式进行抽取，在增强中使用 `pointcut-ref` 属性代替 `pointcut` 属性来引用抽取后的切点表达式。

```xml
<aop:config>
    <!--引用myAspect的Bean为切面对象-->
    <aop:aspect ref="myAspect">
        <!--抽取切点表达式-->
        <aop:pointcut id="myPointcut" expression="execution(* com.itheima.aop.*.*(..))"></aop:pointcut>
        <aop:around method="around" pointcut-ref="myPointcut"/>
        <aop:after method="after" pointcut-ref="myPointcut"/>
    </aop:aspect>
</aop:config>
```

通知方法存在于切面类中，根据切点表达式，对切点增强。

# 3. 基于注解的AOP开发

## 3.1 快速入门 

**步骤：** 
1、创建目标接口和目标类（内部有切点） 
2、创建切面类（内部有增强方法） 
3、将目标类和切面类的对象创建权交给 spring 
4、在切面类中使用注解配置织入关系 
5、在配置文件中开启组件扫描和 AOP 的自动代理
6、测试



**1、创建目标接口和目标类（内部有切点）** 

```java
public interface TargetInterface {
    public void save();
}
```

```java
public class Target implements TargetInterface {
    public void save() {
        System.out.println("save running.....");
    }
}
```

**2、创建切面类（内部有增强方法）** 

```java
public class MyAspect { 
    //前置增强方法 
    public void before(){ 
        System.out.println("前置代码增强.....");
	}
}
```

**3、将目标类和切面类的对象创建权交给 spring** 

配置相应注解 `@Component`

**4、在切面类中使用注解配置织入关系**

```java
@Component("myAspect")
@Aspect //标注当前MyAspect是一个切面类
public class MyAspect {
    //配置前置通知
    @Before("execution(* com.itheima.anno.*.*(..))")
    public void before(){
        System.out.println("前置增强..........");
    }
}
```

 **5、在配置文件中开启组件扫描和 AOP 的自动代理**

```xml
<!--组件扫描-->
<context:component-scan base-package="com.itheima.anno"/>

<!--aop自动代理-->
<aop:aspectj-autoproxy/>
```

**6、测试**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-anno.xml")
public class AnnoTest {

    @Autowired
    private TargetInterface target;

    @Test
    public void test1() {
        target.save();
    }
}
```

## 3.2 注解配置 AOP 详解

### 1）注解通知的类型

通知的配置语法：`@通知注解("切点表达式")`

| 名称         | 注解             | 说明                                                         |
| ------------ | ---------------- | ------------------------------------------------------------ |
| 前置通知     | `@Before`        | 用于配置前置通知。指定增强的方法在切入点方法之前执行         |
| 后置通知     | `@AfterReturning` | 用于配置后置通知。指定增强的方法在切入点方法之后执行         |
| 环绕通知     | `@Around`         | 用于配置环绕通知。指定增强的方法在切入点方法之前和之后都执行 |
| 异常抛出通知 | `@AfterThrowing`  | 用于配置异常抛出通知。指定增强的方法在出现异常时执行         |
| 最终通知     | `@After`          | 用于配置最终通知。无论增强方式执行是否有异常都会执行         |

### 2）切点表达式的抽取

通xml 配置 aop 一样，我们可以将切点表达式抽取。**抽取方式是在切面内定义方法**，在该方法上使用`@Pointcut` 注解定义切点表达式，然后在在增强注解中进行引用。具体如下：

```java
@Component("myAspect")
@Aspect //标注当前MyAspect是一个切面类
public class MyAspect {

    //Proceeding JoinPoint:  正在执行的连接点===切点
    //@Around("execution(* com.itheima.anno.*.*(..))")
    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前增强....");
        Object proceed = pjp.proceed();//切点方法
        System.out.println("环绕后增强....");
        return proceed;
    }

    public void afterThrowing() {
        System.out.println("异常抛出增强..........");
    }

    //@After("execution(* com.itheima.anno.*.*(..))")
    @After("MyAspect.pointcut()")
    public void after() {
        System.out.println("最终增强..........");
    }

    //定义切点表达式
    @Pointcut("execution(* com.itheima.anno.*.*(..))")
    public void pointcut() {
    }
}
```



**疑问：**关于`@After("MyAspect.pointcut()")`一定会抛出的问题，如何解决？

