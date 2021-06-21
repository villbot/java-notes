推荐👍 [在线阅读，阅读效果更好](http://elgchat.com)

![](assets/1B61E8B8-8862-4EC7-AA02-BDEEA16F666D.png)

# 关于ElgCaht

> **关于命名：** ElgChat取自于Earnestly、Learn、Guide、Chat，本意想梳理一份可以认真学习，互相交流的指南文档。
>
> **适宜人群：**：Java各阶段均可适用
>
> **产品介绍：**：本文档会系统梳理Java基础、数据结构/算法、中间件、缓存、分布式、容器、数据库、jvm、部署以及调优策略等。
>
> **如何进行贡献：** 
>
>   1. 本文档所有内容均为手敲（包含各种示例图），如有错别字请指正
>   2. 很多内容没有涉及，欢迎补充，如有不完善或错误，可以提出，会及时修正
>
> **转载须知：** 如果你需要转载本文档的一些文章，记得注明原文地址就可以了，如发现恶意抄袭/搬运，会动用法律武器维护自己的权益。让我们一起维护一个良好的技术创作环境！
>
> **说明：** 本文档会持续更新，长期进行维护。
>
> **源码：** 本文涉及到的所有代码实现均在项目 [learning](https://github.com/elgchat/learning)中可以找到。
>
> **提示：** 真实面试题汇总在本文档最后！



# Java

## JVM（必须掌握✅）

 1. **[JVM的类加载机制和双亲委派机制](docs/java/jvm/JVM的类加载机制和双亲委派机制.md)**
 2. **[JVM优化](docs/java/jvm/JVM优化.md)**

## 关键源码

 * **[Java基础之HashMap源码剖析](docs/java/basic/hashmap.md)**
 * **[Java基础之String源码剖析](docs/java/basic/string.md)**


# 网络

1. **[十分钟搞定HTTP、HTTPS协议](docs/network/HTTP和HTTPS的关系.md)**

# 数据结构与算法

数据结构与算法无疑是软件开发的基础之一，重要性毋容置疑，在当今被各类强大的三方类库在一点点减弱数据结构与算法的必要性，但是这些强大的三方类库也是基于数据结构与算法之上实现的，如果我们不去深入了解学习，那我们只能是一个使用"轮子"的人，而永远造不出来一个优秀的"轮子"。

那么衡量算法好坏的重要指标就是 [时间、空间复杂度](docs/data_structures-algorithm/时间与空间复杂度.md) ，当我们理解了计算方法之后才能去更好的学习数据结构与算法。

## 数据结构

这里给大家推荐一个[可视化数据结构和算法](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html)的网站

* **图解线性数据结构：[数组](docs/data_structures-algorithm/data_structures/数组.md)、[链表](docs/data_structures-algorithm/data_structures/链表.md)、[栈](docs/data_structures-algorithm/data_structures/栈.md)、[队列](docs/data_structures-algorithm/data_structures/队列.md)** 

* **图解散列表数据结构：[散列表（hash）](docs/data_structures-algorithm/data_structures/散列表.md)**

* **图解树状数据结构：  [树的概念](docs/data_structures-algorithm/data_structures/tree/树的概念.md)、 [二叉树](docs/data_structures-algorithm/data_structures/tree/二叉树.md)、 [红黑树](docs/data_structures-algorithm/data_structures/tree/红黑树.md)**

## 算法

* **常见算法总结：[递归](docs/data_structures-algorithm/algorithm/递归.md)、[二分查找法](docs/data_structures-algorithm/algorithm/二分查找法.md)**

* **经典题解：[斐波那契数列](docs/data_structures-algorithm/algorithm/solve/斐波那契数列.md)、[有序数组找出只出现一次的元素](docs/data_structures-algorithm/algorithm/solve/有序数组找出只出现一次的元素.md)**



# 数据库、缓存、搜索引擎

## 数据库

### MySQL

* **[从架构原理开始学习MySQL](docs/storage/mysql/MySQL架构原理.md)** 

## 缓存

缓存原指CPU上的一种高速存储器，它先于内存与CPU交换数据，速度很快，现在泛指存储在计算机上的原始数据的复制集，便于快速访问。 在互联网技术中，缓存是系统快速响应的关键技术之一

常见的缓存分为客户端缓存（页面缓存、浏览器缓存、app缓存）、网络缓存（web代理缓存、边缘缓存）、服务端缓存（数据库级缓存、平台级缓存、应用级缓存）三种

关于 **缓存** 的详细解读请看： **[《缓存的分类》](docs/storage/cache/缓存的分类.md) ** 、 **[《缓存优势及代价》](docs/storage/cache/缓存优势及代价.md)**、 **[《缓存的读写模式 》](docs/storage/cache/缓存的读写模式.md)**、 **[《缓存架构设计思路》](docs/storage/cache/缓存架构设计思路.md)** 

### Redis

* 待完善 [redis](docs/storage/redis/redis.md) 

## 搜索引擎

### Elasticsearch

Elaticsearch简称为ES,是一个开源的可扩展的分布式的**全文检索引擎**，它可以近乎实时的存储、检索数 据。本身扩展性很好，可扩展到上百台服务器，处理**PB**级别的数据。

*  **[全文搜索引擎Elasticsearch基础](docs/storage/elasticsearch/全文搜索引擎Elasticsearch基础.md)** （ES简介、特点、版本、使用场景及主流搜索方案对比）
*  **[全文搜索引擎Elasticsearch搭建](docs/storage/elasticsearch/全文搜索引擎Elasticsearch搭建.md)** （ES搭建部署、kibana搭建部署）
*  **[全文搜索引擎Elasticsearch之集成Ik分词器](docs/storage/elasticsearch/玩转Elasticsearch之集成Ik分词器.md)** （IK分词器的配置及使用）
*  **[玩转Elasticsearch之入门使用](docs/storage/elasticsearch/玩转Elasticsearch之入门使用.md)** （待完善）

# 常用框架

## Spring MVC

* **[从入门到进阶：初识SpringMVC](docs/framework/springmvc/初识SpringMVC.md)** 

# 系统设计

## web应用服务器

### Tomcat

* **[从入门到进阶：彻底掌握Apache Tomcat](docs/system_design/web_container/Apache%20Tomcat.md)** 
* **[灵魂拷问：Tomcat的类加载器是怎么打破了双亲委派机制](docs/system_design/web_container/Tomcat的类加载器.md)** 
* **[灵魂拷问：Tomcat是怎样支持HTTPS](docs/system_design/web_container/Tomcat是怎样支持HTTPS.md)** 
* **[灵魂拷问：Tomcat如何进行性能优化](docs/system_design/web_container/Tomcat性能优化.md)** 
* **[【进阶】Tomcat源码剖析](docs/system_design/web_container/Tomcat源码剖析.md)** 

## 分布式

### 分布式系统理论

一致性是分布式理论中的根本性问题，近半个世纪以来，科学家们围绕着一致性问题提出了很多理论模型，依据这些理论模型，业界也出现了很多工程实践投影。下面我们从一致性问题、特定条件下解决一致性问题的两种方法(2PC、3PC)入门，了解最基础的分布式系统理论。。

关于 分布式一致性 的详细解读请看：[一致性理论](docs/system_design/distributed/一致性理论.md)、[2PC协议](docs/system_design/distributed/一致性协议2PC.md)、[3PC协议](docs/system_design/distributed/一致性协议3PC.md)

### CAP理论

CAP 也就是 Consistency（一致性）、Availability（可用性）、Partition Tolerance（分区容错性） 这三个单词首字母组合。

关于 CAP 的详细解读请看：[《简述CAP定理》](docs/system_design/distributed/分布式CAP定理.md)

### BASE 理论

BASE 是 Basically Available（基本可用） 、Soft-state（软状态） 和 Eventually Consistent（最终一致性） 三个短语的缩写。

BASE 理论是对 CAP 中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是基于 CAP 定理逐步演化而来的，它大大降低了我们对系统的要求。

关于 BASE 的详细解读请看：[《简述BASE理论》](docs/system_design/distributed/分布式BASE理论.md)

### Paxos 算法和 Raft 算法

**Paxos 算法**诞生于 1990 年，这是一种解决分布式系统一致性的经典算法 。但是，由于 Paxos 算法非常难以理解和实现，不断有人尝试简化这一算法。到了2013 年才诞生了一个比 Paxos 算法更易理解和实现的分布式一致性算法—**Raft 算法**。

关于 Paxos算法和Raft算法 的详细解读请看：[《Paxos算法》](docs/system_design/distributed/一致性算法Paxos.md)、[《Raft算法》](docs/system_design/distributed/一致性算法Raft.md)

### 分布式事务

分布式事务就是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。

简单的说，就是一次大的操作由不同的小操作组成，这些小的操作分布在不同的服务器上，且属于不同的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。本质上来说，分布式事务就是为了保证不同数据库的数据一致性。

关于 分布式事务 的详细解读请看：[《分布式事务》](docs/system_design/distributed/分布式事务.md)

### 分布式系统架构（概念、设计策略）

随着业务量级越来越大，我们从单体项目慢慢转变称集群项目，但是发现偶然某些业务场景QPS增加，为了考虑整体业务预算等问题，最终进行了业务拆分，慢慢变进行了服务化，这就是分布式的前世今生，虽然这样解决了性能问题，但相应也面临了一些新的问题：通信异常、网路分区、节点故障、三态

关于 分布式系统概念、发展历程以及面临新的问题 的详细解读请看：[《解读分布式架构概念、发展历程以及面临的新问题》](docs/system_design/distributed/分布式架构概念、发展问题.md)

分布式系统本质是通过低廉的硬件攒在一起以获得更好的吞吐量、性能以及可用性等。
在分布式环境下，有几个问题是普遍关心的，我们称之为设计策略:

  * 如何检测当前节点还活着?
  * 如何保障高可用?
  * 容错处理
  * 负载均衡

关于 分布式系统设计策略 的详细解读 请看： [《分布式系统设计策略》](docs/system_design/distributed/分布式系统设计策略.md)

# 真实面试题汇总
 [真实面试题汇总](docs/question/面试题.md)

