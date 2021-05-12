MySQL 是最流行的关系型数据库软件之一，由于其体积小、速度快、开源免费、简单易用、维护成本
低等，在集群架构中易于扩展、高可用，因此深受开发者和企业的欢迎。

## mySql 体系架构

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_05_12/image-20210512155544830.png" alt="image-20210512155544830" style="zoom:33%;" />

MySQL Server架构自顶向下大致可以分网络连接层、服务层、存储引擎层和系统文件层。

### 网络连接层

* 客户端连接器(Client Connectors)：提供与MySQL服务器建立的支持。目前几乎支持所有主流的服务端编程技术，例如常见的 Java、C、Python、.NET等，它们通过各自API技术与MySQL建立连接。

### 服务层

服务层是MySQL Server的核心，主要包含系统管理和控制工具、连接池、SQL接口、解析器、查询优化器和缓存六个部分。

* 连接池：负责存储和管理客户端与数据库的连接，一个线程负责管理一个连接
* 系统管理和控制工具：备份回复、安全管理、集群管理等
* SQL接口：用于接受客户端发送的各种sql命令，并且返回用户需要查询的结果，比如DDL、DML、存储过程、视图、触发器等。
* 解析器：负责将请求的SQL解析生成一个"解析树"，然后根据一些Mysql规则进一步检查解析树是否合法
* 查询优化器：当"解析树"通过解析器语法检查后，将由优化器将其转化为执行计划，然后与存储引擎交互

>Select id form t_user where gender = 1;
>
>选取 -> 投影 -> 联接 策略
>
>1. Select先根据where语句进行选取，并不是查询出全部数据再过滤
>2. select查询根据id进行属性投影，并不是取出所有字段
>3. 将前面选取和投影连接起来最终生成查询结果

* 缓存：缓存机制是由一系列小缓存组成的，比如表缓存、记录缓存、权限缓存、引擎缓存等。如果查询缓存命中的查询结果，查询语句就可以去查询缓存中取数据

### 存储引擎层

存储引擎负责Mysql中数据的存储与读取，与底层系统文件进行交互，mysql存储引擎是插件式的，服务器中节骨的查询执行引擎通过接口与存储引擎进行通讯，接口屏蔽了不同存储引擎之间的差异，现在有很多存储引擎，各有个的特点，最常见的是MyISAM和InnoDB引擎

### 系统文件

该层负责将数据库的数据和日志存储在文件系统之上，并完成与存储引擎的交互，是文件的物理存储层，主要包含日志文件，数据文件，配置文件，pid文件，socket文件等

* 日志文件

  * 错误日志（error log）

    ```sql
    # 默认开启
    show variables like '%log_error%'
    ```

  * 通用查询日志（General query log）

    ```sql
    # 记录一般查询语句
    show variables like '%general%'
    ```

  * 二进制日志（binary log）

    记录了对mysql数据库执行的更改操作，并且记录了语句的发生时间、执行时常；但他不记录select、show等不修改数据的sql，主要用于数据库的恢复和主从复制

    ```sql
    # 是否开启
    show variables like '%log_bin%';
    
    # 参数查看
    show variables like '%binlog%';
    
    # 查看日志文件
    show binary logs;
    ```

  * 慢查询日志（slow query log）

    记录所有执行时间超市的查询sql，默认是10秒

    ```sql
    # 是否开启
    show variables like '%slow_query%'
    
    #时长
    show variables like '%long_query_time%'
    ```

* 配置文件

  用于存放mysql所有的配置信息文件，比如my.cnf，my.ini等

* 数据文件

  * db.opt文件：记录这个库的默认使用的字符集和校验规则
  * frm文件：存储与表相关的元数据（meta）信息，包括表结构的定义信息等，每一张表都会有一个frm文件
  * MYD文件：MyISAM存储引擎专用，存放MyISAM表的数据（data），每一张表都会有一个MYD文件
  * MYI文件：MyISAM存储引擎专用，存放MyISAM表的索引信息，每一张MyISAM表对应一个MYI文件
  * ibd文件和IBDATA文件：存放InnoDB的数据文件（包含索引），InnoDB存储引擎有两种表空间方式：独享表空间和共享表空间，独享表空间使用.ibd文件存放时间，且每一张InnoDB表对应一个.ibd文件，共享表使用.ibdata文件，所有的表共同使用一个（或多个，自行配置）.ibdata文件
  * ibdata1文件：系统表空间数据文件，存储表元数据、Undo日志等
  * ib_logfile0、ib_logfile1文件：Redo log日志文件

* pid文件

  pid文件是mysqld应用程序在unix/linux环境下的一个进程文件，和许多其他unix/linux服务端程序一样，它存放着自己进程id

* socket文件

  socket文件也是在unix/linux环境下拥有的，用户在unix/linux环境下客户端连接可以不通过tcp/ip网络而之间使用unix socket来连接mysql

## MySql运行机制