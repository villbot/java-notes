<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_05_21/image-20210521150829408.png" alt="image-20210521150829408" style="zoom:50%;" />

Elaticsearch简称为ES,是一个开源的可扩展的分布式的**全文检索引擎**，它可以近乎实时的存储、检索数 据。本身扩展性很好，可扩展到上百台服务器，处理**PB**级别的数据。ES使用Java开发并使用Lucene作 为其核心来实现索引和搜索的功能，但是它通过简单的**RestfulAPI**和**javaAPI**来隐藏Lucene的复杂性， 从而让全文搜索变得简单。

## Elaticsearch功能

* 分布式搜索引擎

  Elasticsearch自动将海量数据分散到多台服务器上去存储和检索

* 全文检索

  提供模糊等自动度很高的查询方式，并进行相关性排名、高亮等功能

* 数据分析引擎（分组聚合）

  电商网站，可以检索最近时间段内的某种品类top前几的商品有那些？

  新闻网站，可以检索最近时间段内的访问量排名top前几的新闻板块有那些？

* 对海量数据进行近实时的处理

  因为是分布式架构，elasticsearch可以采用大量的服务器去存储和检索数据，自然而然就可以实现海量数据的处理，elasticsearch可以实现秒级的数据搜索和分析

## Elaticsearch的特点

> Elasticsearch的特点是它提供了一个极速的搜索体验。这源于它的高速(**speed**)。相比较其它 的一些大数据引擎，Elasticsearch可以实现秒级的搜索，速度非常有优势。Elasticsearch的 cluster是一种分布式的部署，极易扩展(**scale** )这样很容易使它处理PB级的数据库容量。最重要 的是Elasticsearch是它搜索的结果可以按照分数进行排序，它能提供我们最相关的搜索结果 (**relevance**) 。

1. 安装方便：没有过多的依赖，下载后只需要修改几个参数就可以搭建起一个集群
2. Json：输入和输出均为Json格式，意味着不需要定义Schema，快捷方便
3. RESTful：基本所有操作（索引、查询、甚至配置）都可以通过HTTP接口进行
4. 分布式：节点对外表现对等（每个节点都可以用来做入口）加入节点自动负载均衡
5. 多租户：可以根据不同用途分索引，可以操作多个索引
6. 支持超大数据：可以扩展到PB级的结构化和非结构化数据 海量数据近实时处理

## Elaticsearch 企业使用场景

### 常见场景

1. 搜索类场景

   举例：电商网站、招聘网站、新闻咨询类网站、各种app内的搜索

2. 日志分析场景

   经典ELK组合（Elaticsearch、Logstash、Kibana）可以完成日志收集、日志存储、日志分析查询界面基本功能，目前该方案实现很普及，大部分企业日志分析系统都使用了此方案

3. 数据预警平台及数据分析场景

   举例：电商价格预警，在支持的平台上设置价格预警，当优惠的价格低于某个值时，触发通知消息，通知用户购买

   数据分析常见的比如说电商网站销售top10的品牌，关注度、点击率等

4. 商业BI（Business intelligence）系统

   比如说大型超市，需要分析上一季度用户的消费金额、年龄段、时间段到店人数等信息，输出相应报表，并预测下一季度的热卖商品，根据年龄段定向推送适宜产品

### 常见案例

* 维基百科、百度百科：有全文检索、高亮、搜索推荐功能
* stack overflow ：有全文检索，可以根据报错信息，检索解决方法
* github：从上亿行代码中搜索需要的代码及项目
* 日志分析平台：企业内部搭建的ELK平台



## 主流全文搜索方案对比

Lucene、Solr、Elasticsearch是目前主流的全文搜索方案，基于**倒排索引**机制完成快速全文搜索。

* Lucene：Lucene是Apache基金会维护的一套完全基于Java编写的信息搜索工具包（Jar），它包含了索引结构、读取索引工具、相关性工具、排序等功能，因此在使用Lucene时仍需要我们进一步开发搜索引擎，例如：数据获取、解析、分词等方面的东西

  注意:Lucene只是一个框架，我们需要在Java程序中集成它再使用。而且需要很多的学习才能明 白它是如何运行的，熟练运用Lucene非常复杂。

* Solr：Solr是一个有HTTP接口的基于Lucene的查询服务器，是一个搜索引擎系统，封装了很多Lucene细

  节，Solr可以直接利用HTTP GET/POST请求去查询，维护修改索引。

* Elasticsearch

  Elasticsearch也是一个建立在全文搜索引擎 Apache Lucene基础上的搜索引擎。采用的策略是分 布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。

### Lucene、Solr、Elasticsearch关系

Solr和Elasticsearch都是基于Lucene实现的。但Solr和Elasticsearch之间也是有区别的

1. Solr利用zookeeper进行分布式管理，而Elasticsearch自身带有分布式协调管理功能
2. Solr比Elasticsearch实现更加全面，Solr官方提供的功能更多，而Elasticsearch本身更注重于核心功能， 高级功能多由第三方插件提供
3. Solr在传统的搜索应用中表现好于Elasticsearch，而Elasticsearch在实时搜索应用方面比Solr 表现好

## Elasticsearch的版本

Elasticsearch 主流版本为5.x、6.x、7.x版本

### 7.x版本变更内容

1. 集群连接变化：TransportClient被废弃

   > 以至于，es7的java代码，只能使用restclient。对于java编程，建议采用 High-level-rest- client 的方式操作ES集群。High-level REST client 已删除接受Header参数的API方 法,Cluster Health API默认为集群级别。

2. ES存储结构结构变化：简化Type 默认使用_doc

   > es6时，官方就提到了es7会逐渐删除索引type，并且es6时已经规定每一个index只能有一个 type。在es7中使用默认的_doc作为type，官方说在8.x版本会彻底移除type。 api请求方式也发送变化，如获得某索引的某ID的文档:GET index/_doc/id其中index和id为具 体的值

3. ES程序包默认打包jdk:以至于7.x版本的程序包大小突然增大了200MB+, 对比6.x发现，包大了 200MB+， 正是JDK的大小

4. 默认配置变化:默认节点名称为主机名，默认分片数改为1，不再是5。

5. Lucene升级为lucene 8 查询相关性速度优化:Weak-AND算法

   es可以看过是分布式lucene，lucene的性能直接决定es的性能。lucene8在top k及其他查询上有 很大的性能提升。

   > weak-and算法 核心原理:取TOPN结果集，估算命中记录数。 TOP N的时候会跳过得分低于10000的文档来达到更快的性能。

6. 间隔查询（intervals queries）：intervals query 允许用户精确控制查询词在文档中出现的先后关系，实现了对terms顺序、terms之间的距离以及它们之间的包含关系的灵活控制

7. 引入新的集群协调子系统，移除mininum_master_nodes参数，让Elasticsearch自己选择可以形成仲裁的节点

8. 7.0不会再有OOM的情况，Jvm引入新的circuit breaker（熔断）机制，当查询或聚合的数据量超过单机处理的最大内存限制时会被截断

9. 分片搜索空闲时跳过refresh

   以前的版本的数据插入时，每一秒都会有refresh动作，这使ES能成为一个近乎实时的搜索引擎，但是当没有查询操作的时候，该动作会使得es的资源得到较大的浪费

## Elasticsearch 兼容

### 与操作系统间的兼容
[点击查看Elasticsearch与操作系统兼容官网描述](https://www.elastic.co/cn/support/matrix#matrix_os)

### 与Java虚拟机（JDK）的兼容

[点击查看Elasticsearch与JDK兼容官网描述](https://www.elastic.co/cn/support/matrix#matrix_jvm)