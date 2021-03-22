## 浅谈Http与Tomcat关系
提及Tomcat自然会想到http请求，那么http请求的处理过程是什么样的？

![](Apache%20Tomcat/73E87451-295C-4EC1-82AF-09116C6F0AA6.png)

1. 用户请求某个URL资源
2. 浏览器/客户端监听到用户操作，解析请求域名，检索DNS（浏览器DNS -> 系统DNS -> 路由DNS ->  ISP（互联网服务提供商） DNS缓存，除ISP其他的位置检索到DNS，需要验证是否过期，过期会向后位继续查询 )，查询服务器具体IP地址
3. 浏览器会根据查询到的ip地址进行TCP/IP连接
4. 连接建立成功后浏览器会生成HTTP格式的数据包使用TCP进行传输
5. 服务器接收到TCP请求并解析HTTP格式的数据包
6. 服务器根据解析后HTTP格式的数据包执行业务请求
7. 服务器执行业务请求后生成HTTP格式的数据包并使用TCP进行传输
8. 浏览器接收到TCP连接传输的HTTP格式的数据包解析，并关闭TCP连接
9. 浏览器根据解析后的HTTP格式数据进行渲染页面

**浏览器服务服务器使用的是http协议（应用层协议），用于定义数据通信的格式，但具体传输使用[[TCP连接]]协议**

## Tomcat处理流程
因此可以得出结论，Tomcat是一个http服务器（能够接收并处理http请求）

那么Tomcat是怎么样实现与业务代码进行交互的呢？

![](Apache%20Tomcat/0ED88F49-099B-4891-B798-AF43A4ADEED5.png)

1. 当用户请求与某个资源时，HTTP服务器会将Request对象封装成ServletRequest对象
2. 进一步会在Servlet容器中根据URL和Servlet映射关系，找到相应的Servlet，进一步去调用Servlet
3. **如果Servlet没有被加载，就用反射机制创建Servlet，并调用Servlet的init方法进行初始化**
4. 根据找到相应的Servlet，并且调用具体的实现方法，处理完成后将返回的信息用servletResponse进行封装
5. HTTP服务器会将Servlet容器返回的ServletResponse转为Response返回给客户端（浏览器）

## Tomcat核心组件
可以发现Tomcat完成了两个重要的功能
1. 和客户端进行交互，进行socket通信，并将request和response对象进行封装转换
2. Servlet容器逻辑处理

那么Tomcat是如何实现这两个重要功能？

Tomcat设计了两个核心组件
1. 连接器组件（Coyote）：负责对外交流，处理socket连接，负责网络字节流与request、Response对象的转换。
2. 容器组件（Catalina ）：负责内部处理，加载和管理servlet，以及具体处理request请求

### Tomcat 模块分层图

当然除了Coyote和Catalina 之外也存在一些其他的模块
Jasper 模块提供 JSP 引 擎，Naming 提供JNDI 服务，Juli 提供日志服务。 

![](Apache%20Tomcat/70CE4F07-A502-4B2D-9380-B4B1A5507F00.png)

根据上图Tomcat模块分层图很直观能看到coyote 和Catalina是比较关键组件，那么下面我们具体来分析这个几个组件之间的关系

### 连接器组件（Coyote）
顾名思义，连接器组件的职责就是服务器建立连接，发送请求并接受响应

![](Apache%20Tomcat/396A7420-42BC-43A8-B6B8-B7D331F811C8.png)

如上图所示，Coyote（连接器组件）与容器（Catalina）之间交互，

1. Coyote封装了底层网络通信（socket请求及响应处理）
2. Coyote使Catalina容器与具体请求协议及IO操作完全解耦
3. coyote将socket输入封装为request对象，进一步封装后交由Catalina容器进行处理，处理完成后，Catalina通过coyote提供的response对象结果写入输出流
4. coyote是具体协议（应用层）和IO（传输层）相关内容

#### Tomcat Coyote 支持的IO模型与协议

![](Apache%20Tomcat/page6image30771856.png) 

