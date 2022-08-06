
> 黑马SSM框架学习笔记



# 前言- [前言](#前言)
- [前言- 前言](#前言--前言)
- [一、Spring](#一spring)
  - [1）IoC控制反转](#1ioc控制反转)
  - [2）AOP面向切面编程](#2aop面向切面编程)
    - [基本概念](#基本概念)
    - [AOP的实现](#aop的实现)
- [二、SpringMVC](#二springmvc)
  - [1）Spring集成Web环境](#1spring集成web环境)
  - [2）SpringMVC简介](#2springmvc简介)
    - [快速入门](#快速入门)
    - [SpringMVC 执行流程](#springmvc-执行流程)
    - [组件解析](#组件解析)
  - [3）SpringMVC的请求和响应](#3springmvc的请求和响应)
    - [数据响应](#数据响应)
    - [请求数据](#请求数据)
    - [细节](#细节)
  - [4）SpringMVC拦截器](#4springmvc拦截器)
  - [5）SpringMVC的异常处理](#5springmvc的异常处理)
    - [使用SpringMVC提供的简单异常处理器SimpleMappingExceptionResolver](#使用springmvc提供的简单异常处理器simplemappingexceptionresolver)
    - [自定义异常处理](#自定义异常处理)
- [三、MyBatis](#三mybatis)
  - [1）简介](#1简介)
  - [2）增删改查操作](#2增删改查操作)
  - [3）核心配置文件解析](#3核心配置文件解析)
  - [4）代理开发](#4代理开发)
  - [5）注解开发](#5注解开发)
   
---
`主要记录一些SSM框架的知识点`

Java SSM框架即指Spring+SpringMVC+MyBatis的简称，框架集由Spring、MyBatis两个开源框架整合而成（SpringMVC是Spring中的部分内容）,常作为数据源较简单的web项目的框架。


---



# 一、Spring
 Spring是一个开源框架，Spring是于2003 年兴起的一个轻量级的Java开发框架，由Rod Johnson 在其著作Expert One-On-One J2EE Development and Design中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。 简单来说，Spring是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架。
 
 ## 1）IoC控制反转
 传统的创建对象的方法是直接通过 new 关键字，而 spring 则是通过 IOC 容器来创建对象，也就是说我们将创建对象的控制权交给了 IOC 容器。我们可以用一句话来概括 IOC：
**IOC 让程序员不在关注怎么去创建对象，而是关注与对象创建之后的操作，把对象的创建、初始化、销毁等工作交给spring容器来做。**
 
 Spring创建容器的三种方式：
 1、利用默认的构造方法，在 src 目录下新建 applicationContext.xml 文件，在 Spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建。
```xml
<!--
    创建对象的第一种方式：利用无参构造器
    id:唯一标识符
    class:类的全类名
      -->
    <bean id="helloIoc" class="com.ys.ioc.HelloIoc" ></bean>  
    <!-- 别名属性  name：和bean的 id 属性对应 -->
    <alias name="helloIoc" alias="helloIoc2"/>
```
2、利用静态方法创建，首先创建静态工厂类 HelloStaticFactory.java，接着在 applicationContext.xml 中进行配置。
```xml
<!--
        创建对象的第二种方式：利用静态工厂方法
        factory-method：静态工厂类的获取对象的静态方法
        class:静态工厂类的全类名
      -->   
    <bean id="helloStaticFactory" factory-method="getInstances" class="com.ys.ioc.HelloStaticFactory"></bean>
```
3、利用实例工厂方法，首先创建实例工厂类 HelloInstanceFactory .java，然后接着在 applicationContext.xml 中进行如下配置：
```xml
<!--
        创建对象的第三种方式：利用实例工厂方法
        factory-bean:指定当前Spring中包含工厂方法的beanID
        factory-method:工厂方法名称
      --> 
      <bean id="instanceFactory" class="com.ys.ioc.HelloInstanceFactory"></bean> 
    <bean id="instance" factory-bean="instanceFactory" factory-method="getInstance"></bean> 
```
Spring容器的生命周期：1、spring容器创建对象 2、执行init方法 3、调用自己的方法 4、当spring容器关闭的时候执行destroy方法。

## 2）AOP面向切面编程
AOP （Aspect Orient Programming）,直译过来就是 面向切面编程。AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。

OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP技术恰恰相反，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

### 基本概念

：
1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、切面（aspect）

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

4、切入点（pointcut）

对连接点进行拦截的定义

5、通知（advice）

所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

6、目标对象

代理的目标对象

7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）

在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段

SpringAOP中的代理技术
![SpringAOP中的代理技术](https://img-blog.csdnimg.cn/b2bf287f78b2493d9a01cd33808617b3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6649aa2737794637a6d29a1eef50728e.png)

Spring AOP 按照通知（Advice）织入到目标类方法的连接点位置，为 Advice 接口提供了 6 个子接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/8ab4ee55c4eb479d96ba6f36c3227d85.png)
### AOP的实现
**a）XML实现方式**
首先，定义一个接口：

```java
public interface HelloWorld
{
    void printHelloWorld();
    void doPrint();
}
```
定义两个实现类

```java
public class HelloWorldImpl1 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl1.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl1.doPrint()");
        return ;
    }
}
```

```java
public class HelloWorldImpl2 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl2.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl2.doPrint()");
        return ;
    }
}
```
切面类定义：

```java
public class TimeHandler
{
    public void printTime()
    {
        System.out.println("CurrentTime = " + System.currentTimeMillis());
    }
}
```
xml配置织入关系，实现SpringAOP

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
        
        <bean id="helloWorldImpl1" class="com.xrq.aop.HelloWorldImpl1" />
        <bean id="helloWorldImpl2" class="com.xrq.aop.HelloWorldImpl2" />
        <bean id="timeHandler" class="com.xrq.aop.TimeHandler" />
        
        <aop:config>
            <aop:aspect id="time" ref="timeHandler">
                <aop:pointcut id="addAllMethod" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />
                <aop:before method="printTime" pointcut-ref="addAllMethod" />
                <aop:after method="printTime" pointcut-ref="addAllMethod" />
            </aop:aspect>
        </aop:config>
</beans>
```
**XML配置AOP详解：**
切点表达式：**execution([修饰符] 返回值类型 包名.类名.方法名(参数))**
 - 访问修饰符可以省略 
 - 返回值类型、包名、类名、方法名可以使用星号* 代表任意 
 - 包名与类名之间一个点 . 代表当前包下的类，两个点 ..
 - 表示当前包及其子包下的类 参数列表可以使用两个点 .. 表示任意个数，任意类型的参数列表

通知的类型： **<aop:通知类型 method=“切面类中方法名” pointcut=“切点表达式"></aop:通知类型>**
![在这里插入图片描述](https://img-blog.csdnimg.cn/cbb687d387fb4820b3c407f7804f9779.png)
**b)基于注解开发**
基于注解的aop开发步骤：
① 创建目标接口和目标类（内部有切点）
② 创建切面类（内部有增强方法）
③ 将目标类和切面类的对象创建权交给 spring
④ 在切面类中使用注解配置织入关系
⑤ 在配置文件中开启组件扫描和 AOP 的自动代理
⑥ 测试
通知的配置语法： **@通知注解(“切点表达式")**
![在这里插入图片描述](https://img-blog.csdnimg.cn/11088a250d95475b89f2586ff789ccdb.png)
抽取方式是在切面内定义方法，在该方法上使用 **@Pointcut注解**定义切点表达式，然后在在增强注解中进行引用。

```java
@Component("myAspect")
@Aspect
public class MyAspect {
	@Before("MyAspect.myPoint()")
	public void before(){
		System.out.println("前置代码增强.....");
	} 
	@Pointcut("execution(* com.itheima.aop.*.*(..))")
	public void myPoint(){}
}
```



# 二、SpringMVC
## 1）Spring集成Web环境
应用上下文对象是通过new ClasspathXmlApplicationContext(spring配置文件) 方式获取的，但是每次从
容器中获得Bean时都要编写new ClasspathXmlApplicationContext(spring配置文件) ，这样的弊端是配置
文件加载多次，应用上下文对象创建多次。在Web项目中，可以使用ServletContextListener监听Web应用的启动，我们可以在Web应用启动时，就加载Spring的配置文件，创建应用上下文对象ApplicationContext，在将其存储到最大的域servletContext域中，这样就可以在任意位置从域中获得应用上下文ApplicationContext对象了。
主要步骤：
① 在web.xml中配置ContextLoaderListener监听器（导入spring-web坐标）
② 使用WebApplicationContextUtils获得应用上下文对象ApplicationContext

```xml
<!--全局参数-->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--Spring的监听器-->
<listener>
	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
```
## 2）SpringMVC简介

> **MVC模式：** M即model模型是指模型表示业务规则。在MVC的三个部件中，模型拥有最多的处理任务。被模型返回的数据是中立的，模型与数据格式无关，这样一个模型能为多个视图提供数据，由于应用于模型的代码只需写一次就可以被多个视图重用，所以减少了代码的重复性。
> 
> V即View视图是指用户看到并与之交互的界面。比如由html元素组成的网页界面，或者软件的客户端界面。MVC的好处之一在于它能为应用程序处理很多不同的视图。在视图中其实没有真正的处理发生，它只是作为一种输出数据并允许用户操作的方式。
> 
> C即controller控制器是指控制器接受用户的输入并调用模型和视图去完成用户的需求，控制器本身不输出任何东西和做任何处理。它只是接收请求并决定调用哪个模型构件去处理请求，然后再确定用哪个视图来显示返回的数据。

SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于SpringFrameWork 的后续产品。SpringMVC 已经成为目前最主流的MVC框架之一，并且随着Spring3.0 的发布，全面超越 Struts2，成为最优秀的 MVC 框架。它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口。同时它还支持 RESTful 编程风格的请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/37cbc2a8ed6d40d29ec9237f4d83c091.png)
### 快速入门
SpringMVC的开发步骤
① 导入SpringMVC相关坐标
② 配置SpringMVC核心控制器DispathcerServlet
③ 创建Controller类和视图页面
④ 使用注解配置Controller类中业务方法的映射地址
⑤ 配置SpringMVC核心文件 spring-mvc.xml
⑥ 客户端发起请求测试
一个例子：
1、导入坐标（pom.xml）：
```xml
<!--Spring坐标-->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>5.0.5.RELEASE</version>
</dependency>
<!--SpringMVC坐标-->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>5.0.5.RELEASE</version>
</dependency>
<!--Servlet坐标-->
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>servlet-api</artifactId>
	<version>2.5</version>
</dependency>
<!--Jsp坐标-->
<dependency>
	<groupId>javax.servlet.jsp</groupId>
	<artifactId>jsp-api</artifactId>
	<version>2.0</version>
</dependency>
```
2、在web.xml配置SpringMVC的核心控制器
```xml
<servlet>
	<!--前端控制器DispatcherServlet-->
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
3、创建Controller和业务方法

```java
public class UserController {
    public String save(){
        System.out.println("Controller save running...");
        return "success.jsp";
    }
}

```
4、创建页面视图success.jsp:

```html
<html>
<body>
    <h1>Success!</h1>
</body>
</html>
```
5、配置注解：

```java
@Controller
@RequestMapping("/user")
public class UserController {
    //请求地址：http://localhost:8080/user/quick
    @RequestMapping("/quick")
    public String save(){
        System.out.println("Controller save running...");
        return "/success.jsp";//加/代表从web路径下加载资源，如果不加/代表从/user目录下加载资源，会报错
    }
}

```
6、创建spring-mvc.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!--Controller的组件扫描-->
    <context:component-scan base-package="com.itheima.controller"></context:component-scan>
</beans>

```
7、访问测试地址：***http://localhost:8080/quick***
### SpringMVC 执行流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/529847ef7b714106b0ad82d664286fb6.png)
① 用户发送请求至前端控制器DispatcherServlet。
② DispatcherServlet收到请求调用HandlerMapping处理器映射器。
③ 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果
有则生成)一并返回给DispatcherServlet。
④ DispatcherServlet调用HandlerAdapter处理器适配器。
⑤ HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
⑥ Controller执行完成返回ModelAndView。
⑦ HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
⑧ DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
⑨ ViewReslover解析后返回具体View。
⑩ DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。 DispatcherServlet响应用户。

### 组件解析
【1. HandlerMapping】

是用来查找Handler的。在SpringMVC中会有很多请求，每个请求都需要一个Handler处理，具体接收到一个请求之后使用哪个Handler进行处理呢？这就是HandlerMapping需要做的事。

【2. HandlerAdapter】

从名字上看，它就是一个适配器。因为SpringMVC中的Handler可以是任意的形式，只要能处理请求就ok，但是Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情。

小结：Handler是用来干活的工具；HandlerMapping用于根据需要干的活找到相应的工具；HandlerAdapter是使用工具干活的人。

【3. HandlerExceptionResolver】

其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？这就需要有一个专门的角色对异常情况进行处理，在SpringMVC中就是HandlerExceptionResolver。具体来说，此组件的作用是根据异常设置ModelAndView，之后再交给render方法进行渲染。

【4. ViewResolver】

ViewResolver用来将String类型的视图名和Locale解析为View类型的视图。View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。这里就有两个关键问题：使用哪个模板？用什么技术（规则）填入参数？这其实是ViewResolver主要要做的工作，ViewResolver需要找到渲染所用的模板和所用的技术（也就是视图的类型）进行渲染，具体的渲染过程则交由不同的视图自己完成。

【5. RequestToViewNameTranslator】

ViewName是根据ViewName查找View，但有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了，如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了。RequestToViewNameTranslator在Spring MVC容器里只可以配置一个，所以所有request到ViewName的转换规则都要在一个Translator里面全部实现。

【6. LocaleResolver】

解析视图需要两个参数：一是视图名，另一个是Locale。视图名是处理器返回的，Locale是从哪里来的？这就是LocaleResolver要做的事情。LocaleResolver用于从request解析出Locale，Locale就是zh-cn之类，表示一个区域，有了这个就可以对不同区域的用户显示不同的结果。SpringMVC主要有两个地方用到了Locale：一是ViewResolver视图解析的时候；二是用到国际化资源或者主题的时候。

【7. ThemeResolver】

用于解析主题。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、如图片、css样式等。SpringMVC的主题也支持国际化，同一个主题不同区域也可以显示不同的风格。SpringMVC中跟主题相关的类有 ThemeResolver、ThemeSource和Theme。主题是通过一系列资源来具体体现的，要得到一个主题的资源，首先要得到资源的名称，这是ThemeResolver的工作。然后通过主题名称找到对应的主题（可以理解为一个配置）文件，这是ThemeSource的工作。最后从主题中获取资源就可以了。

【8. MultipartResolver】

用于处理上传请求。处理方法是将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File，如果上传多个文件，还可以调用getFileMap得到FileName->File结构的Map。此组件中一共有三个方法，作用分别是判断是不是上传请求，将request包装成MultipartHttpServletRequest、处理完后清理上传过程中产生的临时资源。

【9. FlashMapManager】

用来管理FlashMap的，FlashMap主要用在redirect中传递参数。
## 3）SpringMVC的请求和响应
### 数据响应

1、 页面跳转  
直接返回字符串

```java
    @RequestMapping(value = "/quick", method = RequestMethod.GET, params = {"username"})   //请求映射
    public String quickMethod(){
        System.out.println("quickMethod running.......");
        //return：要跳转的视图
        //此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转。
        return "start";
    }

```

通过ModelAndView对象返回

```java
@RequestMapping("/quick2")
public ModelAndView quickMethod2(){
	ModelAndView modelAndView = new ModelAndView();
	modelAndView.setViewName("redirect:index.jsp");
	return modelAndView;
}
@RequestMapping("/quick3")
public ModelAndView quickMethod3(){
	ModelAndView modelAndView = new ModelAndView();
	modelAndView.setViewName("forward:/WEB-INF/views/index.jsp");
	return modelAndView;
}
```
通过SpringMVC框架注入的request对象setAttribute()方法要向request域中存储数据。

```java
@RequestMapping("/quick")
public String quickMethod(HttpServletRequest request){
	request.setAttribute("name","zhangsan");
	return "index";
}
```

  2、回写数据  
  直接返回字符串  
  a）：通过SpringMVC框架注入的response对象，使用response.getWriter().print(“hello world”) 回写数据，此时不需要视图跳转，业务方法返回值为void。
  
  b）：将需要回写的字符串直接返回，但此时需要通过@ResponseBody注解告知SpringMVC框架，方法返回的字符串不是跳转是直接在http响应体中返回
  
```java
@RequestMapping("/quick5")
@ResponseBody
public String quickMethod5() throws IOException {
	return "hello springMVC!!!";
}
```
  返回对象或者集合

```java
@RequestMapping("/quick8")
@ResponseBody
public User quickMethod8() throws IOException {
	User user = new User();
	user.setUsername("zhangsan");
	user.setAge(18);
	return user;
}
```
使用 < mvc:annotation-driven >默认底层就会集成jackson进行对象或集合的json格式字符串的转换。

```xml
<!--mvc的注解驱动-->
<mvc:annotation-driven/>
```
### 请求数据
一般来讲，客户端请求数据一般都是这种格式：name=XXX&password=XXX，在Javaweb阶段，这些请求数据都需要程序猿对数据进行封装和处理。但是在SpringMVC框架中，这些相应的封装可以由框架完成SpringMVC可以接受如下的类型的参数：

 - 基本类型参数
 -  POJO类型参数
 -  数组类型参数 
 - 集合类型参数
 
1、基本类型参数
Controller中的业务方法的参数名称要与请求参数的name一致，参数值会Spring框架就会自动映射匹配。
2、POJO类型参数
Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值会自动映射匹配。

```java
    public class User {
    	private String name;
    	private int age;
    	get/set
    }
    
    
    @RequestMapping(value = "/quick13")   //请求映射
    @ResponseBody
    public void quickMethod13(User user){
        System.out.println(user.getName());
        System.out.println(user.getAge());
    }
```
```xml
http://localhost:8080/SpringMVC/quick13?username=zhangsan&age=12
```
3、数组类型参数
Controller中的业务方法数组名称与请求参数的名称一致，参数值会自动映射匹配。
4、集合类型参数
获取集合的请求数据的方法有两种：
1、将集合参数包装到一个POJO类中 
首先讲集合包装到一个POJO类中：

```java
package com.kang.poj;

import java.util.List;

public class Vo {
    List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }

    @Override
    public String toString() {
        return "Vo{" +
                "userList=" + userList +
                '}';
    }
}
```
编写Controller

```java
    @RequestMapping(value = "/quick15")   //请求映射
    @ResponseBody
    public void quickMethod15(Vo vo){
        System.out.println(vo);
    }
```
编写对应的jsp

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="${pageContext.request.contextPath}/user/quick15" method="post">
        <input type="text" name="userList[0].name"><br>
        <input type="text" name="userList[0].age"><br>
        <input type="text" name="userList[1].name"><br>
        <input type="text" name="userList[1].age"><br>
        <input type="submit" value="提交">
    </form>
</body>
</html>

```
2、使用ajax提交。
当使用ajax提交时，可以指定contentType为json形式，那么在方法参数位置使用@RequestBody可以 直接接收集合数据而无需使用POJO进行包装。
${pageContext.request.contextPath}就是获得对应的虚拟地址，也就是SpringMVC这个地址
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<script src="js/jquery-3.6.0.js"></script>
<script>
    var userList = new Array();
    userList.push({name: "zhangsan", age: "20"});
    userList.push({name: "lisi", age: "18"});
    $.ajax({
        type: "POST",
        url: "${pageContext.request.contextPath}/user/quick16",
        data: JSON.stringify(userList),
        contentType: 'application/json;charset=utf-8'
    });
</script>
</body>
</html>

```

```java
    @RequestMapping(value = "/quick16")   //请求映射
    @ResponseBody
    public void quickMethod16(@RequestBody List<User> userList){
        System.out.println(userList);
    }

```
### 细节
**参数绑定注解**@requestParam
注解@RequestParam还有如下参数可以使用：
 value：与请求参数名称
 required：此在指定的请求参数是否必须包括，默认是true，提交时如果没有此参数则报错
defaultValue：当没有指定请求参数时，则使用指定的默认值赋值

```java
@RequestMapping("/quick14")
@ResponseBody
public void quickMethod14(@RequestParam(value="name",required =
	false,defaultValue = "itcast") String username) throws IOException {
	System.out.println(username);
}
```
**自定义类型转换器**
① 定义转换器类实现Converter接口

```java
package com.kang.converter;

import org.springframework.core.convert.converter.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @ClassName DataConverter
 * @Description TODO
 * @Author kang
 * @Date 2022/2/27 下午 2:18
 * @Version 1.0
 */
public class DataConverter implements Converter<String, Date> {
    @Override
    public Date convert(String s) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = simpleDateFormat.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```
2、在配置文件中声明转换器

```xml
    <!--配置文件中声明转换器-->
    <bean id="converterService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.kang.converter.DataConverter"></bean>
            </list>
        </property>
    </bean>

```
3、在< annotation-driven >中引用转换器
```xml
    <!--注解驱动-->
    <mvc:annotation-driven conversion-service="converterService"/>
```
 **获得请求头**
1、@RequestHeader
使用@RequestHeader可以获得请求头信息，相当于web阶段学习的request.getHeader(name)
@RequestHeader注解的属性如下：
value：请求头的名称
required：是否必须携带此请求头

```java
    @RequestMapping(value = "/quick21")   //请求映射
    @ResponseBody
    public void quickMethod21(@RequestHeader(value = "User-Agent", required = false) String userAgent){
        System.out.println(userAgent);
    }
```
2、@CookieValue
使用@CookieValue可以获得指定Cookie的值 @CookieValue注解的属性如下：
value：指定cookie的名称
required：是否必须携带此cookie

```java
    @RequestMapping(value = "/quick22")   //请求映射
    @ResponseBody
    public void quickMethod22(@CookieValue(value = "JSESSIONID", required = false) String cookie){
        System.out.println(cookie);
    }

```
**文件上传细节**
设置为多部份表单的时候，不再是键值对的形式，所以无法通过getParameter()方法获取数据。
1、单文件上传
导入fileupload和io坐标：

```xml
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.2.2</version>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.4</version>
    </dependency>

```
配置文件上传解析器：

```xml
    <!--文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--上传文件总大小-->
        <property name="maxInMemorySize" value="5242800"/>
        <!--上传单个文件的大小-->
        <property name="maxUploadSizePerFile" value="5242800"/>
        <!--上传文件的编码类型-->
        <property name="defaultEncoding" value="UTF-8"/>
    </bean>
```
编写文件上传代码：

```java
    @RequestMapping(value = "/quick23")   //请求映射
    @ResponseBody
    public void quickMethod23(String name, MultipartFile uploadFile) throws IOException {
        //获取文件名
        String originalFilename = uploadFile.getOriginalFilename();
        //保存文件
        uploadFile.transferTo(new File("F:\\笔记记录\\文件上传test\\" + originalFilename));
    }
```
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="${pageContext.request.contextPath}/user/quick23" method="post" enctype="multipart/form-data">
        名称<input type="text" name="username"><br>
        文件<input type="file" name="uploadFile"><br>
        <input type="submit" value="提交">
    </form>
</body>
</html>
```
2、多文件上传
```java
    @RequestMapping(value = "/quick24")   //请求映射
    @ResponseBody
    public void quickMethod24(String name, MultipartFile[] uploadFile) throws IOException {
        for (MultipartFile multipartFile : uploadFile) {
            //获取文件名
            String originalFilename = multipartFile.getOriginalFilename();
            //保存文件
            multipartFile.transferTo(new File("F:\\笔记记录\\文件上传test\\" + originalFilename));
        }
    }

```
```xml

<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="${pageContext.request.contextPath}/user/quick24" method="post" enctype="multipart/form-data">
        名称<input type="text" name="username"><br>
        文件1:<input type="file" name="uploadFile"><br>
        文件2:<input type="file" name="uploadFile"><br>
        <input type="submit" value="提交">
    </form>
</body>
</html>
```
## 4）SpringMVC拦截器
 Spring MVC 提供了 Interceptor 拦截器机制，用于请求的预处理和后处理。
 将拦截器按一定的顺序联结成一条链，这条链称为拦截器链（Interceptor Chain）。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。拦截器也是AOP思想的具体实现。

拦截器与过滤器的区别
![在这里插入图片描述](https://img-blog.csdnimg.cn/a4816864d7a64a6c9ed29d69506b35b5.png)
[快速入门](http://c.biancheng.net/spring_mvc/interceptor.html)
自定义拦截器很简单，只有如下三步：
① 创建拦截器类实现HandlerInterceptor接口
② 配置拦截器
③ 测试拦截器的拦截效果

 ①创建拦截器类实现HandlerInterceptor接口
```java
package net.biancheng.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class TestInterceptor implements HandlerInterceptor {
    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        System.out.println("afterCompletion方法在控制器的处理请求方法执行完成后执行，即视图渲染结束之后执行");

    }

    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle方法在控制器的处理请求方法调用之后，解析视图之前执行");
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle方法在控制器的处理请求方法调用之前执行");
        return false;
    }
}
```
② 配置拦截器
 

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 配置一个全局拦截器，拦截所有请求 -->
    <bean class="net.biancheng.interceptor.TestInterceptor" /> 
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/**" />
        <!-- 配置不需要拦截作用的路径 -->
        <mvc:exclude-mapping path="" />
        <!-- 定义<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="net.biancheng.interceptor.Interceptor1" />
    </mvc:interceptor>
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/gotoTest" />
        <!-- 定义在<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="net.biancheng.interceptor.Interceptor2" />
    </mvc:interceptor>
</mvc:interceptors>
```
配置说明：
> <mvc:interceptors>：该元素用于配置一组拦截器。
> <bean>：该元素是 <mvc:interceptors> 的子元素，用于定义全局拦截器，即拦截所有的请求。
> <mvc:interceptor>：该元素用于定义指定路径的拦截器。
> <mvc:mapping>：该元素是 <mvc:interceptor> 的子元素，用于配置拦截器作用的路径，该路径在其属性 path 中定义。path 的属性值为/**时，表示拦截所有路径，值为/gotoTest时，表示拦截所有以/gotoTest结尾的路径。如果在请求路径中包含不需要拦截的内容，可以通过 <mvc:exclude-mapping> 子元素进行配置。
## 5）SpringMVC的异常处理
Spring系统的Dao、 Service、 Controller出现都通过throws Exception向上抛出，最后由SpringMVC前端控制器交由异常处理器进行异常处理，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c911012b347468f94b21ffbb4067a4f.png)
### 使用SpringMVC提供的简单异常处理器SimpleMappingExceptionResolver
SpringMVC已经定义好了该类型转换器，在使用时可以根据项目情况进行相应异常与视图的映射配置

```xml
<!--配置简单映射异常处理器-->
<bean
	class=“org.springframework.web.servlet.handler.SimpleMappingExceptionResolver”>
	<property name=“defaultErrorView” value=“error”/>
	<property name=“exceptionMappings”>
		<map> 
			<entry key="com.itheima.exception.MyException" value="error"/>
			<entry key="java.lang.ClassCastException" value="error"/>
		</map>
	</property>
</bean>
```
### 自定义异常处理
实现Spring的异常处理接口HandlerExceptionResolver 自定义自己的异常处理器。基本步骤：
1、创建异常处理器类实现HandlerExceptionResolver
```java
public class MyExceptionResolver implements HandlerExceptionResolver {
    /*
       参数Exception：异常对象
       返回值ModelAndView：跳转到错误视图信息
    */
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView = new ModelAndView();
        if (e instanceof MyException) {
            modelAndView.addObject("info", "自定义异常");
        } else if (e instanceof ClassCastException) {
            modelAndView.addObject("info", "类转换异常");
        }
        modelAndView.setViewName("error");
        return modelAndView;
    }
}

```
2、配置异常处理器
```xml
<!--自定义异常处理器-->
<bean class="com.itheima.resolver.MyExceptionResolver"></bean>

```
3、编写异常页面

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>通用的错误提示页面</h1>
    <h1>${info}</h1>
</body>
</html>

```

# 三、MyBatis
## 1）简介

 - mybatis是一个优秀的基于java的持久层框架，它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。
 - mybatis通过xml或注解的方式将要执行的各种 statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句。 
 - mybatis框架执行sql并将结果映射为java对象并返回。采用ORM思想解决了实体和数据库映射的问题，对jdbc进行了封装，屏蔽了jdbc api 底层访问细节，使我们不用与jdbc api打交道，就可以完成对数据库的持久化操作。
开发步骤：
1、添加MyBatis的坐标

```xml
<!--mybatis坐标-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.32</version>
</dependency>
<!--mysql驱动坐标-->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<!--单元测试坐标-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
<!--日志坐标-->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

```
2、创建user数据库
3、编写User实体类

```java
public class User {
	private int id;
	private String username;
	private String password;
	//省略get和set方法
}

```

4、编写映射文件UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c601edaf6154cbe9a16baea96849a04.png)

5、编写核心映射文件SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--数据源环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>

    <!--加载映射文件-->
    <mappers>
        <mapper resource="com/itheima/mapper/UserMapper.xml"></mapper>
    </mappers>
</configuration>

```
6、编写测试代码

```java
//获得核心配置文件
InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
//获得session工厂对象
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
//获得session回话对象
SqlSession sqlSession = sqlSessionFactory.openSession();
//执行操作  参数：namespace+id
List<User> userList = sqlSession.selectList("userMapper.findAll");
//打印数据
System.out.println(userList);
//释放资源
sqlSession.close();
```
## 2）增删改查操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/a4a799c2c28e433aa6b6d60128c55953.png)
## 3）核心配置文件解析
1、environments标签
数据库环境的配置，支持多环境配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/63660f0fcad74e38978d2afe65778834.png)

> 其中，事务管理器（transactionManager）类型有两种： 
> ● JDBC：这个配置就是直接使用了JDBC的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域
>  ●MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如JEE 应用服务器的上下文）。默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false来阻止它默认的关闭行为。
> 
> 其中，数据源（dataSource）类型有三种：
>  ● UNPOOLED：这个数据源的实现只是每次被请求时打开和关闭连接。
>   ● POOLED：这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来。
>    ● JNDI：这个数据源的实现是为了能在如 EJB或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用

2、mapper标签
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c37930649954840bd5a3606104e7826.png)
3、Properties标签
![在这里插入图片描述](https://img-blog.csdnimg.cn/03a83a1e76ca4b86a01ce966fa14361b.png)
## 4）代理开发
采用 Mybatis 的代理开发方式实现 DAO 层的开发，这种方式是我们后面进入企业的主流。Mapper 接口开发方法只需要程序员编写Mapper 接口（相当于Dao 接口），由Mybatis 框架根据接口定义创建接口的动态代理对象，代理对象的方法体同上边Dao接口实现类方法。

> Mapper 接口开发需要遵循以下规范： 
> 1、Mapper.xml文件中的namespace与mapper接口的全限定名相同
> 2、Mapper接口方法名和Mapper.xml中定义的每个statement的id相同
> 3、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同 
> 4、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同

## 5）注解开发
MyBatis常用注解
![在这里插入图片描述](https://img-blog.csdnimg.cn/ee463b48333d40cb8f7a3a5dab9b705a.png)
简单入门：
修改MyBatis核心配置文件，我们使用了注解替代的映射文件

```xml
<mappers>
	<!--扫描使用注解的类-->
	<mapper class="com.itheima.mapper.UserMapper"></mapper>
</mappers>

```
UserMapper改为注解形式

```java
public interface UserMapper {
    @Insert("insert into user values(#{id},#{username},#{password},#{birthday})")
    public void save(User user);

    @Update("update user set username=#{username},password=#{password} where id=#{id}")
    public void update(User user);

    @Delete("delete from user where id=#{id}")
    public void delete(int id);

    @Select("select * from user where id=#{id}")
    public User findById(int id);

    @Select("select * from user")
    public List<User> findAll();
}

```
实现复杂关系映射之前我们可以在映射文件中通过配置<resultMap>来实现，使用注解开发后，我们可以使用@Results注解，@Result注解，@One注解，@Many注解组合完成复杂关系的配置：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d393bf7240147a594851d2fd7c45cfb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fe29b54e4ae7416db7ddadab0389e9b7.png)

