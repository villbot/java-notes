# Tomcat源码剖析
## 构建源代码
### 下载源代码
1. 访问 [Apache Tomcat® 官网](https://tomcat.apache.org) ，选择Download，找到对应的版本
2. 下载**Source Code Distributions/** [8.5.64 - tar.gz](https://apache.website-solution.net/tomcat/tomcat-8/v8.5.64/src/apache-tomcat-8.5.64-src.tar.gz) （当前以Tomcat 8.5.64版本为案例）

### 构建源代码
1. 将下载的 apache-tomcat-8.5.64-src.tar.gz 文件解压
2. 在 apache-tomcat-8.5.64-src根目录建立source目录（目录名称可以随意定义）
3. 将conf和webapp目录移动到source文件夹
4. 回到apache-tomcat-8.5.50-src根目录，创建pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>apache-tomcat-8.5.64-src</artifactId>
    <name>Tomcat8.5</name>
    <version>8.5</version>
    <build>
        <!--指定源目录-->
        <finalName>Tomcat8.5</finalName>
        <sourceDirectory>java</sourceDirectory>
        <resources>
            <resource>
                <directory>java</directory>
            </resource>
        </resources>
        <plugins>
            <!--引入编译插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <!--tomcat 依赖的基础包-->
    <dependencies>
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.0</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.5.1</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.soap</groupId>
            <artifactId>javax.xml.soap-api</artifactId>
            <version>1.4.0</version>
        </dependency>
    </dependencies>
</project>
```

5. 将项目导入IDEA中，找到Bootstrap文件中的main启动
6. （🐞可能会出现问题）
**问题：java: 程序包 sun.rmi.registry 不可见  (程序包 sun.rmi.registry 已在模块 java.rmi 中声明, 但该模块未将它导出到未命名模块)**
将sun.rml.registry 加入编译器运行配置即可解决问题

7. 在启动类Bootstrap中配置VM参数，Tomcat启动需要加载配置文件（即最开始创建的source目录）

``` properties
# 此处目录地址应响应做修改，更换成自己本地地址
-Dcatalina.home=/Users/elgchat/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source
        -Dcatalina.base=/Users/elgchat/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source
        -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
        -Djava.util.logging.config.file=/Users/elgchat/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source/conf/logging.properties
```

8. 启动服务（找到Bootstrap文件中的main启动），启动成功访问[localhost:8080](http://localhost:8080)
9. （🐞可能会出现问题）
**问题：出现500错误 org.apache.jasper.JasperException: 无法为JSP编译类**
在ContextConfig类中configureStart方法内 webConfig()下增加一行代码将jsp引擎初始化

```java

protected synchronized void configureStart() {
	//....
	webConfig();
	//初始化JSP引擎 -> Jasper
	context.addServletContainerInitializer(new JasperInitializer(),null);
	//....
}
```

10. 再次启动项目，成功访问[localhost:8080](http://localhost:8080)，可以看到Tomcat管理页面，这样我们就完全编译好Tomcat的源码了。

## 剖析核心代码
Tomcat 我们只需要关注两个流程，Tomcat启动流程、Tomcat请求处理流程

### Tomcat 启动流程

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/Tomcat%E5%90%AF%E5%8A%A8%E6%97%B6%E5%BA%8F%E5%9B%BE.png)

根据上面Tomcat的时序图我们去追踪源代码

1. 启动startup.sh/startup.bat（这里只查看startup.sh，区别是sh为linux/unix脚本，bat为windows平台）
查看startup.sh可以看到实际调用的是catalina.sh
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/B7F76242-94C8-4358-9A96-354981AFA3A4.png)
查看catalina.sh可以看到调用是org.apache.catalina.startup.Bootstrap “$@“ start (即Tomcat启动类)
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/55969279-F171-49F0-B697-410A6E0C5BC9.png)

查看BootStrap类中的main方法，可以看到，启动会调用init方法
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/7C8C0D07-753C-4B6E-994D-257685CFC1D8.png)
2. 进入init()方法，可以看到进行类初始化操作，以及对catalinaDaemon进行赋值 -> Catalina实例
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/DE0D882B-375A-47B2-8A4F-3E8153144375.png)

3. 回到main方法中可以看到根据传递进来的参数进入到start判断，进行load()和start().
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/917B13E9-D9ED-450C-B03D-444B504AAFE8.png)

进入load()方法， **关键method.invoke(catalinaDaemon, param);** ，可以看到通过反射调用catalinaDaemon中的load方法 -> Catalina.load()；
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/F2133538-1BF2-470D-BC7A-C4E5E8C51D1A.png)

4. 进入Catalina.load()方法，可以看到创建了一个Digester（xml解析器）
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/3C4E1FD0-F634-4A1F-9B7A-C7FB52234CE1.png)

5. 可以看到根据上面的xml解析器创建server容器，并调用init方法
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/12C8A0EE-A519-4A22-9845-121C841DF006.png)

进入init()方法可以看到进入Lifecycle接口
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/A2838959-A1FA-4152-825A-B9D62417C7B7.png)

可以看到Lifecycle init()实现是在LifecycleBase中的init()方法![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/31989E07-D777-4690-B28D-BC2DA275678F.png)

进入initInternal()，发现这是一个抽象的类，可以发现这是一个典型的模板方法的调用
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/D025D159-14DE-430A-9EF7-44FC34B044E7.png)

6. 因为当前调用server，那么进入实现找到StandardServer
  
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/31D6FAF5-606F-420E-B800-08C45BACCCFE.png)

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/1D53034D-5FB4-4393-8178-39D53A4FBCA7.png)


7. 进入init() 发现又进入到Lifecycle中，但因为当前调用service，那么进入实现找到StandardService

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/2F864AB8-A100-4C3D-A102-0EB492404DCE.png)

8. 可以看到engine.init();点击进入
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/96F9A494-0CF0-4E44-A6CA-B603F0C1FF94.png)
点击进入 super.initInternal();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/61510C9D-30D4-4BD0-8333-10E8A311C28D.png)
这里创建了线程池，继续点击super.initInternal();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/85BF67A4-8458-4BB5-8481-89779D53F74D.png)
9. 这里getObjectNameKeyProperties()，可以看到调用StandardHost中的getObjectNameKeyProperties()
注册并初始化 host
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/CB18F3F6-BF9A-4D92-97A9-63EED19F1827.png)
10. 进入StandardContext.getObjectNameKeyProperties().初始化context
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/69CC5FC1-213B-4441-926B-E95C0923AA08.png)

11. 回到StandardService.initInternal()中可以看到又调用了executor.init();
12. 在for循环中可以看到调用了connector.init()；证明在一个service中可以存在多个connector，点击进入init()方法，发现又进入到了LifecycleBase中，点击进入initInternal()，按照以上的惯例，判定为模板方法，那么寻找他的实现，找到Connector类中
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/BC98207C-D40E-421D-B8BF-AB3783738453.png)
13. 点击进入protocolHandler.init()中，找到对应的实现AbstractHttp11Protocol

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/87DFB713-881F-4B29-B4DF-BF075A65A2F6.png)
点击进入super.init();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/B93E4457-8B83-4D6F-BB57-876BA7554E26.png)

可以看到这里又调用了endpoint.init();

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/1BAFEE6C-A417-437C-B0D2-95A19455990D.png)

点击 bind()，可以看到是一个抽象类，那么寻找他的实现，默认是NioEndpoint
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/4587B15E-0C71-4A25-A8C5-89C377C947C0.png)
可以看出，这里最终实现serverSock绑定端口逻辑。以上为Tomcat各个容器之间的初始化

14. 回到BootStrap的main方法中
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/DD8D7499-1CB2-4F4A-B195-CA7D343982C5.png)

点击start() 进入启动流程。

15. 通过反射调用Catalina.start方法
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/0E25A800-DE54-4FAF-95FF-F08541F3D87D.png)

16. 进入Catalina.start()方法，可以看到调用了getServer().start();

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/1C24AF3C-E2BD-44AA-9391-A6027D34CC09.png)


17. 进入StandardServer，会发现start方法在LifecycleMBeanBase中
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/E988C595-4A1B-4337-85FD-091551584087.png)

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/F9A30210-F953-4FAC-986A-EAF79E3259A3.png)

![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/0F9F72A7-4CFB-4768-B70F-A726659BA1D1.png)

这时进入StandardService.startInternal()；
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/041D11FC-F9D8-463A-A6FD-EE4FBE3B628F.png)
18. 可以看到调用了engine.start();
19. 
20.  
21. executor.start();
22. 循环体调用了connector.start();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/87ACA8DC-2C6E-4709-9A59-1A4439C6BA2D.png)

进入startInternal();
找到实现Connector中的startInternal()
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/602B5D54-48EE-45E7-A34E-87E75F2A850C.png)

再次进入protocolHandler.start(); 选择AbstractProtocol.start();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/6EB574CC-32BF-4BD8-BBD7-3ACB570EF066.png)
可以看到这里又调用了endpoint.start();
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/69702437-C910-447D-AE27-6991C6DEB18D.png)

点击startInternal();选择默认NioEndpoint
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/6F2DDD02-98AE-4AA0-BCAF-6AED1862B768.png)
点击startAcceptorThreads()；
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/E7029F77-1C0C-4471-B067-89BDF6F7E9D9.png)

点击createAcceptor(); 
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/3C6A0150-06C8-499C-9220-F5BBC686771B.png)

发现这个类是一个线程类，那么点击run()方法查看当前类的处理内容
![](Tomcat%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90/5ADE9B50-4311-420B-AC07-73FD09F8582A.png)
发现 在这里调用了serverSock.accept() 开始监听端口。


















































