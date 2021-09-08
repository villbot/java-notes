## Java Virtual Machine 整体架构

> 根据Java Virtual Machine 规范，Java Virtual Machine 内容共分为虚拟机桟、堆、方法区、程序计数器、本地方法桟五个部分。

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_08/image-20210908114554614.png" alt="image-20210908114554614" style="zoom:50%;" />

| 名称       | 特征                                     | 作用                   | 配置参数 | 异常 |
| ---------- | ---------------------------------------- | ---------------------- | -------- | ---- |
| 程序计数器 | 占用内存小，线程私有，生命周期与线程相同 | 大致为字节码行号指示器 | 无        | 无   |
| 虚拟机桟 | 线程私有，生命周期与线程相同，使用连续的内存空间 | Java方法执行的内存模型，存储局部变量表、操作桟、动态链接、方法出口等信息 | -Xss | StackOverflowError/<br>OutOfMemoryError |
| 堆 | 线程共享，生命周期与虚拟机相同，可以不适用连续的内存地址 | 保存对象实例，所有对象实例（包括数组）都要在堆上分配 | -xms -Xsx -Xmn | OutOfMemoryError |
| 方法区 | 线程共享，生命周期与虚拟机相同，可以不适用连续的内存地址 | 存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据 | -XX:PermSize:16M-XX:MaxPermSize64M/-xx:MetaspaceSize=16M-XX:MaxMetaspaceSize=64M | OutOfMemoryError |
| 本地方法桟 | 线程私有 | 为虚拟机使用到的Native方法服务 | 无 | StackOverflowError/<br/>OutOfMemoryError |

> JVM分为五大模块: **类装载器子系统** 、 **运行时数据区** 、 **执行引擎** 、 **本地方法接口** 和 **垃圾收集模块** 

![image-20210908134736407](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_08/image-20210908134736407.png)

## Java Virtual Machine 运行时内存

Java虚拟机有自动内存管理机制，如果出现面的问题，排查错误就必须要了解虚拟机是怎样使用内存的

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_08/image-20210908141147305.png" alt="image-20210908141147305" style="zoom:50%;" />

### Java7和Java8内存结构的不同主要体现在方法区的实现

方法区是java虚拟机规范中定义的一种概念上的区域。不同的厂商可以对虚拟机进行不同的实现。

我们通常使用的是Java SE 都有由Sun Jdk 和Open Jdk所提供的，着也是应用最广泛的版本，而该版本使用的vm就是HotSpot VM，通常情况下，我们所讲的java虚拟机指的就是HotSpot版本。

#### Jdk7 内存结构

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_08/image-20210908142931878.png" alt="image-20210908142931878" style="zoom:50%;" />

#### Jdk8 内存结构

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_08/image-20210908143620292.png" alt="image-20210908143620292" style="zoom:50%;" />
