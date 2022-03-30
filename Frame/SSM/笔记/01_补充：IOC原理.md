> **参考资料：**
>
> \[1] [我们到底为什么要用 IoC 和 AOP](https://www.cnblogs.com/RudeCrab/p/14365296.html#ioc)
>
> \[2] [IoC原理 - 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1282381977747489)
>
> \[3] [Spring IoC有什么好处呢？ - Mingqi的回答 - 知乎](https://www.zhihu.com/question/23277575/answer/169698662)



IoC全称Inversion of Control，直译为控制反转，是指对象的创建和配置的控制权从调用方转移给容器。

![](https://pic1.zhimg.com/80/v2-ee924f8693cff51785ad6637ac5b21c1_720w.jpg?source=1940ef5c)

# 1. 依赖倒置原则

**依赖倒置原则**——把原本的高层建筑依赖底层建筑“倒置”过来，变成底层建筑依赖高层建筑。高层建筑决定需要什么，底层去实现这样的需求，但是高层并不用管底层是怎么实现的。

先举一个现实生活中的例子。

假设设计一辆汽车：先设计轮子，然后根据轮子大小设计底盘，接着根据底盘设计车身，最后根据车身设计好整个汽车。这里就出现了一个“依赖”关系：汽车依赖车身，车身依赖底盘，底盘依赖轮子。

![](https://pic1.zhimg.com/80/v2-c68248bb5d9b4d64d22600571e996446_720w.jpg?source=1940ef5c)

这样的设计维护性很低。假设设计完工之后，根据市场需求的变动，把车子的轮子设计都改大一码。因为我们是根据轮子的尺寸设计的底盘，轮子的尺寸一改，底盘的设计就得修改；同样因为我们是根据底盘设计的车身，那么车身也得改，同理汽车设计也得改——整个设计几乎都得改！

换一种思路。我们先设计汽车的大概样子，然后根据汽车的样子来设计车身，根据车身来设计底盘，最后根据底盘来设计轮子。这时候，依赖关系就倒置过来了：轮子依赖底盘， 底盘依赖车身， 车身依赖汽车。

![](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301517271.png)

这时候，上司再说要改动轮子的设计，我们就只需要改动轮子的设计，而不需要动底盘，车身，汽车的设计了。

# 2. 传统的三层架构

回到代码中，三层架构是经典的开发模式，我们一般将视图控制、业务逻辑和数据库操作分别抽离出来单独形成一个类，这样各个职责就非常清晰且易于复用和维护。

大致代码如下：

**Controller**

```java
@WebServlet("/user")
public class UserServlet extends HttpServlet {
    // 用于执行业务逻辑的对象
    private UserService userService = new UserServiceImpl();
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...省略其他代码
            
        // 执行业务逻辑
        userService.doService();
        
        // ...返回页面视图
    }
}
```

**Service**

```java
public class UserServiceImpl implements UserService{
    // 用于操作数据库的对象
    private UserDao userDao = new UserDaoImpl();
    
    @Override
    public void doService() {
        // ...省略业务逻辑代码
            
        // 执行数据库操作
        userDao.doUpdate();
        
        // ...省略业务逻辑代码
    }
}
```

**DAO**

```java
public class UserDaoImpl implements UserDao{
    @Override
    public void doUpdate() {
        // ...省略JDBC代码
    }
}
```

上层依赖下层的抽象，代码就分为了三层：

![](https://img2020.cnblogs.com/blog/1496775/202102/1496775-20210203092605506-381644319.jpg)

一个 DAO 对象往往会被多个 Service 对象使用，一个 Service 对象往往也会被多个 Controller 对象使用。

然而目前的代码有一个比较大的问题，那就是只做到了**逻辑复用**，并没有做到**资源复用**。上层调用下一层时，必然会持有下一层的对象引用，即成员变量。目前我们每一个成员变量都会实例化一个对象，如下图所示：

![](https://img2020.cnblogs.com/news/1496775/202102/1496775-20210203092605740-460644962.jpg)

每一个链路都创建了同样的对象，造成了极大的资源浪费。本应多个 Controller 复用同一个 Service，多个 Service 复用同一个 DAO。现在变成了一个 Controller创建多个重复的 Service，多个 Service 又创建了多个重复的 DAO，从倒三角变成了正三角。许多组件只需要实例化一个对象就够了，创建多个没有任何意义。

除了资源浪费的问题之外，目前代码还有一个致命缺陷，那就是**变化的代价太大**。

假设有 10 个 Controller 依赖了 UserService，最开始实例化的是 `UserServiceImpl`，后面需要换一个实现类 `OtherUserServiceImpl`，我就得逐个修改那 10 个 Controller，非常麻烦。

其次，如果组件创建过程复杂，比如 DAO 对象要依赖一个这样的数据源组件：

```java
public class UserDaoImpl implements UserDao{
    private MyDataSource dataSource;

    public UserDaoImpl() {
        // 构造数据源
        dataSource = new MyDataSource("jdbc:mysql://localhost:3306/test", "root", "password");
        // 进行一些其他配置
        dataSource.setInitiaSize(10);
        dataSource.setMaxActive(100);
        // ...省略更多配置项
    }
}
```

该数据源组件要想真正生效需要对其进行许多配置，这个创建和配置过程是非常麻烦的。而且配置可能会随着业务需求的变化经常更改，这时候你就需要修改每一个依赖该组件的地方，牵一发而动全身。

**总结弊端：**

- 创建了许多重复对象，造成大量资源浪费；
- 更换实现类需要改动多个地方；
- 创建和配置组件工作繁杂，给组件调用方带来极大不便。

# 3. IoC

如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。

因此，核心问题是：

1. 谁负责创建组件？
2. 谁负责根据依赖关系组装组件？
3. 销毁时，如何按依赖顺序正确销毁？

换句话说，如果我们编码时，有一个「东西」能帮助我们创建和配置好那些组件，我们只负责调用该多好。

这个「东西」就是IoC容器。

有了 IoC 容器，我们可以将对象交由容器管理，交由容器管理后的对象称之为 Bean。调用方不再负责组件的创建，要使用组件时直接获取 Bean 即可。调用方只需按照约定声明依赖项，所需要的 Bean 就自动配置完毕了，就好像在调用方外部注入了一个依赖项给其使用，所以这种方式称之为 **依赖注入（Dependency Injection，缩写为 DI）**。

对象交由容器管理后，默认是单例的，这就解决了资源浪费问题。

若要更换实现类，只需更改 Bean 的声明配置，即可达到无感知更换：

```java
// 原本的实现类
public class UserServiceImpl implements UserService{
    ...
}

// 将该实现类声明为 Bean
@Component
public class OtherUserServiceImpl implements UserService{
    ...
}
```

共享一个组件也非常简单。

# 4. 依赖注入

在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在IoC容器中被“装配”出来，需要某种“注入”机制。

## 4.1 依赖注入的原理



**所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制**”。

以#1中的车辆设计为例，我们要如何实现基于依赖倒置的设计呢？

用**构造方法传递的依赖注入方式**重新写车类的定义：

![preview](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301555354.jpeg)

自底向上的创建对象，每一层中都不会创建对象，而是将底层对象作为构造器参数。即：

![](https://tangnameless-pic.oss-cn-beijing.aliyuncs.com/img/202203301558443.jpeg)

在这个例子中，**控制反转容器(IoC Container)**如何发挥作用呢？

![](https://pica.zhimg.com/80/v2-c845802f9187953ed576e0555f76da42_720w.jpg?source=1940ef5c)

因为采用了依赖注入，在车类初始化的过程中就不可避免的会写大量的new。这里IoC容器就解决了这个问题。**这个容器可以自动对你的代码进行初始化，你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码**。这是引入IoC Container的第一个好处。

IoC Container在进行这个工作的时候是反过来的，它先从最上层开始往下找依赖关系，到达最底层之后再往上一步一步new。

![](https://pica.zhimg.com/80/v2-24a96669241e81439c636e83976ba152_720w.jpg?source=1940ef5c)

这里IoC Container可以直接隐藏具体的创建实例的细节，在我们来看它就像一个工厂：

![](https://pic3.zhimg.com/80/v2-5ca61395f37cef73c7bbe7808f9ea219_720w.jpg?source=1940ef5c)

IoC Container的第二个好处是：**我们在创建实例的时候不需要了解其中的细节。**

## 4.2 Spring中的依赖注入方式

Spring的IoC容器同时支持属性注入和构造方法注入，并允许混合使用。

这里不再展开。

# 5. 无侵入容器

在设计上，Spring的IoC容器是一个高度可扩展的**无侵入容器**。

所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。

这种无侵入的设计有以下好处：

1. 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
2. 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率。

