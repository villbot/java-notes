# Tomcat性能优化策略

系统性能的衡量指标，主要是响应时间和吞吐量

1. 响应时间：执行某个操作的耗时
2. 吞吐量：系统在给定时间内能够支持的事务数量，单位为TPS(Transactions PerSecond的缩写)，也就是事务数/秒，一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。



**Tomcat优化主要从两个方面进行**

1. JVM虚拟机优化（Tomcat是基于Java语言编写的，所以是运行在Java虚拟机JVM之上）
2. Tomcat自身配置优化



## JVM虚拟机运行优化

对于对JVM模型有了解的同学可以直接参考 [JVM优化](../jvm/JVM优化.md)这篇文章

## Tomcat配置优化

* 调整Tomcat线程池

  ![image-20210322163625937](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_22/image-20210322163625937.png)

* 调整tomcat的连接器
   调整tomcat/conf/server.xml 中关于链接器的配置可以提升应用服务器的性能。

  | 参数           | 说明                                                         |
  | -------------- | ------------------------------------------------------------ |
  | maxConnections | 最大连接数，当到达该值后，服务器接收但不会处理更多的请求， 额外的请 求将会阻塞直到连接数低于maxConnections 。可通过ulimit -a 查看服务器 限制。对于CPU要求更高(计算密集型)时，建议不要配置过大 ; 对于CPU要求 不是特别高时，建议配置在2000左右(受服务器性能影响)。 当然这个需要服 务器硬件的支持 |
  | maxThreads     | 最大线程数,需要根据服务器的硬件情况，进行一个合理的设置      |
  | acceptCount    | 最大排队等待数,当服务器接收的请求数量到达maxConnections ，此时 Tomcat会将后面的请求，存放在任务队列中进行排序， acceptCount指的 就是任务队列中排队等待的请求数 。 一台Tomcat的最大的请求处理数量， 是maxConnections+acceptCount |

* 禁用 AJP 连接器

  ![image-20210322163834415](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_22/image-20210322163834415.png)

* 调整 IO 模式

  Tomcat8之前的版本默认使用BIO(阻塞式IO)，对于每一个请求都要创建一个线程来处理，不适合高并发;Tomcat8以后的版本默认使用NIO模式(非阻塞式IO)

  ![image-20210322164019083](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_22/image-20210322164019083.png)

* 动静分离

  可以使用Nginx+Tomcat相结合的部署方案，Nginx负责静态资源访问，Tomcat负责Jsp等动态资源访问处理(因为Tomcat不擅⻓处理静态资源)。