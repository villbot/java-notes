

# 初识Spring MVC

Spring MVC 是Spring给我们提供的一个用于简化Web开发的框架

## Spring MVC简介

###  **MVC** 体系结构

MVC 全名是 Model View Controller，是 模型(model)-视图(view)-控制器(controller) 的缩写， 是一种用于设计创建 Web 应用程序表现层的模式。MVC 中每个部分各司其职: 

* Model(模型):模型包含业务模型和数据模型，数据模型用于封装数据，业务模型用于处理业务。

* View(视图): 通常指的就是我们的 jsp 或者 html。作用一般就是展示数据的。通常视图是依据 模型数据创建的。

* Controller(控制器): 是应用程序中处理用户交互的部分。作用一般就是处理程序逻辑的。 MVC提倡:每一层只编写自己的东⻄，不编写任何其他的代码;分层是为了解耦，解耦是为了维护方便和分工协作。

### Spring MVC是什么？

SpringMVC 全名叫 Spring Web MVC，是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级Web 框架，属于 SpringFrameWork 的后续产品。

![Spring的体系结构](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_23/5-1Z606104H1294.gif)

它通过一套注解，让一个简单的 Java 类成为处理请求的控制器，而无须实现任何接口。

同时它还支持 RESTful 编程⻛格的请求。

Spring MVC 本质可以认为是对servlet的封装，简化了我们serlvet的开发作用:

1. 接收请求
2. 返回响应，跳转⻚面

## Spring MVC工作流程

### Spring MVC请求处理流程

![image-20210323122039959](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_23/image-20210323122039959.png)

1. 用户发送请求到前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用HandlerMapping处理映射器
3. HandlerMapping根据请求的url地址找到具体的Handler，生成处理器对象及处理器拦截器（如果有则生成）并返回DispatcherServlet
4. DispatcherServlet调用HandlerAdapter处理器适配器去调用Handler
5. 处理器适配器执行Handler
6. Handler执行完成给处理器适配器返回ModelAndView
7. 处理器适配器向前端控制器返回 ModelAndView，ModelAndView 是SpringMVC 框架的一个 底层对象，包括 Model 和 View
8. 前端控制器请求视图解析器去进行视图解析，根据逻辑视图名来解析真正的视图。
9. 视图解析器向前端控制器返回View
10. 前端控制器进行视图渲染，就是将模型数据(在 ModelAndView 对象中)填充到 request 域
11. 前端控制器向用户响应结果

### Spring MVC 九大组件

* **HandlerMapping(处理器映射器)**

  HandlerMapping 是用来查找 Handler 的，也就是处理器，具体的表现形式可以是类，也可以是 方法。比如，标注了@RequestMapping的每个方法都可以看成是一个Handler。Handler负责具体实际的请求处理，在请求到达后，HandlerMapping 的作用便是找到请求相应的处理器 Handler 和 Interceptor.

* **HandlerAdapter(处理器适配器)**

  HandlerAdapter 是一个适配器。因为 Spring MVC 中 Handler 可以是任意形式的，只要能处理请求即可。但是把请求交给 Servlet 的时候，由于 Servlet 的方法结构都是 doService(HttpServletRequest req,HttpServletResponse resp)形式的，要让固定的 Servlet 处理 方法调用 Handler 来进行处理，便是 HandlerAdapter 的职责。

* **HandlerExceptionResolver**
   HandlerExceptionResolver 用于处理 Handler 产生的异常情况。它的作用是根据异常设置ModelAndView，之后交给渲染方法进行渲染，渲染方法会将 ModelAndView 渲染成⻚面。

* **ViewResolver**

  ViewResolver即视图解析器，用于将String类型的视图名和Locale解析为View类型的视图，只有一个resolveViewName()方法。从方法的定义可以看出，Controller层返回的String类型视图名 viewName 最终会在这里被解析成为View。View是用来渲染⻚面的，也就是说，它会将程序返回 的参数和数据填入模板中，生成html文件。ViewResolver 在这个过程主要完成两件事情: ViewResolver 找到渲染所用的模板(第一件大事)和所用的技术(第二件大事，其实也就是找到 视图的类型，如JSP)并填入参数。默认情况下，Spring MVC会自动为我们配置一个 InternalResourceViewResolver,是针对 JSP 类型视图的。

* **RequestToViewNameTranslator**

  RequestToViewNameTranslator 组件的作用是从请求中获取 ViewName.因为 ViewResolver 根据 ViewName 查找 View，但有的 Handler 处理完成之后,没有设置 View，也没有设置 ViewName， 便要通过这个组件从请求中查找 ViewName。

* **LocaleResolver**

  ViewResolver 组件的 resolveViewName 方法需要两个参数，一个是视图名，一个是 Locale。 LocaleResolver 用于从请求中解析出 Locale，比如中国 Locale 是 zh-CN，用来表示一个区域。这个组件也是 i18n 的基础。

