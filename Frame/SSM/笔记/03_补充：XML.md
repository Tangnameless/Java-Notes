**参考资料:**

https://www.w3school.com.cn/xml/xml_namespaces.asp

https://www.runoob.com/schema/schema-intro.html

https://developer.aliyun.com/article/40353

# 1. XML 命名空间（XML Namespaces）

**XML 命名空间提供避免元素命名冲突的方法。**

## 命名冲突

在 XML 中，元素名称是由开发者定义的，当两个不同的文档使用相同的元素名时，就会发生命名冲突。

## 使用前缀来避免命名冲突

此文档带有某个表格中的信息：

```xml
<h:table>
   <h:tr>
   <h:td>Apples</h:td>
   <h:td>Bananas</h:td>
   </h:tr>
</h:table>
```

此 XML 文档携带着有关一件家具的信息：

```xml
<f:table>
   <f:name>African Coffee Table</f:name>
   <f:width>80</f:width>
   <f:length>120</f:length>
</f:table>
```

现在，命名冲突不存在了，这是由于两个文档都使用了不同的名称来命名它们的 `<table> `元素 (<h:table> 和 <f:table>)。

通过使用前缀，我们创建了两种不同类型的 `<table> `元素。

## 使用命名空间（Namespaces）

这个 XML 文档携带着某个表格中的信息：

```xml
<h:table xmlns:h="http://www.w3.org/TR/html4/">
   <h:tr>
   <h:td>Apples</h:td>
   <h:td>Bananas</h:td>
   </h:tr>
</h:table>
```

此 XML 文档携带着有关一件家具的信息：

```xml
<f:table xmlns:f="http://www.w3school.com.cn/furniture">
   <f:name>African Coffee Table</f:name>
   <f:width>80</f:width>
   <f:length>120</f:length>
</f:table>
```

与仅仅使用前缀不同，我们为 `<table> `标签添加了一个 xmlns 属性，这样就为前缀==赋予了一个与某个命名空间相关联的限定名称==。

## XML Namespace (xmlns) 属性

XML 命名空间属性被放置于元素的开始标签之中，并使用以下的语法：

```
xmlns:namespace-prefix="namespaceURI"
```

当命名空间被定义在元素的开始标签中时，所有带有相同前缀的子元素都会与同一个命名空间相关联。

**注释：**用于标示命名空间的地址不会被解析器用于查找信息。其惟一的作用是赋予命名空间一个惟一的名称。不过，很多公司常常会作为指针来使用命名空间指向实际存在的网页，这个网页包含关于命名空间的信息。

## 默认的命名空间（Default Namespaces）

为元素定义默认的命名空间可以让我们省去在所有的子元素中使用前缀的工作。

请使用下面的语法：

```
xmlns="namespaceURI"
```



# 2. XML Schema (简单了解)

XML Schema 是基于 XML 的 DTD 替代者。

==XML Schema 可描述 XML 文档的结构。==

XML Schema 语言也可作为 XSD（XML Schema Definition）来引用。

一个XML Schema文档通常称之为模式文档（约束文档），遵循这个文档书写的xml文件称之为实例文档。

编写了一个XML Schema约束文档后，通常需要把这个文件中声明的元素绑定到一个URI地址上，在XML Schema技术中有一个专业术语来描述这个过程，即把XML Schema文档声明的元素绑定到一个命名空间上，以后XML文件就可以通过这个URI（即命名空间）来告诉解析引擎，XML文档中编写的元素来自哪里，被谁约束。为了在一个XML文档中声明它所遵循的Schema文件具体位置，通常需要在XML文档中的根节点中使用schemaLocation属性来指定。

## 对 XML Schema 的引用

此文件包含对 XML Schema 的引用：

名为 "note.xsd" 的 XML Schema 文件定义了XML 文档（ "note.xml" ）的元素

```xml
<?xml version="1.0"?>

<note xmlns="http://www.runoob.com"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.runoob.com note.xsd">

<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

```xml
xmlns="http://www.runoob.com"
```

规定了默认命名空间的声明。此声明会告知 schema 验证器，在此 XML 文档中使用的所有元素都被声明于 "http://www.runoob.com" 这个命名空间。

```xml
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance
```

显示 schema 中用到的元素和数据类型来自命名空间`"http://www.w3.org/2001/XMLSchema-instance"`, 同时它还规定了来自命名空间 `"http://www.w3.org/2001/XMLSchema-instance"` 的元素和数据类型应该使用命名空间前缀 `xsi:`

> 这个 `xmlns:xsi`在不同的 xml 文档中似乎都会出现。 这是因为，xsi 已经成为了一个业界默认的用于 XSD(（XML Schema Definition) 文件的命名空间。

schemaLocation 属性有两个值。第一个值是需要使用的命名空间。第二个值是供命名空间使用的 XML schema 的位置：

```xml
xsi:schemaLocation = "键" "值"
```

例如:

```xml
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
```

前一个"键" `http://maven.apache.org/POM/4.0.0` 指代 【命名空间】， 只是一个全局唯一字符串而已。

后一个值指代 【XSD location URI】 , 这个值指示了前一个命名空间所对应的 XSD 文件的位置， xml parser 可以利用这个信息获取到 XSD 文件， 从而通过 XSD 文件对所有属于 命名空间 `http://maven.apache.org/POM/4.0.0` 的元素结构进行校验， 因此这个值必然是可以访问的， 且访问到的内容是一个 XSD 文件的内容。

# 3. Spring中的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context 
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/mvc
                           http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="xxx.xxx.controller" />

    <context:annotation-config/>

    <mvc:default-servlet-handler/>

    <mvc:annotation-driven/>

    <mvc:resources mapping="/images/**" location="/images/" />

    <bean id="xxx" class="xxx.xxx.xxx.Xxx">
        <property name="xxx" value="xxxx"/>
    </bean>

</beans>
```



```xml
xmlns="http://www.springframework.org/schema/beans"
```

这一句表示该文档默认的XML Namespace为`http://www.springframwork.org/schema/beans`。**对于默认的Namespace中的元素，可以不使用前缀**。

`xsi:schemaLocation`属性其实是Namespace为`http://www.w3.org/2001/XMLSchema-instance`里的schemaLocation属性，正是因为我们一开始声明了：

```xml
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
```

这里才写作`xsi:schemaLocation`（当然一般都使用这个前缀）。

它定义了XML Namespace和对应的XSD（Xml Schema Definition）文档的位置的关系。

例如：

```xml
xsi:schemaLocation="http://www.springframework.org/schema/context"         
                   "http://www.springframework.org/schema/context/spring-context.xsd"
```

这里表示Namespace为`http://www.springframework.org/schema/context`的Schema的位置为`http://www.springframework.org/schema/context/spring-context.xsd`。

