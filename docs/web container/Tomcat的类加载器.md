# Tomcat 的类加载器
Tomcat类加载器机制不得不提到Jvm的类加载机制，因为Tomcat类是由Java编写的，即Tomcat的类加载器机制是基于Jvm的类加载机制实现的，对于Jvm的类加载及双亲委派机制可以查看[Tomcat的类加载器](../jvm/JVM的类加载机制和双亲委派机制.md)。
那么Tomcat类加载机制是什么样子的呢？其实是相对于Jvm类加载器机制做了一些改变，没有严格遵从双亲委派机制，也可以说是打破了双亲委派机制。

## 为什么说Tomcat类加载机制是打破了双亲委派机制
假设Tomcat下webapps里部署了两个应用
/tomcat/webapps/demo-1.jar	
/tomcat/webapps/demo-2.jar	
其两个应用下的包名均为com.elgchat.demo，但包体下的类名一致，但内容不一致

这样的情况就会导致Tomcat启动，加载完demo-1.jar中的类，去加载demo-2.jar中的类时，会认为这个类已经被加载过了，不会再去加载，那么这种情况肯定不能满足要求

## Tomcat类加载机制是怎么实现的呢？

![](Tomcat%20%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8/37CA89F1-3453-4FC3-87B7-66EA45E365C2.png)


* 引导类加载器 和 扩展类加载器 的作用不变 
* 系统类加载器正常情况下加载的是 CLASSPATH 下的类，但是 Tomcat 的启动脚本并未使用该变 量，而是加载tomcat启动的类，比如bootstrap.jar，通常在catalina.bat或者catalina.sh中指定。 位于CATALINA_HOME/bin下 
* Common 通用类加载器加载Tomcat使用以及应用通用的一些类，位于CATALINA_HOME/lib下， 比如servlet-api.jar 
* Catalina ClassLoader 用于加载服务器内部可⻅类，这些类应用程序不能访问 
* Shared ClassLoader 用于加载应用程序共享类，这些类服务器不会依赖 
* Webapp ClassLoader，每个应用程序都会有一个独一无二的Webapp ClassLoader，他用来加载本应用程序 /WEB-INF/classes 和 /WEB-INF/lib 下的类。 

**Tomcat 8.5 默认改变了严格的双亲委派机制**
* 首先从 Bootstrap Classloader加载指定的类 
* 如果未加载到，则从 /WEB-INF/classes加载
* 如果未加载到，则从 /WEB-INF/lib/\*.jar 加载
* 如果未加载到，则依次从 System、Common、Shared 加载(在这最后一步，遵从双亲委派机制) 
