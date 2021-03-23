# Spring MVC

Spring MVC 是Spring给我们提供的一个用于简化Web开发的框架

## 初识Spring MVC

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

### Spring MVC请求处理流程

![image-20210323122039959](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_23/image-20210323122039959.png)

1. 用户发送请求到前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用HandlerMapping处理映射器
3. HandlerMapping根据请求的url地址找到具体的Handler，生成处理器对象及处理器拦截器（如果有则生成）并返回DispatcherServlet
4. 