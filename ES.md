## 安装

### 下载地址

ElasticSearch:[https://mirrors.huaweicloud.com/elasticsearch/?C=N&O=D](https://mirrors.huaweicloud.com/elasticsearch/?C=N&O=D)

logstash:[https://mirrors.huaweicloud.com/logstash/?C=N&O=D](https://mirrors.huaweicloud.com/logstash/?C=N&O=D)

kibana:[https://mirrors.huaweicloud.com/kibana/?C=N&O=D](https://mirrors.huaweicloud.com/kibana/?C=N&O=D)

### windos安装

下载Elasticsearch`7.6.2`的zip包，并解压到指定目录

* 安装中文分词插件，在`elasticsearch-7.6.2\bin`目录下执行以下命令：
```plain
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip
```
![图片](https://uploader.shimo.im/f/C6Kz0ZWgjyMop44E.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 运行bin目录下的`elasticsearch.bat`启动Elasticsearch服务。
### docker安装

* 下载Elasticsearch`7.6.2`的docker镜像：
```plain
docker pull elasticsearch:7.6.2
```
* 修改虚拟内存区域大小，否则会因为过小而无法启动:
```plain
sysctl -w vm.max_map_count=262144
```
* 使用如下命令启动Elasticsearch服务：
```plain
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.6.2
```
* 启动时会发现`/usr/share/elasticsearch/data`目录没有访问权限，只需要修改`/mydata/elasticsearch/data`目录的权限，再重新启动即可；
```plain
chmod 777 /mydata/elasticsearch/data/
```
* 安装中文分词器IKAnalyzer，并重新启动：
```plain
docker exec -it elasticsearch /bin/bash
#此命令需要在容器中运行
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip
docker restart elasticsearch
```
### [集群状态查看](https://macrozheng.github.io/mall-learning/#/reference/elasticsearch_start?id=%e9%9b%86%e7%be%a4%e7%8a%b6%e6%80%81%e6%9f%a5%e7%9c%8b)

* 查看集群健康状态；
```plain
GET /_cat/health?v
```
* 查看节点状态；
```plain
GET /_cat/nodes?v
```
* 查看所有索引信息；
```plain
GET /_cat/indices?v
```
### IK 分词器

#### 测试分词器

在添加文档时会进行分词，索引中存放的就是一个一个的词（term），当你去搜索时就是拿关键字去匹配词，最终

找到词关联的文档。

测试当前索引库使用的分词器：

```plain
post 发送：localhost:9200/_analyze
{"text":"测试分词器，后边是测试内容：spring cloud 实战"}
```
结果会发现分词的效果将 “测试” 这个词拆分成两个单字“测”和“试”，这是因为当前索引库使用的分词器对中文就是单字分词。
#### 安装 IK 分词器

* 使用 IK 分词器可以实现对中文分词的效果。
* 下载 IK 分词器：（Github 地址：[https://github.com/medcl/elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)）
* 下载 zip：
* 解压，并将解压的文件拷贝到 ES 安装目录的 plugins 下的 ik 目录下
* 测试分词效果：
```plain
   post localhost:9200/_analyze
{
    "text": "测试分词器，后边是测试内容：spring cloud 实战",
    "analyzer": "ik_max_word"
}
```
#### 两种分词模式

ik 分词器有两种分词模式：ik_max_word 和 ik_smart 模式。

1、ik_max_word

会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、

华人、人民共和国、人民、共和国、大会堂、大会、会堂等词语。

2、ik_smart

会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。

测试两种分词模式：

发送：post localhost:9200/_analyze

{"text":"中华人民共和国人民大会堂","analyzer":"ik_smart" }

4.4 自定义词库

如果要让分词器支持一些专有词语，可以自定义词库。

iK 分词器自带一个 main.dic 的文件，此文件为词库文件。

在上边的目录中新建一个 my.dic 文件（注意文件格式为 utf-8（不要选择 utf-8 BOM））

可以在其中自定义词汇：

比如定义：

配置文件中配置 my.dic，

重启 ES，测试分词效果：

```plain
发送：post localhost:9200/_analyze
{"text":"测试分词器，后边是测试内容：spring cloud 实战","analyzer":"ik_max_word" }
```
## ES 快速入门

ES 作为一个索引及搜索服务，对外提供丰富的 REST 接口，快速入门部分的实例使用 head 插件来测试，目的是对 ES的使用方法及流程有个初步的认识。

### 1 创建索引库

ES 的索引库是一个逻辑概念，它包括了分词列表及文档列表，同一个索引库中存储了相同类型的文档。它就相当于MySQL 中的表，或相当于 Mongodb 中的集合。

关于索引这个语：索引（名词）：ES 是基于 Lucene 构建的一个搜索服务，它要从索引库搜索符合条件索引数据。索引（动词）：索引库刚创建起来是空的，将数据添加到索引库的过程称为索引。下边介绍两种创建索引库的方法，它们的工作原理是相同的，都是客户端向 ES 服务发送命令。

1. 使用 postman 或 curl 这样的工具创建：
```json
put http://localhost:9200/索引库名称
{
    "settings": {
        "index": {
            "number_of_shards": 1,
            "number_of_replicas": 0
        }
    }
}
```
number_of_shards：设置分片的数量，在集群中通常设置多个分片，表示一个索引库将拆分成多片分别存储不同的结点，提高了 ES 的处理能力和高可用性，入门程序使用单机环境，这里设置为 1。
number_of_replicas：设置副本的数量，设置副本是为了提高 ES 的高可靠性，单机环境设置0.

1. 使用head插件创建

![图片](https://uploader.shimo.im/f/FVhBg8v7p687kzve.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

### 2 映射

#### 2.1 概念说明

在索引中每个文档都包括了一个或多个 field，创建映射就是向索引库中创建 field 的过程，下边是 document 和 field

与关系数据库的概念的类比：

```plain
Near Realtime（近实时）：Elasticsearch是一个近乎实时的搜索平台，这意味着从索引文档到可搜索文档之间只有一个轻微的延迟(通常是一秒钟)。
Cluster（集群）：群集是一个或多个节点的集合，它们一起保存整个数据，并提供跨所有节点的联合索引和搜索功能。每个群集都有自己的唯一群集名称，节点通过名称加入群集。
Node（节点）：节点是指属于集群的单个Elasticsearch实例，存储数据并参与集群的索引和搜索功能。可以将节点配置为按集群名称加入特定集群，默认情况下，每个节点都设置为加入一个名为elasticsearch的群集。
Index（索引）：索引是一些具有相似特征的文档集合，类似于MySql中数据库的概念。
Type（类型）：类型是索引的逻辑类别分区，通常，为具有一组公共字段的文档类型，类似MySql中表的概念。注意：在Elasticsearch 6.0.0及更高的版本中，一个索引只能包含一个类型。
Document（文档）：文档是可被索引的基本信息单位，以JSON形式表示，类似于MySql中行记录的概念。
Shards（分片）：当索引存储大量数据时，可能会超出单个节点的硬件限制，为了解决这个问题，Elasticsearch提供了将索引细分为分片的概念。分片机制赋予了索引水平扩容的能力、并允许跨分片分发和并行化操作，从而提高性能和吞吐量。
Replicas（副本）：在可能出现故障的网络环境中，需要有一个故障切换机制，Elasticsearch提供了将索引的分片复制为一个或多个副本的功能，副本在某些节点失效的情况下提供高可用性。
doc_values :支持排序
```
注意：6.0 之前的版本有 type（类型）概念，type 相当于关系数据库的表，ES 官方将在 ES9.0 版本中彻底删除 type。

上边讲的创建索引库相当于关系数据库中的数据库还是表？

1、如果相当于数据库就表示一个索引库可以创建很多不同类型的文档，这在 ES 中也是允许的。

2、如果相当于表就表示一个索引库只能存储相同类型的文档，ES 官方建议 在一个索引库中只存储相同类型的文

档。

#### 2.2 创建映射

```plain
post http://localhost:9200/索引库名称/类型名称/_mapping
```
创建类型为 xc_course 的映射，共包括三个字段：name、description、studymondel
由于 ES6.0 版本还没有将 type 彻底删除，所以暂时把 type 起一个没有特殊意义的名字。

```plain
post 请求： http://localhost:9200/xc_course/doc/_mapping
```
表示：在 xc_course 索引库下的 doc 类型下创建映射。doc 是类型名，可以自定义，在 ES6.0 中要弱化类型的概念，
给它起一个没有具体业务意义的名称。

```plain
{
    "properties": {
        "name": {
            "type": "text"
        },
        "description": {
            "type": "text"
        },
        "studymodel": {
            "type": "keyword"
        }
    }
}
```
#### 创建索引并指定数据结构

```plain
PUT /book
{
"settings": {
#分片数
"number_of_shards": 5,
#备份数
"number_of_replicas": 1
},
#指定数据类型
"mappings": {
#类型 Type
"novel":{
#文档存储的field
"properties":{
#field属性名
"name":{
#类型
"type":"text",
#指定分词器
"analyzer":"ik_max_word",
#指定当前的field可以被作为查询的条件
"index":true,
#是否需要额外存储
"store":false
},
"author":{
"type":"keyword"
},
"count":{
"type":"long"
},
"on-sale":{
"type":"date",
#指定时间类型的格式化方式
"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
},
"descr":{
"type":"text",
"analyzer":"ik_max_word"
}
}
}
}
}
```
#### 

#### 2.3 数据类型

```plain
字符串类型:
  text: 一般用于全文检索，将当前field 进行分词
  keyword:当前field  不会进行分词
数值类型：
  long:
  Intger:
  short:
  byte:
  double:
  float:
  half_float: 精度比float 小一半
  scaled_float:根据一个long 和scaled 来表达一个浮点型 long-345, -scaled 100 ->3.45
时间类型：
  date类型,根据时间类型指定具体的格式
    PUT my_index
    {
      "mappings": {
        "_doc": {
          "properties": {
            "date": {
              "type":   "date",
              "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
          }
        }
      }
    }
布尔类型：
  boolean 类型，表达true 和false
二进制类型：
  binary类型暂时支持Base64编码的字符串
范围类型：
  integer_range：
  float_range：
  long_range：赋值时，无需指定具体的内容，只需存储一个范围即可，gte,lte,gt,lt,
  double_range：
  date_range：
  ip_range：
    PUT range_index
    {
      "settings": {
        "number_of_shards": 2
      },
      "mappings": {
        "_doc": {
          "properties": {
            "expected_attendees": {
              "type": "integer_range"
            },
            "time_frame": {
              "type": "date_range", 
              "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
          }
        }
      }
    }
    PUT range_index/_doc/1?refresh
    {
      "expected_attendees" : { 
        "gte" : 10,
        "lte" : 20
      },
      "time_frame" : { 
        "gte" : "2015-10-31 12:00:00", 
        "lte" : "2015-11-01"
      }
    }
经纬度类型：
  geo_point:用来存储经纬度
IP类型：
  ip:可以存储IPV4 和IPV6
```
#### 2.4 创建文档

ES 中的文档相当于 MySQL 数据库表中的记录。

发送：put 或 Post[http://localhost:9200/xc_course/doc/id值](http://localhost:9200/xc_course/doc/id值)

（如果不指定 id 值 ES 会自动生成 ID）

```plain
http://localhost:9200/xc_course/doc/1
{
    "name": "Bootstrap开发框架",
    "description": "Bootstrap是由Twitter推出的一个前台页面开发框架，在行业之中使用较为广泛。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长页面开发程序人员）轻松的实现一个不受浏览器限制的精美界面效果。",
    "studymodel": "201001"
}
```
#### 2.5 简单搜索

1、根据课程 id 查询文档

发送：get[ http://localhost:9200/xc_course/doc/1]( http://localhost:9200/xc_course/doc/1)

2、查询所有记录

发送 get[http://localhost:9200/xc_course/doc/_search](http://localhost:9200/xc_course/doc/_search)

```json
{
  "query": { "match_all": {} }
}
```
3、查询名称中包括 spring 关键字的的记录
发送：get[http://localhost:9200/xc_course/doc/_search?q=name:bootstrap](http://localhost:9200/xc_course/doc/_search?q=name:bootstrap)

4、查询学习模式为 201001 的记录

发送 get[http://localhost:9200/xc_course/doc/_search?q=studymodel:201001](http://localhost:9200/xc_course/doc/_search?q=studymodel:201001)

查询结果分析

分析上边查询结果：

```json

{
  "took" : 63,// 耗费的时间
  // 是否超时了，默认情况下不存在time_out,比如你的搜索耗时1分钟，它就等1分钟，但是不超时
  // 在发送搜索请求时可以指定超时时间
  // 比如你指定了10ms超时，它就会把这10ms内获得的数据返回给你
  "timed_out" : false,
  "_shards" : { // 你的搜索请求打到了几个shard上面去。
    // Primary Shard可以承接读、写流量。Replica Shard会承接读流量。
    // 因为我是默认配置，有五个primary shard。
    // 所以它的搜索请求会被打到5个分片上去，并且都成功了
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,// 跳过了0个
    "failed" : 0 // 失败了0个
  },
  "hits" : {//命中的情况
    "total" : 1000,// 命中率 1000个
    // _score 全文检索时使用，这个相关性得分越高，说明doc和检索的内容的越相关、越匹配
    // max_score就是最大的 _score
    "max_score" : null,
    // 默认查询前10条，直接返回每个doc的完整数据
    "hits" : [ {   
      "_index" : "bank",// 索引
      "_type" : "_doc",// type
      "_id" : "0",// id 
      "sort": [0],
      "_score" : null,// 相关性得分
      // _source里面存放的是doc的具体数据
      "_source" : 		{"account_number":0,
                       "balance":16623,
                       "firstname":"Bradshaw",
                       "lastname":"Mckenzie",
                       "age":29,
                       "gender":"F",
                       "address":"244 Columbus Place",
                       "employer":"Euron",
                       "email":"bradshawmckenzie@euron.com",
                       "city":"Hobucken",
                       "state":"CO"}
    		},
		 {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,
                   "balance":39225,
                   "firstname":"Amber",
                   "lastname":"Duke",
                   "age":32,
                   "gender":"M",
                   "address":"880 Holmes Lane",
                   "employer":"Pyrami",
                   "email":"amberduke@pyrami.com",
                   "city":"Brogan",
                   "state":"IL"}
    }, ...
    ]
  }
}
```
* took：本次操作花费的时间，单位为毫秒。
* timed_out：请求是否超时
* _shards：说明本次操作共搜索了哪些分片
* hits：搜索命中的记录
* hits.total： 符合条件的文档总数 hits.hits：匹配度较高的前 N 个文档
* hits.max_score：文档匹配得分，这里为最高分
* _score：每个文档都有一个匹配度得分，按照降序排列。
* _source：显示了文档的原始内容。
```plain

```

#### 2.6更新映射

映射创建成功可以添加新字段，已有字段不允许更新。

#### 2.7 删除映射

通过删除索引来删除映射。

#### 2.8 常用映射类型

* text 文本字段
```json
{
    "properties": {
        "name": {
            "type": "text"
        },
        "description": {
            "type": "text"
        },
        "studymodel": {
            "type": "keyword"
        }
    }
}
```
字符串包括 text 和 keyword 两种类型：
* text
* analyzer

通过 analyzer 属性指定分词器。

下边指定 name 的字段类型为 text，使用 ik 分词器的 ik_max_word 分词模式。

上边指定了 analyzer 是指在索引和搜索都使用 ik_max_word，如果单独想定义搜索时使用的分词器则可以通过

search_analyzer 对于 ik 分词器建议是索引时使用 ik_max_word 将搜索内容进行细粒分词，搜索时使用 ik_smart 提高搜索精确性。

* index通过 index 属性指定是否索引。默认为 index=true，即要进行索引，只有进行索引以从索引库搜索到。但是也有一些内容不需要索引，比如：商品图片地址只被用来展示图片，不进行搜索图片，此时可以将 index 设置为 false。

删除索引，重新创建映射，将 pic 的 index 设置为 false，尝试根据 pic 去搜索，结果搜索不到数据

* store是否在 source 之外存储，每个文档索引后会在 ES 中保存一份原始文档，存放在"_source"中，一般情况下不需要设置store 为 true，因为在_source 中已经有一份原始文
### 3.数据类型

#### 1 keyword 关键字字段

```json
{
    "properties": {
        "studymodel": {
            "type": "keyword"
        },
        "name": {
            "type": "keyword"
        }
    }
}
```

上边介绍的 text 文本字段在映射时要设置分词器，keyword 字段为关键字字段，通常搜索 keyword 是按照整体搜

索，所以创建 keyword 字段的索引时是不进行分词的，比如：邮政编码、手机号码、身份证等。keyword 字段通常用于过虑、排序、聚合等。

#### 2 date 日期类型

日期类型不用设置分词器。通常日期类型的字段用于排序。通过 format 设置日期格式

例子：下边的设置允许 date 字段存储年月日时分秒、年月日及毫秒三种格式。

```json
{
    "properties": {
        "timestamp": {
            "type": "date",
            "format": "yyyy‐MM‐dd HH:mm:ss||yyyy‐MM‐dd"
        }
    }
}
```
```plain
http://localhost:9200/xc_course/doc/3
{
    "name": "spring开发基础",
    "description": "spring 在java领域非常流行，java程序员都在用。",
    "studymodel": "201001",
    "pic": "group1/M00/00/01/wKhlQFqO4MmAOP53AAAcwDwm6SU490.jpg",
    "timestamp": "2018‐07‐04 18:28:58"
}
```
#### 3 数值类型

1、尽量选择范围小的类型，提高搜索效率

2、对于浮点数尽量用比例因子，比如一个价格字段，单位为元，我们将比例因子设置为100这在ES中会按 分 存 储，映射如下：

```plain
"price": { "type": "scaled_float", "scaling_factor": 100 }, 
```
由于比例因子为100，如果我们输入的价格是23.45则ES中会将23.45乘以100存储在ES中。 如果输入的价格是23.456，ES会将23.456乘以100再取一个接近原始值的数，得出2346。 使用比例因子的好处是整型比浮点型更易压缩，节省磁盘空间。 如果比例因子不适合，则从下表选择范围小的去用：
![图片](https://uploader.shimo.im/f/FwDlpWAbcjHhpGz5.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

#### 4 综合

```json
{
    "properties": {
        "description": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "pic": {
            "type": "text",
            "index": false
        },
        "price": {
            "type": "float"
        },
        "studymodel": {
            "type": "keyword"
        },
        "timestamp": {
            "type": "date",
            "format": "yyyy‐MM‐dd HH:mm:ss||yyyy‐MM‐dd||epoch_millis"
        }
    }
}
```
### 4.ES客户端

ES提供多种不同的客户端：

1、TransportClient ES提供的传统客户端，官方计划8.0版本删除此客户端。

2、RestClient RestClient是官方推荐使用的，它包括两种：Java Low Level REST Client和 Java High Level REST Client。 ES在6.0之后提供 Java High Level REST Client， 两种客户端官方更推荐使用 Java High Level REST Client，不过当 前它还处于完善中，有些功能还没有。  3添加RestHighLevelClient依赖及junit依赖。


```xml
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch‐rest‐high‐level‐client</artifactId>
            <version>6.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>6.2.1</version>
        </dependency>


```


```java
package com.xuecheng.search.config;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ElasticsearchConfig {
    @Value("${xuecheng.elasticsearch.hostlist}")
    private String hostlist;

    @Bean
    public RestHighLevelClient restHighLevelClient() { //解析hostlist配置信息
        String[] split = hostlist.split(",");
//创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")
                    [1]), "http");
        }
//创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }

    //项目主要使用RestHighLevelClient，对于低级的客户端暂时不用
    @Bean
    public RestClient restClient() {
//解析hostlist配置信息
        String[] split = hostlist.split(",");
//创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")
                    [1]), "http");
        }
        return RestClient.builder(httpHostArray).build();
    }
}
```
#### 创建索引库

* API 创建索引： put[http://localhost:9200/索引名称](http://localhost:9200/索引名称)创建映射： 发送：

put

```json
http://localhost:9200/索引库名称/类型名称/_mapping
{
    "settings": {
        "index": {
            "number_of_shards": 1,
            "number_of_replicas": 0
        }
    }
}
```
创建类型为xc_course的映射，共包括三个字段：name、description、studymodel

```json
http://localhost:9200/xc_course/doc/_mapping
{
    "properties": {
        "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "description": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "studymodel": {
            "type": "keyword"
        },
        "price": {
            "type": "float"
        },
        "timestamp": {
            "type": "date",
            "format": "yyyy‐MM‐dd HH:mm:ss||yyyy‐MM‐dd||epoch_millis"
        }
    }
}
```
* Java Client
```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestIndex {
    @Autowired
    RestHighLevelClient client;
    @Autowired
    RestClient restClient;

    //创建索引库
    @Test
    public void testCreateIndex() throws IOException {
//创建索引请求对象，并设置索引名称
        CreateIndexRequest createIndexRequest = new CreateIndexRequest("xc_course");
//设置索引参数
        createIndexRequest.settings(Settings.builder().put("number_of_shards", 1)
                .put("number_of_replicas", 0));
//设置映射
        createIndexRequest.mapping("doc", " {\n" +
                " \t\"properties\": {\n" +
                " \"name\": {\n" +
                " \"type\": \"text\",\n" +
                " \"analyzer\":\"ik_max_word\",\n" +
                " \"search_analyzer\":\"ik_smart\"\n" +
                " },\n" +
                " \"description\": {\n" +
                " \"type\": \"text\",\n" +
                " \"analyzer\":\"ik_max_word\",\n" +
                " \"search_analyzer\":\"ik_smart\"\n" +
                " },\n" +
                " \"studymodel\": {\n" +
                " \"type\": \"keyword\"\n" +
                " },\n" +
                " \"price\": {\n" +
                " \"type\": \"float\"\n" +
                " }\n" +
                " }\n" +
                "}", XContentType.JSON);
//创建索引操作客户端
        IndicesClient indices = client.indices();
//创建响应对象
        CreateIndexResponse createIndexResponse = indices.create(createIndexRequest);
//得到响应结果
        boolean acknowledged = createIndexResponse.isAcknowledged();
        System.out.println(acknowledged);
    }

    //删除索引库
    @Test
    public void testDeleteIndex() throws IOException {
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("xc_course");
//删除索引
        DeleteIndexResponse deleteIndexResponse = client.indices().delete(deleteIndexRequest);
//删除索引响应结果
        boolean acknowledged = deleteIndexResponse.isAcknowledged();
        System.out.println(acknowledged);
    }
}
```
#### 添加文档

* API 格式如下： PUT /{index}/{type}/{id} { "field": "value", ... } 如果不指定id，ES会自动生成。
```plain
 put http://localhost:9200/xc_course/doc/3 
 {
    "name": "spring cloud实战",
    "description": "本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud 基础入3.实战SpringBoot 4.注册中心eureka。",
    "studymodel": "201001",
    "price": 5.6
}
```



* Java Client
```java
//添加文档
@Test
public void testAddDoc()throws IOException{
//准备json数据
Map<String, Object> jsonMap=new HashMap<>();
jsonMap.put("name","spring cloud实战");
jsonMap.put("description","本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud
基础入门 3.实战Spring Boot 4.注册中心eureka。");
jsonMap.put("studymodel","201001");
SimpleDateFormat dateFormat=new SimpleDateFormat("yyyy‐MM‐dd HH:mm:ss");
jsonMap.put("timestamp",dateFormat.format(new Date()));
jsonMap.put("price",5.6f);
//索引请求对象
IndexRequest indexRequest=new IndexRequest("xc_course","doc");
//指定索引文档内容
indexRequest.source(jsonMap);
//索引响应对象
IndexResponse indexResponse=client.index(indexRequest);
//获取响应结果
DocWriteResponse.Result result=indexResponse.getResult();
System.out.println(result);
}

```
#### 更新文档

通过请求Url有两种方法：

1、完全替换 Post：[http://localhost:9200/xc_test/doc/3](http://localhost:9200/xc_test/doc/3)

```plain
{
    "name": "spring cloud实战",
    "description": "本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud 基础入3.实战SpringBoot 4.注册中心eureka。",
    "studymodel": "201001",
    "price": 5.6
}
```
2、局部更新 下边的例子是只更新price字段。
```plain
 post: http://localhost:9200/xc_test/doc/3/_update 
 { "doc":{"price":66.6} }
```
Java Client

```java
@Test
public void updateDoc()throws IOException{
        UpdateRequest updateRequest=new UpdateRequest("xc_course","doc",
        "4028e581617f945f01617f9dabc40000");
        Map<String, String> map=new HashMap<>();
        map.put("name","spring cloud实战");
        updateRequest.doc(map);
        UpdateResponse update=client.update(updateRequest);
        RestStatus status=update.status();
        System.out.println(status);
        }
```
#### 删除文档

根据id删除，格式如下：DELETE /{index}/{type}/{id}

搜索匹配删除，将搜索出来的记录删除，格式如下： POST /{index}/{type}/_delete_by_query 下边是搜索条件例子：

```json
{
    "query": {
        "term": {
            "studymodel": "201001"
        }
    }
}
```
Java Client
```java
@Test
public void testDelDoc()throws IOException{
//删除文档id
        String id="eqP_amQBKsGOdwJ4fHiC";
//删除索引请求对象
        DeleteRequest deleteRequest=new DeleteRequest("xc_course","doc",id);
//响应对象
        DeleteResponse deleteResponse=client.delete(deleteRequest);
//获取响应结果
        DocWriteResponse.Result result=deleteResponse.getResult();
        System.out.println(result);
        }
```
## 搜索管理

### 创建映射

创建xc_course索引库。 创建如下映射 post：

```plain
http://localhost:9200/xc_course/doc/_mapping
{
    "properties": {
        "description": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "pic": {
            "type": "text",
            "index": false
        },
        "price": {
            "type": "float"
        },
        "studymodel": {
            "type": "keyword"
        },
        "timestamp": {
            "type": "date",
            "format": "yyyy‐MM‐dd HH:mm:ss||yyyy‐MM‐dd||epoch_millis"
        }
    }
}
http://localhost:9200/xc_course/doc/1 
{
"name": "Bootstrap开发",
"description": "Bootstrap是由Twitter推出的一个前台页面开发框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长页面开发的程序人员）轻松的实现一个不受浏览器限制的精美界面效果。",
"studymodel": "201002",
"price":38.6,
"timestamp":"2018‐04‐25 19:11:35",
"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
http://localhost:9200/xc_course/doc/2
{
"name": "java编程基础",
"description": "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
"studymodel": "201001",
"price":68.6,
"timestamp":"2018‐03‐25 19:11:35",
"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
http://localhost:9200/xc_course/doc/3
{
    "name": "spring开发基础",
    "description": "spring 在java领域非常流行，java程序员都在用。",
    "studymodel": "201001",
    "price": 88.6,
    "timestamp": "2018‐02‐24 19:11:35",
    "pic": "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
```
### 查询所有文档

发送：post[http://localhost:9200/xc_course/doc/_search](http://localhost:9200/xc_course/doc/_search)

```json
{
    "query": {
        "match_all": {}
    },
    "_source": [
        "name",
        "studymodel"
    ]
}
```
结果说明：
* took：本次操作花费的时间，单位为毫秒。
* timed_out：请求是否超时 _shards：说明本次操作共搜索了哪些分片
* hits：搜索命中的记录
* hits.total ： 符合条件的文档总数
* hits.hits ：匹配度较高的前N个文档
* hits.max_score：文档匹配得分，这里为最高分
* _score：每个文档都有一个匹配度得分，按照降序排列。
* _source：显示了文档的原始内容。

JavaClient：

```plain
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestSearch {
    @Autowired
    RestHighLevelClient client;
    @Autowired
    RestClient restClient;

    //搜索type下的全部记录
    @Test
    public void testSearchAll() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(searchSourceBuilder);
        SearchResponse searchResponse = client.search(searchRequest);
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String) sourceAsMap.get("name");
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.out.println(name);
            System.out.println(studymodel);
            System.out.println(description);
        }
    }
```
### 分页查询

ES支持分页查询，传入两个参数：from和size。 form：表示起始文档的下标，从0开始。 size：查询的文档数量。 发送：

```json
post http://localhost:9200/xc_course/doc/_search 
{
    "from": 0,
    "size": 1,
    "query": {
        "match_all": {}
    },
    "_source": [
        "name",
        "studymodel"
    ]
}
```
```java
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("xc_course");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
//分页查询，设置起始下标，从0开始
        searchSourceBuilder.from(0);
//每页显示个数
        searchSourceBuilder.size(10);
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(searchSourceBuilder);
        SearchResponse searchResponse = client.search(searchRequest);
```
### Term Query and terms

Term Query为精确查询，在搜索时会整体匹配关键字，不再将关键字分词。 发送：

```plain
post http://localhost:9200/xc_course/doc/_search 
{
    "query": {
        "term": {
            "name": "spring"
        }
    },
    "_source": [
        "name",
        "studymodel"
    ]
}
```
```java
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("xc_course");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.termQuery("name", "spring"));
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(searchSourceBuilder);
        SearchResponse searchResponse = client.search(searchRequest);
```
```plain
#terms 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "terms": {
      "province": [
        "北京",
        "晋城"
      ]
    }
  }
}
public class TermSearch {
    ObjectMapper mapper = new ObjectMapper();
    RestHighLevelClient client =  EsClient.getClient();
    String index = "sms-logs-index";
    String type="sms-logs-type";
    @Test
    public void termsSearchTest() throws IOException {
        // 1.创建request对象
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        // 2.创建查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.query(QueryBuilders.termsQuery("province","北京","晋城"));
        request.source(builder);
        // 3.执行查询
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 输出查询结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
}
```

### 根据id精确匹配

```json

post： http://127.0.0.1:9200/xc_course/doc/_search
{
    "query": {
        "ids": {
            "type": "doc",
            "values": [
                "3",
                "4",
                "100"
            ]
        }
    }
}
```
JavaClient:
```java
String[] split = new String[]{"1","2"}; List idList = Arrays.asList(split); searchSourceBuilder.query(QueryBuilders.termsQuery("_id", idList));
```
### match Query

1、基本使用 match Query即全文检索，它的搜索方式是先将搜索字符串分词，再使用各各词条从索引中搜索。 match query与Term query区别是match query在搜索前先将搜索关键字分词，再拿各各词语去索引中搜索。 发送：

```json
post http://localhost:9200/xc_course/doc/_search 
{
    "query": {
        "match": {
            "description": {
                "query": "spring开发",
                "operator": "or"
            }
        }
    }
}
```
query：搜索的关键字，对于英文关键字如果有多个单词则中间要用半角逗号分隔，而对于中文关键字中间可以用 逗号分隔也可以不用。 operator：or 表示 只要有一个词在文档中出现则就符合条件，and表示每个词都在文档中出现则才符合条件。 上边的搜索的执行过程是：

1、将“spring开发”分词，分为spring、开发两个词

2、再使用spring和开发两个词去匹配索引中搜索。

3、由于设置了operator为or，只要有一个词匹配成功则就返回该文档。

JavaClient：

```java
        //根据关键字搜索
        @Test
        public void testMatchQuery () throws IOException {
            SearchRequest searchRequest = new SearchRequest("xc_course");
            searchRequest.types("xc_course");
            SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//source源字段过虑
            searchSourceBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
//匹配关键字
            searchSourceBuilder.query(QueryBuilders.matchQuery("description", "spring开
                    发").operator(Operator.OR));
                    searchRequest.source(searchSourceBuilder);
            SearchResponse searchResponse = client.search(searchRequest);
            SearchHits hits = searchResponse.getHits();
            SearchHit[] searchHits = hits.getHits();
            for (SearchHit hit : searchHits) {
                String index = hit.getIndex();
                String type = hit.getType();
                String id = hit.getId();
                float score = hit.getScore();
                String sourceAsString = hit.getSourceAsString();
                Map<String, Object> sourceAsMap = hit.getSourceAsMap();
                String name = (String) sourceAsMap.get("name");
                String studymodel = (String) sourceAsMap.get("studymodel");
                String description = (String) sourceAsMap.get("description");
                System.out.println(name);
                System.out.println(studymodel);
                System.out.println(description);
```
2、minimum_should_match 上边使用的operator = or表示只要有一个词匹配上就得分，如果实现三个词至少有两个词匹配如何实现？ 使用minimum_should_match可以指定文档匹配词的占比： 比如搜索语句如下
```json
{
    "query": {
        "match": {
            "description": {
                "query": "spring开发框架",
                "minimum_should_match": "80%"
            }
        }
    }
}
```
“spring开发框架”会被分为三个词：spring、开发、框架 设置"minimum_should_match": "80%"表示，三个词在文档的匹配占比为80%，即3*0.8=2.4，向上取整得2，表 示至少有两个词在文档中要匹配成功。 对应的RestClient如下：
```java
//匹配关键字 MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("description", "前台页面开发框架 架 构") .minimumShouldMatch("80%");//设置匹配占比 searchSourceBuilder.query(matchQueryBuilder);
```
### multi Query

上边学习的termQuery和matchQuery一次只能匹配一个Field，multiQuery，一次可以匹配多个字段。 1、基本使用 单项匹配是在一个field中去匹配，多项匹配是拿关键字去多个Field中匹配。

```json
post http://localhost:9200/xc_course/doc/_search
{
    "query": {
        "multi_match": {
            "query": "spring css",
            "minimum_should_match": "50%",
            "fields": [
                "name",
                "description"
            ]
        }
    }
}
```
拿关键字 “spring css”去匹配name 和description字段。
匹配多个字段时可以提升字段的boost（权重）来提高得分 例子： 提升boost之前，执行下边的查询：

```json
{
    "query": {
        "multi_match": {
            "query": "spring css",
            "minimum_should_match": "50%",
            "fields": [
                "name",
                "description"
            ]
        }
    }
}
```
通过查询发现Bootstrap排在前边。 提升boost，通常关键字匹配上name的权重要比匹配上description的权重高，这里可以对name的权重提升。
```json
{
    "query": {
        "multi_match": {
            "query": "spring框架",
            "minimum_should_match": "50%",
            "fields": [
                "name^10",
                "description"
            ]
        }
    }
}
```
“name^10” 表示权重提升10倍，执行上边的查询，发现name中包括spring关键字的文档排在前边。
JavaClient:

```json
MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框架", "name", "description") .minimumShouldMatch("50%"); multiMatchQueryBuilder.field("name",10);//提升boost
```
### 布尔查询

布尔查询对应于Lucene的BooleanQuery查询，实现将多个查询组合起来。 三个参数： must：文档必须匹配must所包括的查询条件，相当于 “AND” should：文档应该匹配should所包括的查询条件其 中的一个或多个，相当于 "OR" must_not：文档不能匹配must_not所包括的该查询条件，相当于“NOT”

分别使用must、should、must_not测试下边的查询： 发送：

```json
POST http://localhost:9200/xc_course/doc/_search 
{
    "_source": [
        "name",
        "studymodel",
        "description"
    ],
    "from": 0,
    "size": 1,
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "spring框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ]
                    }
                },
                {
                    "term": {
                        "studymodel": "201001"
                    }
                }
            ]
        }
    }
}
```
must：表示必须，多个查询条件必须都满足。（通常使用must） should：表示或者，多个查询条件只要有一个满足即可。 must_not：表示非。
JavaClient：

```java
    //BoolQuery，将搜索关键字分词，拿分词去索引库搜索
    @Test
    public void testBoolQuery() throws IOException {
//创建搜索请求对象
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
//创建搜索源配置对象
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.fetchSource(new String[]{"name", "pic", "studymodel"}, new String[]{});
//multiQuery
        String keyword = "spring开发框架";
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框架",
                "name", "description")
                .minimumShouldMatch("50%");
        multiMatchQueryBuilder.field("name", 10);
//TermQuery
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("studymodel", "201001");
        //布尔查询
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(multiMatchQueryBuilder);
        boolQueryBuilder.must(termQueryBuilder);
//设置布尔查询对象
        searchSourceBuilder.query(boolQueryBuilder);
        searchRequest.source(searchSourceBuilder);//设置搜索源配置
        SearchResponse searchResponse = client.search(searchRequest);
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            System.out.println(sourceAsMap);
          }
```
### 过虑器

过虑是针对搜索的结果进行过虑，过虑器主要判断的是文档是否匹配，不去计算和判断文档的匹配度得分，所以过 虑器性能比查询要高，且方便缓存，推荐尽量使用过虑器去实现查询或者过虑器和查询共同使用。 过虑器在布尔查询中使用，下边是在搜索结果的基础上进行过虑：

```json
{
    "_source": [
        "name",
        "studymodel",
        "description",
        "price"
    ],
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "spring框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ]
                    }
                }
            ],
            "filter": [
                {
                    "term": {
                        "studymodel": "201001"
                    }
                },
                {
                    "range": {
                        "price": {
                            "gte": 60,
                            "lte": 100
                        }
                    }
                }
            ]
        }
    }
}
```
range：范围过虑，保留大于等于60 并且小于等于100的记录。
term：项匹配过虑，保留studymodel等于"201001"的记录。 注意：range和term一次只能对一个Field设置范围过虑。

client：

```java
    //布尔查询使用过虑器
    @Test
    public void testFilter() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel", "price", "description"},
                new String[]{});
        searchRequest.source(searchSourceBuilder);
//匹配关键字
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框
                架", " name", " description");
//设置匹配占比
                multiMatchQueryBuilder.minimumShouldMatch("50%");
//提升另个字段的Boost值
        multiMatchQueryBuilder.field("name", 10);
        searchSourceBuilder.query(multiMatchQueryBuilder);
//布尔查询
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(searchSourceBuilder.query());
//过虑
        boolQueryBuilder.filter(QueryBuilders.termQuery("studymodel", "201001"));
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(60).lte(100));
        SearchResponse searchResponse = client.search(searchRequest);
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String) sourceAsMap.get("name");
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.out.println(name);
            System.out.println(studymodel);
            System.out.println(description);
        }
    }
```
### 排序

可以在字段上添加一个或多个排序，支持在keyword、date、float等类型上添加，text类型的字段上不允许添加排 序。 过虑0--10元价格范围的文档，并且对结果进行排序，先按studymodel降序，再按价格升序

发送 POST[http://localhost:9200/xc_course/doc/_search](http://localhost:9200/xc_course/doc/_search)

```json
{
    "_source": [
        "name",
        "studymodel",
        "description",
        "price"
    ],
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "price": {
                            "gte": 0,
                            "lte": 100
                        }
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "studymodel": "desc"
        },
        {
            "price": "asc"
        }
    ]
}
```
client：
```java
 @Test
    public void testSort() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel", "price", "description"},
                new String[]{});
        searchRequest.source(searchSourceBuilder);
//布尔查询
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
//过虑
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(0).lte(100));
        //排序
        searchSourceBuilder.sort(new FieldSortBuilder("studymodel").order(SortOrder.DESC));
        searchSourceBuilder.sort(new FieldSortBuilder("price").order(SortOrder.ASC));
        SearchResponse searchResponse = client.search(searchRequest);
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String) sourceAsMap.get("name");
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.out.println(name);
            System.out.println(studymodel);
            System.out.println(description);
        }
    }
```
### 高亮显示

高亮显示可以将搜索结果一个或多个字突出显示，以便向用户展示匹配关键字的位置。 在搜索语句中添加highlight即可实现，如下：

Post：[http://127.0.0.1:9200/xc_course/doc/_search](http://127.0.0.1:9200/xc_course/doc/_search)

```json
{
    "_source": [
        "name",
        "studymodel",
        "description",
        "price"
    ],
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "开发框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ],
                        "type": "best_fields"
                    }
                }
            ],
            "filter": [
                {
                    "range": {
                        "price": {
                            "gte": 0,
                            "lte": 100
                        }
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "price": "asc"
        }
    ],
    "highlight": {
        "pre_tags": [
            "<tag1>"
        ],
        "post_tags": [
            "</tag2>"
        ],
        "fields": {
            "name": {},
            "description": {}
        }
    }
}
```
client代码如下：

```java
@Test
    public void testHighlight() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//source源字段过虑
        searchSourceBuilder.fetchSource(new String[]{"name", "studymodel", "price", "description"},
                new String[]{});
        searchRequest.source(searchSourceBuilder);
//匹配关键字
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("开发",
                "name", "description");
        searchSourceBuilder.query(multiMatchQueryBuilder);
//布尔查询
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(searchSourceBuilder.query());
//过虑
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(0).lte(100));
//排序
        searchSourceBuilder.sort(new FieldSortBuilder("studymodel").order(SortOrder.DESC));
        searchSourceBuilder.sort(new FieldSortBuilder("price").order(SortOrder.ASC));
//高亮设置
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<tag>");//设置前缀
        highlightBuilder.postTags("</tag>");//设置后缀
// 设置高亮字段
        highlightBuilder.fields().add(new HighlightBuilder.Field("name"));
// highlightBuilder.fields().add(new HighlightBuilder.Field("description"));
        searchSourceBuilder.highlighter(highlightBuilder);
        SearchResponse searchResponse = client.search(searchRequest);
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
//名称
            String name = (String) sourceAsMap.get("name");
//取出高亮字段内容
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            if (highlightFields != null) {
                HighlightField nameField = highlightFields.get("name");
                if (nameField != null) {
                    Text[] fragments = nameField.getFragments();
                    StringBuffer stringBuffer = new StringBuffer();
                    for (Text str : fragments) {
                        stringBuffer.append(str.string());
                    }
                    name = stringBuffer.toString();
                }
            }
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.out.println(name);
            System.out.println(studymodel);
            System.out.println(description);
        }
    }
```
### prefix 查询

前缀查询，可以通过一个关键字去指定一个field 的前缀，从而查询到指定文档

```plain
#prefix 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    # 前缀匹配，相对于全文检索，前缀匹配是不会对前缀进行分词的。
  # 而且每次匹配都会扫描整个倒排索引，直到扫描完一遍才会停下来
  # 前缀搜索不会计算相关性得分所有的doc的得分都是1
  # 前缀越短能匹配到的doc就越多，性能越不好
    "prefix": {
      "corpName": {
        "value": "海"
      }
    }
  }
}

GET /your_index/your_type/_search
{    
  # 前缀搜索 + 添加权重
  "query": { 
    "prefix" : { 
  		"name" :  { 
  			"value" : "白日梦", 
  			"boost" : 2.0 
			}
		}
  }
}

#match 查询 在这里是什么都查不到的 和上边的prefix 做比较
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "match": {
      "corpName": "海"
    }
  }
}
    public  void findByPrefix() throws IOException {
        //  创建request对象
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        //  指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        //--------------------------------------------------
        builder.query(QueryBuilders.prefixQuery("corpName","阿"));
        //------------------------------------------------------
        request.source(builder);
        // 执行
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 输出结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
```
### fuzzy 查询

```plain
模糊查询，我们可以输入一个字符的大概，ES 可以根据输入的大概去匹配内容。查询结果不稳定
#fuzzy 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "fuzzy": {
      "corpName": {
        "value": "腾讯客堂",
          #指定前边几个字符是不允许出现错误的
        "prefix_length": 2
      }
    }
  }
}
    public  void findByFuzzy() throws IOException {
        //  创建request对象
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        //  指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        //--------------------------------------------------
        builder.query(QueryBuilders.fuzzyQuery("corpName","腾讯客堂").prefixLength(2));
        //------------------------------------------------------
        request.source(builder);
        // 执行
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 输出结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
    	}
    }
```
### wildcard 查询

```plain
通配查询，同mysql中的like 是一样的，可以在查询时，在字符串中指定通配符*和占位符？
#wildcard 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "wildcard": {
      "corpName": {
        "value": "海尔*"
      }
    }
  }
}
#wildcard 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "wildcard": {
      "corpName": {
        "value": "海尔??"
      }
    }
  }
}
public  void findByWildCard() throws IOException {
        //  创建request对象
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        //  指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        //--------------------------------------------------
        builder.query(QueryBuilders.wildcardQuery("corpName","海尔*"));
        //------------------------------------------------------
        request.source(builder);
        // 执行
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 输出结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
```
### 搜索推荐 match_phrase_prefix

```json
{    
   "query": {
      # 前缀匹配（关键字）
      "match_phrase_prefix" : {
        "message" : {
     						# 比如你搜索关注白日梦，经过分词器处理后会得到最后一个词是：“白日梦”
     					  # 然后他会拿着白日梦再发起一次搜索，于是你就可能搜到下面的内容：
                # “关注白日梦的微信公众号”
     						# ”关注白日梦的圈子“
                "query" : "关注白日梦",
                # 指定前缀最多匹配多少个term，超过这个数量就不在倒排索引中检索了，提升性能
                "max_expansions" : 10,
                # 提高召回率，使用slop调整term persition，贡献得分
                "slop":10
            }
       } 
  }
}
```



### rang 查询

范围查询，只针对数值类型，对一个field 进行大于或者小于的范围指定

```plain
#rang 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "range": {
      "fee": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
public  void findByRang() throws IOException {
        //  创建request对象
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        //  指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        //--------------------------------------------------
        builder.query(QueryBuilders.rangeQuery("fee").gt(10).lte(30));
        //------------------------------------------------------
        request.source(builder);
        // 执行
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 输出结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
```
### regexp 查询

```plain
正则查询，通过你编写的正则表达式去匹配内容
Ps:prefix wildcard  fuzzy 和regexp 查询效率比较低 ,在要求效率比较高时，避免使用
#regexp 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "regexp": {
      "mobile": "138[0-9]{8}"
    }
  }
}
public  void findByRegexp() throws IOException {
    //  创建request对象
    SearchRequest request = new SearchRequest(index);
    request.types(type);
    //  指定查询条件
    SearchSourceBuilder builder = new SearchSourceBuilder();
    //--------------------------------------------------
    builder.query(QueryBuilders.regexpQuery("mobile","138[0-9]{8}"));
    //------------------------------------------------------
    request.source(builder);
    // 执行
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 输出结果
    for (SearchHit hit : response.getHits().getHits()) {
        System.out.println(hit.getSourceAsMap());
    }
}
```
### 深分页 scrol l

```plain
ES 对from +size时又限制的，from +size 之和 不能大于1W,超过后 效率会十分低下
原理：
  from+size  ES查询数据的方式，
  第一步将用户指定的关键词进行分词，
  第二部将词汇去分词库中进行检索，得到多个文档id,
  第三步去各个分片中拉去数据， 耗时相对较长
  第四步根据score 将数据进行排序， 耗时相对较长
  第五步根据from 和size 的值 将部分数据舍弃，
  第六步，返回结果。
  
  scroll +size ES 查询数据的方式
  第一步将用户指定的关键词进行分词，
  第二部将词汇去分词库中进行检索，得到多个文档id,
  第三步，将文档的id放在一个上下文中
  第四步，根据指定的size去ES中检索指定个数数据，拿完数据的文档id,会从上下文中移除
  第五步，如果需要下一页的数据，直接去ES的上下文中找后续内容。
  第六步，循环第四步和第五步
  scroll 不适合做实时查询。
#scroll 查询,返回第一页数据，并将文档id信息存放在ES上下文中，并指定生存时间
POST /sms-logs-index/sms-logs-type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "sort": [
    {
      "fee": {
        "order": "desc"
      }
    }
  ]
}

#根据scroll 查询下一页数据
POST _search/scroll
{
  "scroll_id":"DnF1ZXJ5VGhlbkZldGNoAwAAAAAAABbqFk04VlZ1cjlUU2t1eHpsQWNRY1YwWWcAAAAAAAAW7BZNOFZWdXI5VFNrdXh6bEFjUWNWMFlnAAAAAAAAFusWTThWVnVyOVRTa3V4emxBY1FjVjBZZw==",
  "scroll":"1m"
}
#删除scroll上下文中的数据
DELETE _search/scroll/DnF1ZXJ5VGhlbkZldGNoAwAAAAAAABchFk04VlZ1cjlUU2t1eHpsQWNRY1YwWWcAAAAAAAAXIBZNOFZWdXI5VFNrdXh6bEFjUWNWMFlnAAAAAAAAFx8WTThWVnVyOVRTa3V4emxBY1FjVjBZZw==
public class ScrollSearch {
    ObjectMapper mapper = new ObjectMapper();
    RestHighLevelClient client =  EsClient.getClient();
    String index = "sms-logs-index";
    String type="sms-logs-type";
    @Test
    public void scrollSearch() throws IOException {
        // 1.创建request
        SearchRequest searchRequest = new SearchRequest(index);
        searchRequest.types(type);
        //  2.指定scroll信息,过期时间
        searchRequest.scroll(TimeValue.timeValueMinutes(1L));
        //  3.指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.size(4);
        builder.sort("fee", SortOrder.DESC);
        searchRequest.source(builder);
        // 4.获取返回结果scrollId,获取source
        SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
        String scrollId = response.getScrollId();
        System.out.println("-------------首页数据---------------------");
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
        while (true){
            // 5.创建scroll request
            SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId);
            // 6.指定scroll 有效时间
            scrollRequest.scroll(TimeValue.timeValueMinutes(1L));
            // 7.执行查询，返回查询结果
            SearchResponse scroll = client.scroll(scrollRequest, RequestOptions.DEFAULT);
            // 8.判断是否查询到数据，查询到输出
            SearchHit[] searchHits =  scroll.getHits().getHits();
            if(searchHits!=null && searchHits.length >0){
                System.out.println("-------------下一页数据---------------------");
                for (SearchHit hit : searchHits) {
                    System.out.println(hit.getSourceAsMap());
                }
            }else{
                //  9.没有数据，结束
                System.out.println("-------------结束---------------------");
                break;
            }
        }
        // 10.创建 clearScrollRequest
        ClearScrollRequest clearScrollRequest = new ClearScrollRequest();
        // 11.指定scrollId
        clearScrollRequest.addScrollId(scrollId);
        //12.删除scroll
        ClearScrollResponse clearScrollResponse = client.clearScroll(clearScrollRequest, RequestOptions.DEFAULT);
        // 13.输出结果
        System.out.println("删除scroll:"+clearScrollResponse.isSucceeded());
    }
}
```
### delete-by-query

```plain
根据term,match 等查询方式去删除大量索引
PS:如果你要删除的内容，时index下的大部分数据，推荐创建一个新的index,然后把保留的文档内容，添加到全新的索引
#Delet-by-query 删除
POST /sms-logs-index/sms-logs-type/_delete_by_query
{
   "query": {
    "range": {
      "fee": {
        "lt": 20
      }
    }
  }
}
    public void deleteByQuery() throws IOException {
        // 1.创建DeleteByQueryRequest
        DeleteByQueryRequest request = new DeleteByQueryRequest(index);
        request.types(type);
        // 2.指定条件
        request.setQuery(QueryBuilders.rangeQuery("fee").lt(20));
        // 3.执行
        BulkByScrollResponse response = client.deleteByQuery(request, RequestOptions.DEFAULT);
        // 4.输出返回结果
        System.out.println(response.toString());
    }
```
### boosting 查询

```plain
boosting 查询可以帮助我们去影响查询后的score
   positive:只有匹配上positive 查询的内容，才会被放到返回的结果集中
   negative: 如果匹配上了positive 也匹配上了negative, 就可以 降低这样的文档score.
   negative_boost:指定系数,必须小于1   0.5 
关于查询时，分数时如何计算的：
	搜索的关键字再文档中出现的频次越高，分数越高
	指定的文档内容越短，分数越高。
	我们再搜索时，指定的关键字也会被分词，这个被分词的内容，被分词库匹配的个数越多，分数就越高。
   
#boosting 查询
POST /sms-logs-index/sms-logs-type/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "smsContent": "战士"
        }
      }, 
      "negative": {
        "match": {
          "smsContent": "团队"
        }
      },
      "negative_boost": 0.2
    }
  }
}
    public void  boostSearch() throws IOException {
        //  1.创建 searchRequest
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        // 2.指定查询条件
        SearchSourceBuilder builder = new SearchSourceBuilder();
        BoostingQueryBuilder boost = QueryBuilders.boostingQuery(
                QueryBuilders.matchQuery("smsContent", "战士"),
                QueryBuilders.matchQuery("smsContent", "团队")
        ).negativeBoost(0.2f);
        builder.query(boost);
        request.source(builder);
        //  3.执行查询
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4.输出结果
        for (SearchHit hit : response.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
```
### 聚合查询

```plain
ES的聚合查询和mysql 的聚合查询类似，ES的聚合查询相比mysql 要强大得多。ES提供的统计数据的方式多种多样。
#ES 聚合查询的RSTFul 语法
POST /index/type/_search
{
    "aggs":{
        "(名字)agg":{
            "agg_type":{
                "属性"："值"
            }
        }
    }
}
```
去重计数聚合查询

```plain
去重计数，cardinality 先将返回的文档中的一个指定的field进行去重，统计一共有多少条
# 去重计数 查询 province
POST /sms-logs-index/sms-logs-type/_search
{
  "aggs": {
    "provinceAgg": {
      "cardinality": {
        "field": "province"
      }
    }
  }
}
    public void aggCardinalityC() throws IOException {
        // 1.创建request
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        // 2. 指定使用聚合查询方式
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.aggregation(AggregationBuilders.cardinality("provinceAgg").field("province"));
        request.source(builder);
        // 3.执行查询
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4.输出返回结果
        Cardinality agg = response.getAggregations().get("provinceAgg");
        System.out.println(agg.getValue());
    }
```
* 嵌套聚合，例如对`state`字段进行聚合，统计出相同`state`的文档数量，再统计出`balance`的平均值；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/vZAkpLGIO05RRuhA.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 对聚合搜索的结果进行排序，例如按`balance`的平均值降序排列；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/QVI5lK9zxPKY6aQU.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 按字段值的范围进行分段聚合，例如分段范围为`age`字段的`[20,30]``[30,40]``[40,50]`，之后按`gender`统计文档个数和`balance`的平均值；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/IjpxQLd6oGrMPx02.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 嵌套聚合，例如对`state`字段进行聚合，统计出相同`state`的文档数量，再统计出`balance`的平均值；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/caJcoGuGtgm1DPYc.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 对聚合搜索的结果进行排序，例如按`balance`的平均值降序排列；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/15mx3yrtT7fbkssf.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

* 按字段值的范围进行分段聚合，例如分段范围为`age`字段的`[20,30]``[30,40]``[40,50]`，之后按`gender`统计文档个数和`balance`的平均值；
```plain
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
Copy to clipboard
Error
Copied
```
![图片](https://uploader.shimo.im/f/NvahJNUHKcMyansE.png!thumbnail?fileGuid=jp9HGwXrRXtKHDpg)

6.9.2 范围统计

```plain
统计一定范围内出现的文档个数，比如，针对某一个field 的值再0~100,100~200,200~300 之间文档出现的个数分别是多少
范围统计 可以针对 普通的数值，针对时间类型，针对ip类型都可以响应。
数值 rang    
时间  date_rang     
ip   ip_rang
#针对数值方式的范围统计  from 带等于效果 ，to 不带等于效果
POST /sms-logs-index/sms-logs-type/_search
{
  "aggs": {
    "agg": {
      "range": {
        "field": "fee",
        "ranges": [
          {
            "to": 30
          },
           {
            "from": 30,
            "to": 60
          },
          {
            "from": 60
          }
        ]
      }
    }
  }
}
#时间方式统计
POST /sms-logs-index/sms-logs-type/_search
{
  "aggs": {
    "agg": {
      "date_range": {
        "field": "sendDate",
        "format": "yyyy", 
        "ranges": [
          {
            "to": "2000"
          },{
            "from": "2000"
          }
        ]
      }
    }
  }
}
#ip 方式 范围统计
POST /sms-logs-index/sms-logs-type/_search
{
  "aggs": {
    "agg": {
      "ip_range": {
        "field": "ipAddr",
        "ranges": [
          {
            "to": "127.0.0.8"
          },
          {
            "from": "127.0.0.8"
          }
        ]
      }
    }
  }
}
 public void aggRang() throws IOException {
        // 1.创建request
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        // 2. 指定使用聚合查询方式
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.aggregation(AggregationBuilders.range("agg").field("fee")
                            .addUnboundedTo(30)
                            .addRange(30,60)
                            .addUnboundedFrom(60));
        request.source(builder);
        // 3.执行查询
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4.输出返回结果
        Range agg = response.getAggregations().get("agg");
        for (Range.Bucket bucket : agg.getBuckets()) {
            String key = bucket.getKeyAsString();
            Object from = bucket.getFrom();
            Object to = bucket.getTo();
            long docCount = bucket.getDocCount();
            System.out.println(String.format("key: %s ,from: %s ,to: %s ,docCount: %s",key,from,to,docCount));
        }
    }
```
6.9.3 统计聚合

```plain
他可以帮你查询指定field 的最大值，最小值，平均值，平方和...
使用 extended_stats
#统计聚合查询 extended_stats
POST /sms-logs-index/sms-logs-type/_search
{
  "aggs": {
    "agg": {
      "extended_stats": {
        "field": "fee"
      }
    }
  }
}
// java实现   
public void aggExtendedStats() throws IOException {
        // 1.创建request
        SearchRequest request = new SearchRequest(index);
        request.types(type);
        // 2. 指定使用聚合查询方式
        SearchSourceBuilder builder = new SearchSourceBuilder();
        builder.aggregation(AggregationBuilders.extendedStats("agg").field("fee"));
        request.source(builder);
        // 3.执行查询
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4.输出返回结果
       ExtendedStats extendedStats =  response.getAggregations().get("agg");
        System.out.println("最大值："+extendedStats.getMaxAsString()+",最小值："+extendedStats.getMinAsString());
    }
```
### 图经纬度搜索

```plain
#创建一个经纬度索引,指定一个 name ,一个location
PUT /map
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
    "map":{
      "properties":{
        "name":{
          "type":"text"
        },
        "location":{
          "type":"geo_point"
        }
      }
    }
  }
}
#添加测试数据
PUT /map/map/1
{
  "name":"天安门",
  "location":{
    "lon": 116.403694,
    "lat":39.914492
  }
}
PUT /map/map/2
{
  "name":"百望山",
  "location":{
    "lon": 116.26284,
    "lat":40.036576
  }
}
PUT /map/map/3
{
  "name":"北京动物园",
  "location":{
    "lon": 116.347352,
    "lat":39.947468
  }
}
```
ES 的地图检索方式

```plain
geo_distance :直线距离检索方式
geo_bounding_box: 以2个点确定一个矩形，获取再矩形内的数据
geo_polygon:以多个点，确定一个多边形，获取多边形的全部数据
```
基于RESTFul 实现地图检索

geo_distance

```plain
#geo_distance 
POST /map/map/_search
{
  "query": {
    "geo_distance":{
        #确定一个点
      "location":{
        "lon":116.434739,
        "lat":39.909843
      },
      #确定半径
      "distance":20000,
      #指定形状为圆形
      "distance_type":"arc"
    }
  }
}
#geo_bounding_box
POST /map/map/_search
{
  "query":{
    "geo_bounding_box":{
      "location":{
        "top_left":{
          "lon":116.327805,
          "lat":39.95499
        },
        "bottom_right":{
          "lon": 116.363162,
          "lat":39.938395
        }
      }
    }
  }
}
#geo_polygon
POST /map/map/_search
{
  "query":{
    "geo_polygon":{
      "location":{
          # 指定多个点确定 位置
       "points":[
         {
           "lon":116.220296,
           "lat":40.075013
         },
          {
           "lon":116.346777,
           "lat":40.044751
         },
         {
           "lon":116.236106,
           "lat":39.981533
         } 
        ]
      }
    }
  }
}
```
java 实现 geo_polygon

```plain
    public class GeoDemo {
    RestHighLevelClient client =  EsClient.getClient();
    String index = "map";
    String type="map";
    @Test
    public void  GeoPolygon() throws IOException {
            //  1.创建searchRequest
            SearchRequest request  = new SearchRequest(index);
            request.types(type);
            //  2.指定 检索方式
            SearchSourceBuilder builder =  new SearchSourceBuilder();
            List<GeoPoint> points = new ArrayList<>();
            points.add(new GeoPoint(40.075013,116.220296));
            points.add(new GeoPoint(40.044751,116.346777));
            points.add(new GeoPoint(39.981533,116.236106));
            builder.query(QueryBuilders.geoPolygonQuery("location",points));
            request.source(builder);
            // 3.执行
            SearchResponse response = client.search(request, RequestOptions.DEFAULT);
            // 4.输出结果
            for (SearchHit hit : response.getHits().getHits()) {
                System.out.println(hit.getSourceAsMap());
            }
    }
}
```

# 7个查询优化技巧

### 优化技巧1

比如我想在name字段中发起全文检索：“赐我白日梦” ，然后呢我还希望优先得到content字段中也有“赐我白日梦” 的doc。于是可以使用下面的查询技巧。

```json
{"query": {
      "bool" : {
         "match":{
            "name":{
                 "query":"赐我白日梦"
            }
         },
         # 注意这里使用的是should，不命中也没关系，只要命中，我就给他的权重加倍。
         "should":[
             "match":{
                 "content":{
                     "query":"赐我白日梦",
                     "boost":2
                 }
             }
         ]
       } 
  }
```

### 优化技巧2

更换写法，改变占用的权重比例。

```json
# 你可以构建一个像下面这样查询
# 其中的java、golang、elasticsearch各占权重比例的1/3
GET my_index/_doc/_search
{
  "query":{
     "should":[
      { "term":{"title":"java"}},         # 1/3
      { "term":{"title":"golang"}},       # 1/3
    { "term":{"title":"elasticsearch"}} # 1/3
   ] 
 }
}

# 你可以将写法改成下面这样:
# java、golang的权重依然是1/3
# elasticsearch和python的权重各占1/6
GET my_index/_doc/_search
{
  "query":{
     "should":[
      { "term":{"title":"java"}},  # 1/3
      { "term":{"title":"golang"}},  # 1/3
      {
        "bool":{
         "should":[
           {"term":{"title":"elasticsearch"}},  # 1/6
           {"term":{"title":"python"}}
           ]
       }
     }
   ] 
 }
}
```

### 优化技巧3

第三种: 如果不希望使用相关性得分，使用下面的语法。

```json
# ES中默认会将命中的doc按照 _score 排序
# 但是filter不会影响最终的相关性得分，所以它不会贡献_score
# 所以直接像这样query直接包裹filter这种语法都不被支持
GET my_index/_doc/_search
{
    "query": {
       # 关键字
       "filter" : {
         "term" : { "title" : "this"}
     },
    "boost" : 1.2
    }
}

# 这时你可以像下面这样，使用constant_score包裹filter，然后在发起请求查询
GET my_index/_doc/_search
{
    "query": {
       # 关键字
        "constant_score" : {
            "filter" : {
              "term" : { "title" : "this"}
            },
            "boost" : 1.2
        }
    }
}
```

### 优化技巧4

 比如我对title字段进行检索，我希望检索结果中包含"java"，并且我允许检索结果中包含：”golang“ ，但是！如果检索结果中包含”golang“，我希望这个title中包含”golang“的doc的排名能靠后一些。

```json
GET my_index/_doc/_search
{
  "query":{
    "boosting": {
      "positive":{
        "match":{
          "title":"java"
        }
      },
      "negative":{
        "match":{
           "title":"golang"
         }
       },
       # 使用negative_boost将权重降低，实现让doc的排名靠后。
       "negative_boost": 0.2
    }
  }
}

```

### 优化技巧6

第六种: 重打分机制

```json
"query": { 
       # 首先发起一次全文检索
       "match":{
           "title":{
               "query":"java elasticsearch",
               "minimum_should_match":"50%"
           }
       },
       # 对全文检索的结果进行重新打分
       "rescore":{ 
           # 对全文检索的前50条进行重新打分
           "window_size":50，  
           "query": { 
                # 关键字
               "rescore_query":{
                    # match_phrase + slop 感知 term persition，贡献分数 
                    "match_phrase":{
                       "title":{
                           "query":"java elasticsearch",
                           "slop":50
                     }
                }
           }
       }
   }
```

### 优化技巧7

第七种: 提高召回率和精准度的技巧：混用match和match_phrase+slop提高召回率。注意下面的嵌套查询层级 bool、must、should

```json
GET /your_index/your_type/_search
{    
   # 混合使用match和match_phrase 平衡精准度和召回率
   "query": { 
      "bool": {  
       "must":  {
           # 全文检索虽然可以匹配到大量的文档，但是它不能控制词条之间的距离
           # 可能i love world在doc1中距离很近，但是它却被ES排在结果集的后面
           # 它的性能比match_phrase和proximity高
           "match": {
             "title": "i love world" 
           } 
        },
       "should": {
            # 因为slop有个特性：词条之间间隔的越近，移动的次数越少 最终的得分就越高
           # 于是可以借助match_phrase+slop感知term position的功能
           # 实现为距离相近的doc贡献分数，让它们靠前排列
           "match_phrase":{
               "title":{
                   "query":"i love world",
                   "slop":15
               }
           }
       }
   }
}
```

https://mp.weixin.qq.com/s/m7PLNAJdN9dAzq_Nsv6Gqw