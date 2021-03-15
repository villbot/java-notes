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
-Dcatalina.home=/Users/jianghai/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source
        -Dcatalina.base=/Users/jianghai/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source
        -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
        -Djava.util.logging.config.file=/Users/jianghai/Documents/resources.nosync/soundCode/apache-tomcat-8.5.64-src/source/conf/logging.properties
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

## 核心剖析源

  正在撰写中...


















