在 8.0 之前 ，Tomcat 默认采用的I/O方式为 BIO，之后改为 NIO。 无论 NIO、NIO2 还是 APR， 在性 能方面均优于以往的BIO。 如果采用APR， 甚至可以达到 Apache HTTP Server 的响应性能。 

#### Tomcat Coyote 的内部组件及流程

![](Apache%20Tomcat/A0C7C7C0-D7CE-481C-950F-8F79E7D1E075.png)

* **EndPoint：**
	EndPoint 是 Coyote 通信端点，即通信监听的接口，是具体Socket接收和发 送处理器，是对传输层的抽象，因此EndPoint用来实现TCP/IP协议的 
* **Processor：** 
	Processor 是Coyote 协议处理接口 ，如果说EndPoint是用来实现TCP/IP协 议的，那么Processor用来实现HTTP协议，Processor接收来自EndPoint的 Socket，读取字节流解析成Tomcat Request和Response对象，并通过 Adapter将其提交到容器处理，Processor是对应用层协议的抽象 
* **ProtocolHandler：** 
	Coyote 协议接口， 通过Endpoint 和 Processor ， 实现针对具体协议的处 理能力。Tomcat 按照协议和I/O 提供了6个实现类 : AjpNioProtocol ， AjpAprProtocol， AjpNio2Protocol ， Http11NioProtocol ， Http11Nio2Protocol ，Http11AprProtocol 
* **Adapter：**
	由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了自己的 Request类来封装这些请求信息。ProtocolHandler接口负责解析请求并生成 Tomcat Request类。但是这个Request对象不是标准的ServletRequest，不 能用Tomcat Request作为参数来调用容器。
	Tomcat设计者的解决方案是引 入CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter的Sevice方法，传入的是Tomcat Request对象， CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调用容器

### Tomcat Servlet 容器 Catalina

Tomcat是一个由多个可配置（conf/server.xml）组件组成的web容器，而Catalina 是Tomcat的servlet容器，也可以说Tomcat是一个Servlet容器，Catalina是Tomcat的核心，其他模块都是为Catalina提供支撑的

![](Apache%20Tomcat/735CCE59-3CE7-407B-8583-D927561D42F5.png)

如上图所示，可以认为Tomcat就是一个Catalina实例，Tomcat启动的时候会初始化Catalina实例，Catalina实例又会加载server.xml完成其他实例创建，创建一个server，server又会创建并管理多个service服务，每个service又会存在多个connector和一个container。

* Catalina：
	负责解析Tomcat配置文件（server.xml）,以此来创建服务器Server组件并进行管理。
* Server：
	服务器标识整个Catalina Servlet容器以及其他组件，负责组装启动servlaet引擎，tomcat连接器。
	Server通过实现Lifecycle接口，提供了一种优雅的启动和关闭系统的方式
* Service：
	是server内部的组件，一个server包含多个service，将若干个connector组件绑定到一个container。
* container：
	负责处理用户Servlert请求，并返回对象给web用户的模块
	
#### 那么在container中其他的组件又负责什么职责呢？
* Engine：
	表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine， 但是一个引擎可包含多个Host 
* Host：
	代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下 
可包含多个Context
* Context：
	表示一个Web应用程序， 一个Web应用可包含多个Wrapper 
* Wrapper
	表示一个Servlet，Wrapper 作为容器中的最底层，不能包含子容器 

## Tomcat服务器核心配置详解
在[[Apache Tomcat]]中可以可以看到整体Apache Tomcat整体的一个架构情况以及核心组件的流程，那么在Tomcat中，如何去配置相关的核心组件呢？其实配置文件就在conf/server.xml中

![](Apache%20Tomcat/93BE0FDF-802D-4D72-B23D-CB10F6530A6F.png)

文件上查看 <a href='https://github.com/elgchat/elgChat/blob/main/docs/web%20container/Apache%20Tomcat/server.xml'>server.xml</a>

## TinyCat - 手写实现简易版的Tomcat

[TinyCat](https://github.com/elgchat/TinyCat)