* **ThemeResolver**

  ThemeResolver 组件是用来解析主题的。主题是样式、图片及它们所形成的显示效果的集合。 Spring MVC 中一套主题对应一个 properties文件，里面存放着与当前主题相关的所有资源，如图 片、CSS样式等。创建主题非常简单，只需准备好资源，然后新建一个“主题名.properties”并将资 源设置进去，放在classpath下，之后便可以在⻚面中使用了。SpringMVC中与主题相关的类有 ThemeResolver、ThemeSource和Theme。ThemeResolver负责从请求中解析出主题名， ThemeSource根据主题名找到具体的主题，其抽象也就是Theme，可以通过Theme来获取主题和 具体的资源。

* **MultipartResolver**

  MultipartResolver 用于上传请求，通过将普通的请求包装成 MultipartHttpServletRequest 来实现。MultipartHttpServletRequest 可以通过 getFile() 方法 直接获得文件。如果上传多个文件，还 可以调用 getFileMap()方法得到Map<FileName，File>这样的结构，MultipartResolver 的作用就 是封装普通的请求，使其拥有文件上传的功能。

* **FlashMapManager**

  FlashMap 用于重定向时的参数传递，比如在处理用户订单时候，为了避免重复提交，可以处理完 post请求之后重定向到一个get请求，这个get请求可以用来显示订单详情之类的信息。这样做虽然 可以规避用户重新提交订单的问题，但是在这个⻚面上要显示订单的信息，这些数据从哪里来获得 呢?因为重定向时么有传递参数这一功能的，如果不想把参数写进URL(不推荐)，那么就可以通 过FlashMap来传递。只需要在重定向之前将要传递的数据写入请求(可以通过ServletRequestAttributes.getRequest()方法获得)的属性OUTPUT_FLASH_MAP_ATTRIBUTE 中，这样在重定向之后的Handler中Spring就会自动将其设置到Model中，在显示订单信息的⻚面 上就可以直接从Model中获取数据。FlashMapManager 就是用来管理 FalshMap 的。

## Spring MVC 参数绑定及请求拦截

### 回顾Spring MVC的配置流程

1. 首先需要在pom.xml导入Spring mvc的jar包

   ```xml
   <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.12.RELEASE</version>
   </dependency>
   ```

2. 在resources中建立springmvc.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/mvc
          http://www.springframework.org/schema/mvc/spring-mvc.xsd">
   
       <!-- 开启controller-->
       <context:component-scan base-package="com.elgchat.controller"/>
       <!--配置spring mvc的视图解析器-->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/jsp/"/>
           <property name="suffix" value=".jsp"/>
       </bean>
   
       <!-- 注册最合适的处理器映射器，处理器适配器-->
       <mvc:annotation-driven/>
   </beans>
   ​```
   ```
   
3. 在webapp/WEB-INF/web.xml 配置DispatcherServlet前端控制器

     ```xml
   <!DOCTYPE web-app PUBLIC
           "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
           "http://java.sun.com/dtd/web-app_2_3.dtd" >
   
   <web-app>
       <display-name>Archetype Created Web Application</display-name>
       <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <!--关联springmvc配置文件-->
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:springmvc.xml</param-value>
           </init-param>
       </servlet>
       <!--那些请求需要被spring mvc 进行处理-->
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
   
           <!-- 方式一：带后缀 *.action，*.do-->
           <!-- 方式二：/ 不会拦截.jsp等静态文件请求-->
           <!-- 方式二：/* 即拦截所有-->
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   </web-app>
     ```

4. 创建controller

   ```java
   @Controller
   @RequestMapping("/demo")
   public class DemoController {
   
       /**
        * url: http://localhost:8080/demo/handle01
        */
       @RequestMapping("/handle01")
       public ModelAndView handle01() {
           LocalDateTime localDateTime = LocalDateTime.now();
           ModelAndView modelAndView = new ModelAndView();
           //addObject 其实是向请求域中赋值request.setAttribute("time",localDateTime);
           modelAndView.addObject("time", localDateTime);
           modelAndView.setViewName("success");
           return modelAndView;
       }
   }
   ```

5. 创建Jsp资源目录，根据springmvc.xml配置的spring mvc的视图解析器中的prefix属性一致，这里是配置资源目录的前缀及后缀

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
   <html>
   <head>
       <title>success</title>
   </head>
   <body>
    请求成功，当前系统时间 ${time}
   </body>
   </html>
   
   ```

    **总结开发过程**
   1. **配置DispatcherServlet前端控制器** 
   2. **开发处理具体业务逻辑的Handler(@Controller、@RequestMapping)** 
   3. **xml配置文件配置controller扫描，配置springmvc三大件(视图解析器、处理器映射器，处理器适配器)** 
   4. **将xml文件路径告诉springmvc(DispatcherServlet)**

### 如何进行请求拦击处理？

​		实际上就在webapp/WEB-INF/web.xml中配置的<url-pattern>

