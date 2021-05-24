## 集成IK分词器

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。从2006年12月推出1.0版 开始，IKAnalyzer已经推出 了3个大版本。最初，它是以开源项目Lucene为应用主体的，结合词典分词 和文法分析算法的中文分词组件。新版本的IKAnalyzer3.0则发展为 面向Java的公用分词组件，独立于 Lucene项目，同时提供了对Lucene的默认优化实现。

IK分词器3.0的特性如下:

1. 采用了特有的“正向迭代最细粒度切分算法“，具有60万字/秒的高速处理能力。 
2. 采用了多子处理器分析模式，支持:英文字母(IP地址、Email、URL)、数字(日期，常用中文数 量词，罗马数字，科学计数法)，中文词汇(姓名、地名处理)等分词处理。
3. 支持个人词条的优化的词典存储，更小的内存占用。
4. 支持用户词典扩展定义。 5)针对Lucene全文检索优化的查询分析器IKQueryParser;采用歧义分析算法优化查询关键字的搜索 排列组合，能极大的提高Lucene检索的命中率。

## 下载插件并安装
 [**IKAnalyzer分词器获取地址**](https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.3.0)

### 安装方式一（下载插件并安装）

1. 在elasticsearch的bin目录下执行以下命令,es插件管理器会自动帮我们安装，然后等待安装完成:

   ```bash
   /usr/elasticsearch/bin/elasticsearch-plugin install
   https://github.com/medcl/elasticsearch-analysis-
   ik/releases/download/v7.3.0/elasticsearch-analysis-ik-7.3.0.zip
   ```

2. 下载完成后会提示 Continue with installation?输入 y 即可完成安装 
3. 重启Elasticsearch 和Kibana

### 安装方式二（上传安装包安装）

1. 在elasticsearch安装目录的plugins目录下新建 analysis-ik 目录

   ```bash
   #新建analysis-ik文件夹
   mkdir analysis-ik
   #切换至 analysis-ik文件夹下
   cd analysis-ik
   #上传资料中的 elasticsearch-analysis-ik-7.3.0.zip #解压
   unzip elasticsearch-analysis-ik-7.3.3.zip
   #解压完成后删除zip
   rm -rf elasticsearch-analysis-ik-7.3.0.zip
   ```

2. 重启Elasticsearch 和Kibana



## 使用案例

IK分词器有两种分词模式:ik_max_word和ik_smart模式。

1. ik_max_word (常用)，会将文本做最细粒度的拆分
2. ik_smart ，会做最粗粒度的拆分

### ik_max_word分词得到的结果

```http
POST  _analyze
{
  "analyzer": "ik_max_word", 
  "text": "北京市有只猫的名字叫喜喵"
}
```

```json
{
  "tokens" : [
    {
      "token" : "北京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "北京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "市",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "有",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "只",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "猫",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "CN_CHAR",
      "position" : 5
    },
    {
      "token" : "的",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 6
    },
    {
      "token" : "名字叫",
      "start_offset" : 7,
      "end_offset" : 10,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "名字",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "CN_WORD",
      "position" : 8
    },
    {
      "token" : "叫",
      "start_offset" : 9,
      "end_offset" : 10,
      "type" : "CN_CHAR",
      "position" : 9
    },
    {
      "token" : "喜",
      "start_offset" : 10,
      "end_offset" : 11,
      "type" : "CN_CHAR",
      "position" : 10
    },
    {
      "token" : "喵",
      "start_offset" : 11,
      "end_offset" : 12,
      "type" : "CN_CHAR",
      "position" : 11
    }
  ]
}

```

### ik_smart分词得到的结果

```http
POST  _analyze
{
  "analyzer": "ik_max_word", 
  "text": "北京市有只猫的名字叫喜喵"
}
```

```json
{
  "tokens" : [
    {
      "token" : "北京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "有",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "只",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "猫",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "的",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "名字叫",
      "start_offset" : 7,
      "end_offset" : 10,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "喜",
      "start_offset" : 10,
      "end_offset" : 11,
      "type" : "CN_CHAR",
      "position" : 6
    },
    {
      "token" : "喵",
      "start_offset" : 11,
      "end_offset" : 12,
      "type" : "CN_CHAR",
      "position" : 7
    }
  ]
}
```

## 扩展词典使用

"喜喵"这个显然是一只猫的名字，那么上面的分词显然是不合理的，该怎么办？

**扩展词：**就是不想让哪些词被分开，让他们分成一个词。比如喜喵

### 自定义扩展词库

1. 进入到 config/analysis-ik/(**插件命令安装方式**) 或 plugins/analysis-ik/config(**安装包安装方式**) 目录下, 新增自定义词典

   ```bash
   vim elgchat_ext_dict.dic
   ```

2. 将我们自定义的扩展词典文件添加到IKAnalyzer.cfg.xml配置中

   ```bash
   vim IKAnalyzer.cfg.xml
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
   <properties>
   <comment>IK Analyzer 扩展配置</comment> 
   <!--用户可以在这里配置自己的扩展字典 -->
   <entry key="ext_dict">lagou_ext_dict.dic</entry>
   <!--用户可以在这里配置自己的扩展停止词字典--><entry key="ext_stopwords">lagou_stop_dict.dic</entry> 
   <!--用户可以在这里配置远程扩展字典 -->
   <!-- <entry
   key="remote_ext_dict">http://192.168.211.130:8080/tag.dic</entry> --> <!--用户可以在这里配置远程扩展停止词字典-->
   <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
   </properties>
   ```

3. 重启elasticsearch

## 停用词典使用

**停用词：**有些词在文本中出现的频率非常高。但对本文的语义产生不了多大的影响。例如英文的"a"、 "an"、"the"、"of"等或中文的"的"、"了"、"呢"等。这样的词称为停用词。停用词经常被过滤掉，不会被进行 索引。在检索的过程中，如果用户的查询词中含有停用词，系统会自动过滤掉。停用词可以加快索引的 速度，减少索引库文件的大小。

### 自定义停用词库

1. 进入到 config/analysis-ik/(**插件命令安装方式**) 或 plugins/analysis-ik/config(**安装包安装方式**) 目录下, 新增自定义词典

   ```bash
   vim lagou_stop_dict.dic
   ```

2. 输入需要停用的词典内容

3. 将我们自定义的停用词典文件添加到IKAnalyzer.cfg.xml配置中

4. 重启Elasticsearch

