Elasticsearch是一个分布式全文搜索引擎，支持单节点模式(Single-Node Mode)和集群模式(Cluster Mode)部署，一般来说，小公司的业务场景往往使用Single-Node Mode部署即可。

## Single-Node Mode部署实例

### 准备环境

访问 [Elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch) 下载页，选择需要的系统版本，这里以[7.12.1 Linux_X86_64](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.1-linux-x86_64.tar.gz)为例。
   <img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_05_21/image-20210521160750308.png" alt="image-20210521160750308" style="zoom:50%;" />

这里我们使用centos 7.9的服务器进行配置，因为elasticsearch需要基于jdk（这里使用jdk1.8的版本，如果使用其他版本jdk需要确认elasticsearch的版本是否兼容，[点此查看兼容版本]([点击查看Elasticsearch与JDK兼容官网描述](https://www.elastic.co/cn/support/matrix#matrix_jvm))）

```bash
[root@iZ8vb951fznoerkxkhut8qZ ~]# java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)
```

解压刚刚下载好的elasticsearch文件

```bash
#解压文件
tar -zxvf elasticsearch-7.12.1-linux-x86_64.tar.gz
#将解压文件移动到/usr/local/elasticsearch
mv elasticsearch-7.12.1 /usr/local/elasticsearch
```

### 配置elasticsearch

1. 编辑elasticsearch.yml文件

   ```ba
   vim /usr/local/elasticsearch/config/elasticsearch.yml
   ```

   单机安装请取消注释:node.name: node-1，否则无法正常启动。

   修改网络和端口，取消注释master节点，单机只保留一个node

   ```yaml
   node.name: node-1
   #可填入本机ip或内网ip，使用0.0.0.0 为开启外网访问
   network.host: 0.0.0.0
   #
   # Set a custom port for HTTP:
   #
   http.port: 9200
   cluster.initial_master_nodes: ["node-1"]
   ```

2. 按需修改jvm.options内存设置

   >根据实际情况修改占用内存，默认都是1G，单机1G内存，启动会占用700m+然后在安装kibana 后，基本上无法运行了，运行了一会就挂了报内存不足。 内存设置超出物理内存，也会无法启 动，启动报错。

   ```options
   -Xms1g
   -Xmx1g

3. 添加系统账户（es默认root账户无法启动，所以需要创建账户）

   ```bash
   useradd  esadmin
   
   #修改密码
   passwd   esadmin
   
   #改变当前es账户对elasticsearch目录权限
   chown -R esadmin /usr/local/elasticsearch/
   ```

4. 修改/etc/sysctl.conf，在文件最后增加vm.max_map_count=655360，执行sysctl -p生效

5. 修改/etc/security/limits.conf

   ```conf
   *               soft    nofile          65536
   *               hard    nofile          65536
   *               soft    nproc           4096
   *               hard    nproc           4096
   ```

6. 切换账户并启动

   ```bash
   #切换账户
   su esadmin
   
   #启动 (& 为后台执行)
   /usr/local/elasticsearch/bin/elasticsearch &
   ```

7. 访问 http://127.0.0.1:9200 

   ```json
   {
     "name" : "node-1", //节点名称
     "cluster_name" : "elasticsearch",
     "cluster_uuid" : "fOcw6wjPTXax8HbvnYTSiw",
     "version" : {
       "number" : "7.12.1", //版本
       "build_flavor" : "default",
       "build_type" : "tar",
       "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
       "build_date" : "2021-04-20T20:56:39.040728659Z",
       "build_snapshot" : false,
       "lucene_version" : "8.8.0",
       "minimum_wire_compatibility_version" : "6.8.0",
       "minimum_index_compatibility_version" : "6.0.0-beta1"
     },
     "tagline" : "You Know, for Search"
   }
   ```

   看到以上返回信息，则Elasticsearch启动成功，以上为Elasfticsearch配置过程

## kibana 安装配置

Kibana是一个基于Node.js的Elasticsearch索引库数据统计工具，可以利用Elasticsearch的聚合功 能，生成各种图表，如柱形图，线状图，饼图等。

而且还提供了操作Elasticsearch索引数据的控制台，并且提供了一定的API提示，非常有利于我们学习 Elasticsearch的语法。

### 准备环境

访问 [Kibana](https://www.elastic.co/cn/downloads/kibana) 下载页，选择需要的系统版本，这里以[7.12.1 Linux_64_BIT](https://artifacts.elastic.co/downloads/kibana/kibana-7.12.1-linux-x86_64.tar.gz)为例（这里需要与Elasticsearch版本保持一致）。

<img src="https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_05_21/image-20210521165609713.png" alt="image-20210521165609713" style="zoom:50%;" />

解压刚刚下载好的kibana文件

```bash
#解压文件
tar -zxvf kibana-7.12.1-linux-x86_64.tar.gz
#将解压文件移动到/usr/local/elasticsearch
mv kibana-7.12.1-linux-x86_64 /usr/local/kibana
#给esadmin用户授权
chown -R esadmin /usr/local/kibana/
#设置访问权限
chmod -R  777  /usr/local/kibana/
```

### 配置kibana

```bash
vim /usr/local/kibana/config/kibana.yml
```

```yaml
server.port: 5601
server.host: "0.0.0.0"
# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://127.0.0.1:9200"]
```

配置完成，启动服务

```bash
#切换用户
su esadmin
#启动kibana (& 为后台执行)
./bin/kibana &
```

