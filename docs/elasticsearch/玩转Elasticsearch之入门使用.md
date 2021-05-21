Elasticsearch是基于Lucene的全文检索引擎，本质也是存储和检索数据。ES中的很多概念与MySQL类似我们可以按照关系型数据库的经验去理解。

## 核心概念

* 索引

  类似的数据放在一个索引，非类似的数据放不同索引， 一个索引也可以理解成一个关系型数据 库。

* 类型

  代表document属于index中的哪个类别(type)也有一种说法一种type就像是数据库的表， 比如dept表，user表。

  > 注意：ES每个大版本之间区别很大
  >
  > ES 5.x中一个index可以有多种type。
  >
  > ES 6.x中一个index只能有一种type。
  >
  > ES 7.x以后 要逐渐移除type这个概念。

* 映射

  mapping定义了每个字段的类型等信息。相当于关系型数据库中的表结构。

  常用数据类型:text、keyword、number、array、boolean、date、geo_point、ip、object

  **[点此查看官方数据类型的描述](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)**
  
  

## 与关系型数据库进行比较

| 关系型数据库       | Elastisearch          |
| ------------------ | --------------------- |
| 数据库（database） | 索引（index）         |
| 表（table）        | 索引index类型（type） |
| 数据行（row）      | 文档（document）      |
| 数据列（column）   | 字段（Field）         |
| 约束（Schema）     | 映射（Mapping）       |

