## 什么是JVM？

> Java Virtual Machine（JVM）即Java虚拟机，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。
>
> 引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。

### 主流虚拟机

| 虚拟机名称 | 介绍                                                         |
| ---------- | ------------------------------------------------------------ |
| HotSpot    | Oracle/Sun JDK和OpenJDK都使用HotSPot VM的相同核心            |
| J9         | J9是IBM开发的高度模块化的JVM                                 |
| JRockit    | JRockit 与 HotSpot 同属于 Oracle，目前为止 Oracle 一直在推进 HotSpot 与 JRockit 两款各有优势的虚拟机进行融合互补 |
| Zing       | 由Azul Systems根据HostPot为基础改进的高性能低延迟的JVM Android上的Dalvik 虽然名字不叫JVM，但骨子里就是不折不扣的JVM |
| Dalvik     | Android上的Dalvik 虽然名字不叫JVM，但骨子里就是不折不扣的JVM |

## JVM和操作系统的关系

### 为什么要在程序和操作系统中间添加一个JVM？

Java 是一门抽象程度特别高的语言，提供了自动内存管理等一系列的特性。这些特性直接在操作系统上实现是不太 可能的，所以就需要 JVM 进行一番转换。

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/image-20210907162102045.png" alt="image-20210907162102045" style="zoom:50%;" />

从上图可以看到，有了 JVM 这个抽象层之后，Java 就可以实现跨平台了。JVM 只需要保证能够正确执行 .class 文 件，就可以运行在诸如 Linux、Windows、MacOS 等平台上了。

而 Java 跨平台的意义在于一次编译，处处运行，能够做到这一点 JVM 功不可没。比如我们在 Maven 仓库下载同一 版本的 jar 包就可以到处运行，不需要在每个平台上再编译一次。

现在的一些 JVM 的扩展语言，比如 Clojure、JRuby、Groovy 等，编译到最后都是 .class 文件，Java 语言的维护 者，只需要控制好 JVM 这个解析器，就可以将这些扩展语言无缝的运行在 JVM 之上了。

### 应用程序、JVM、操作系统之间的关系

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/image-20210907151701564.png" alt="image-20210907151701564" style="zoom:50%;" />

我们用一句话概括 JVM 与操作系统之间的关系：JVM 上承开发语言，下接操作系统，它的中间接口就是字节码。

## JVM、JRE、JDK的关系

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/image-20210907151727578.png" alt="image-20210907151727578" style="zoom:50%;" />

JVM 是 Java 程序能够运行的核心。但是需要注意，JVM 自己什么也干不了，你需要给它提供生产原料(.class 文 件) 。

仅仅是 JVM，是无法完成一次编译，处处运行的。它需要一个基本的类库，比如怎么操作文件、怎么连接网络等。 而 Java 体系很慷慨，会一次性将 JVM 运行所需的类库都传递给它。JVM 标准加上实现的一大堆基础类库，就组成 了 Java 的运行时环境，也就是我们常说的 JRE(Java Runtime Environment)

对于 JDK 来说，就更庞大了一些。除了 JRE，JDK 还提供了一些非常好用的小工具，比如 javac、java、jar 等。它 是 Java 开发的核心，让外行也可以炼剑!

我们也可以看下 JDK 的全拼，Java Development Kit。我非常怕 kit(装备)这个单词，它就像一个无底洞，预示着 你永无休止的对它进行研究。JVM、JRE、JDK 它们三者之间的关系，可以用一个包含关系表示。

![UFOA AM C](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/UFOA%20AM%20C.png)

## Java虚拟机规范和Java语言规范的关系

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/image-20210907154656197.png" alt="image-20210907154656197" style="zoom:50%;" />

左半部分是 Java 虚拟机规范，其实就是为输入和执行字节码提供一个运行环境。右半部分是我们常说的 Java 语法 规范，比如 switch、for、泛型、lambda 等相关的程序，最终都会编译成字节码。而连接左右两部分的桥梁依然是 Java 的字节码。

如果 .class 文件的规格是不变的，这两部分是可以独立进行优化的。但 Java 也会偶尔扩充一下 .class 文件的格式， 增加一些字节码指令，以便支持更多的特性。

我们可以把 Java 虚拟机可以看作是一台抽象的计算机，它有自己的指令集以及各种运行时内存区域，学过《计算机组成结构》的同学会在课程的后面看到非常多的相似性。

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_09_07/image-20210907155402075.png" alt="image-20210907155402075" style="zoom:50%;" />

这里的 Java 程序是文本格式的。比如下面这段 HelloWorld.java，它遵循的就是 Java 语言规范。其中，我们调用了 System.out 等模块，也就是 JRE 里提供的类库。

```java
 public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

使用 JDK 的工具 javac 进行编译后，会产生 HelloWorld 的字节码。
 我们一直在说 Java 字节码是沟通 JVM 与 Java 程序的桥梁，下面使用 javap 来稍微看一下字节码到底长什么样子

```java
Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
  		 // getstatic 获取静态字段的值
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       // ldc 常量池中的常量值入栈
       3: ldc           #3                  // String Hello World
       // invokevirtual 运行时方法绑定调用方法
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       //void 函数返回
       8: return
}
```

Java 虚拟机采用基于栈的架构，其指令由操作码和操作数组成。这些 字节码指令 ，就叫作 opcode。其中， getstatic、ldc、invokevirtual、return 等，就是 opcode，可以看到是比较容易理解的。

JVM 就是靠解析这些 opcode 和操作数来完成程序的执行的。当我们使用 Java 命令运行 .class 文件的时候，实际上就相当于启动了一个 JVM 进程。

然后 JVM 会翻译这些字节码，它有两种执行方式。常见的就是解释执行，将 opcode + 操作数翻译成机器代码;另外一种执行方式就是 JIT，也就是我们常说的即时编译，它会在一定条件下将字节码编译成机器码之后再执行。

